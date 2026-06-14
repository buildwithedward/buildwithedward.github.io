---
layout: single
title: "XGBoost Hyperparameters Explained: What Each Knob Actually Does"
date: 2026-06-07
categories: [machine-learning]
tags: [xgboost, hyperparameters, regularization, early-stopping, subsample, clinical-ml]
excerpt: "n_estimators, max_depth, subsample, lambda, gamma - not as a grid to search, but as levers with specific, observable effects. Run each experiment and watch the model behave differently."
---

## Introduction

After Day 1, I understood *what* gradient boosting does - it builds a chain of correction trees. But I kept running into the same problem: I knew the parameter names but not what would actually happen if I changed them. Does cutting `max_depth` in half hurt performance? Does adding more trees always help? What's the difference between `lambda` and `gamma` if they both "regularize"?

This post answers those questions with experiments. Every parameter gets its own before/after test on our synthetic readmission dataset, so you can see the effect directly rather than reading descriptions of it.

---

## The Dataset

We'll reuse the same 30-day readmission dataset from Day 1 - slightly larger this time so the differences between configurations are more visible.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score
import xgboost as xgb

np.random.seed(42)
N = 5000

age             = np.random.normal(65, 15, N).clip(18, 95)
creatinine      = np.random.exponential(1.2, N).clip(0.5, 12)
bmi             = np.random.normal(28, 6, N).clip(16, 55)
num_comorbid    = np.random.poisson(2.5, N).clip(0, 8)
los_days        = np.random.exponential(5, N).clip(1, 30)
prev_admissions = np.random.poisson(1.0, N).clip(0, 6)
heart_rate_bpm  = np.random.normal(78, 15, N).clip(45, 140)

logit = (-3.5 + 0.025*age + 0.30*creatinine - 0.02*bmi
         + 0.40*num_comorbid + 0.08*los_days + 0.50*prev_admissions
         + np.random.normal(0, 0.6, N))
readmission_30d = (1 / (1 + np.exp(-logit)) > 0.5).astype(int)

X = pd.DataFrame({
    "age": age, "creatinine_mg_dl": creatinine, "bmi": bmi,
    "num_comorbidities": num_comorbid, "los_days": los_days,
    "prev_admissions": prev_admissions, "heart_rate_bpm": heart_rate_bpm,
})
y = readmission_30d

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
X_tr, X_val, y_tr, y_val = train_test_split(X_train, y_train, test_size=0.15, random_state=1)

def evaluate(model, label=""):
    train_auc = roc_auc_score(y_tr,   model.predict_proba(X_tr)[:, 1])
    val_auc   = roc_auc_score(y_val,  model.predict_proba(X_val)[:, 1])
    test_auc  = roc_auc_score(y_test, model.predict_proba(X_test)[:, 1])
    gap = train_auc - val_auc
    print(f"{label:<45} train={train_auc:.3f}  val={val_auc:.3f}  test={test_auc:.3f}  gap={gap:.3f}")

print("Dataset ready:", X_train.shape, "training patients,", y_train.mean():.1%}, "readmission rate")
```

---

## n_estimators and learning_rate: The Budget Pair

Think of building a puzzle. You can either place 10 large pieces (aggressive, fast, but the picture is rough) or 100 small pieces (slower, more precise). The total amount of work is similar - but the fine-grained version fills in detail the coarse version misses.

In XGBoost, `n_estimators` is the number of trees (puzzle pieces) and `learning_rate` controls how much each tree contributes. They work as a pair: lower learning rate + more trees = more precise corrections, but takes longer. The key insight is that **cutting the learning rate in half means you roughly need twice as many trees to reach the same quality**.

```python
configs = [
    ("lr=0.30, n=100  (fast & rough)",   dict(learning_rate=0.30, n_estimators=100)),
    ("lr=0.10, n=300  (balanced)",        dict(learning_rate=0.10, n_estimators=300)),
    ("lr=0.05, n=600  (slow & precise)", dict(learning_rate=0.05, n_estimators=600)),
]

print(f"{'Config':<45} {'train':>6} {'val':>6} {'test':>6} {'gap':>6}")
print("-" * 70)
for label, params in configs:
    m = xgb.XGBClassifier(max_depth=4, subsample=0.8, eval_metric="logloss",
                           random_state=42, **params)
    m.fit(X_tr, y_tr, verbose=False)
    evaluate(m, label)
```

**Try this:** Set `learning_rate=0.30, n_estimators=600` - you're using the budget of the third config but spending it all in coarse steps. Compare to the third row. Same tree count, much worse result. This shows that the *combination* matters, not just the count.

---

## max_depth: How Complex Can Each Correction Be?

Picture a doctor making a diagnosis. A depth-1 decision is: "is the patient's age over 70?" That's one condition. A depth-3 decision is: "is the patient over 70 AND has kidney disease AND was admitted more than twice before?" Three chained conditions catch much more specific patterns.

`max_depth` sets the maximum number of conditions a single tree can chain together. More depth = each tree can capture more complex patient combinations, but also risks memorizing specific training patients instead of learning general patterns.

```python
print(f"{'max_depth':<12} {'train':>6} {'val':>6} {'test':>6} {'gap':>6}  note")
print("-" * 65)
notes = {1: "single condition only", 3: "sweet spot (usually)", 4: "XGBoost default",
         6: "starting to overfit", 10: "memorising patients"}
for depth in [1, 3, 4, 6, 10]:
    m = xgb.XGBClassifier(n_estimators=300, learning_rate=0.1, max_depth=depth,
                           eval_metric="logloss", random_state=42)
    m.fit(X_tr, y_tr, verbose=False)
    train_auc = roc_auc_score(y_tr,   m.predict_proba(X_tr)[:, 1])
    val_auc   = roc_auc_score(y_val,  m.predict_proba(X_val)[:, 1])
    test_auc  = roc_auc_score(y_test, m.predict_proba(X_test)[:, 1])
    gap = train_auc - val_auc
    print(f"{depth:<12} {train_auc:>6.3f} {val_auc:>6.3f} {test_auc:>6.3f} {gap:>6.3f}  {notes[depth]}")
```

The `gap` column (train AUC minus val AUC) is your overfitting alarm. Depth 1 has a tiny gap but low performance - it's not complex enough to learn the patterns. Depth 10 has a huge gap - the model is memorising individual training patients rather than learning general clinical patterns that hold up on new patients.

**Try this:** Set `max_depth=10` and look at the training AUC - it'll be near 1.0. Then ask yourself: does a model that perfectly predicts every training patient actually know anything useful about a patient it's never seen?

---

## subsample and colsample_bytree: Deliberately Using Less Data

This one surprised me at first. Why would you *intentionally* train each tree on a subset of the data? Isn't more always better?

The reason is the same as why you shouldn't ask the same person to review every decision: if you always use all the data for every tree, each tree tends to make similar mistakes - they're all learning from exactly the same patients. By sampling a random 80% of patients for each tree (that's `subsample=0.8`) and a random 80% of features (that's `colsample_bytree=0.8`), each tree sees a slightly different view of the problem. The ensemble corrects its own blind spots.

Think of it like a hiring committee where each member interviews a different random subset of candidates rather than everyone interviewing everyone. The combined judgment is less biased than any one person's view.

```python
print(f"{'subsample / colsample':<28} {'train':>6} {'val':>6} {'test':>6} {'gap':>6}")
print("-" * 55)
for ss, cs, label in [
    (1.0, 1.0, "ss=1.0, cs=1.0  (no sampling)"),
    (0.8, 0.8, "ss=0.8, cs=0.8  (XGBoost default-ish)"),
    (0.6, 0.6, "ss=0.6, cs=0.6  (more diversity)"),
    (0.3, 0.3, "ss=0.3, cs=0.3  (too aggressive)"),
]:
    m = xgb.XGBClassifier(n_estimators=300, learning_rate=0.1, max_depth=4,
                           subsample=ss, colsample_bytree=cs,
                           eval_metric="logloss", random_state=42)
    m.fit(X_tr, y_tr, verbose=False)
    train_auc = roc_auc_score(y_tr,   m.predict_proba(X_tr)[:, 1])
    val_auc   = roc_auc_score(y_val,  m.predict_proba(X_val)[:, 1])
    gap = train_auc - val_auc
    print(f"{label:<28} {train_auc:>6.3f} {val_auc:>6.3f}  gap={gap:.3f}")
```

The `ss=0.3, cs=0.3` row shows what happens when you go too far: each tree sees so little data that it can't learn anything coherent. The sweet spot is typically 0.7–0.9 for both.

**Try this:** Set both to `1.0` and `max_depth=6` - no sampling, deep trees. Watch the gap widen as the model memorises training patients. Then add back `subsample=0.8, colsample_bytree=0.8` and watch the gap shrink.

---

## lambda and gamma: Two Very Different Kinds of Regularization

Both `reg_lambda` and `gamma` are described as "regularization" in the docs, but they work completely differently. Getting them confused is easy - I did for a while.

**`reg_lambda` (L2 weight penalty)** is like a doctor who's naturally cautious - they never assign an extreme risk score without strong evidence. Technically, it adds a penalty to the loss function proportional to the square of each leaf's predicted value. Large leaf values (extreme predictions) get penalised more. The effect: the model hedges its probability estimates for patients it hasn't seen many examples like.

**`gamma` (minimum split gain)** is like a hiring standard: "don't add a new question to the interview unless it meaningfully improves your ability to distinguish good candidates." A split only happens if it reduces the loss by at least `gamma`. It prevents the tree from growing branches that barely improve anything.

```python
print(f"{'lambda / gamma':<35} {'train':>6} {'val':>6} {'gap':>6}  effect")
print("-" * 75)
tests = [
    (dict(reg_lambda=0,  gamma=0),  "no regularization at all"),
    (dict(reg_lambda=1,  gamma=0),  "lambda only (XGBoost default)"),
    (dict(reg_lambda=0,  gamma=2),  "gamma only (split threshold)"),
    (dict(reg_lambda=5,  gamma=2),  "both moderate"),
    (dict(reg_lambda=20, gamma=10), "both heavy"),
]
for params, note in tests:
    m = xgb.XGBClassifier(n_estimators=300, learning_rate=0.1, max_depth=6,
                           subsample=0.8, eval_metric="logloss", random_state=42, **params)
    m.fit(X_tr, y_tr, verbose=False)
    train_auc = roc_auc_score(y_tr,  m.predict_proba(X_tr)[:, 1])
    val_auc   = roc_auc_score(y_val, m.predict_proba(X_val)[:, 1])
    lam = params["reg_lambda"]; gam = params["gamma"]
    print(f"λ={lam:<3} γ={gam:<4}  {note:<26} {train_auc:>6.3f} {val_auc:>6.3f} {train_auc-val_auc:>6.3f}")
```

**Try this:** Set `reg_lambda=0, gamma=0, max_depth=8` - no protection at all, deep trees. Then add `reg_lambda=10` while keeping everything else the same. The gap will shrink significantly even without changing the depth.

---

## Early Stopping: The Self-Calibrating Tree Count

Picking `n_estimators=300` is a guess. Sometimes 150 is enough; sometimes you need 500. Early stopping removes the guessing by watching the validation performance in real time and stopping when it hasn't improved in a while.

The analogy: when you're baking bread, you don't set a timer and walk away - you tap the loaf every few minutes. When it sounds hollow, you stop. Early stopping does the same: it checks the validation score after every round and stops when adding more trees stops helping.

```python
model_es = xgb.XGBClassifier(
    n_estimators=2000,          # high ceiling - won't reach this
    learning_rate=0.05,         # slow learning reveals the plateau clearly
    max_depth=4,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_lambda=1,
    eval_metric="auc",
    early_stopping_rounds=40,
    random_state=42,
)
model_es.fit(X_tr, y_tr, eval_set=[(X_tr, y_tr), (X_val, y_val)], verbose=False)

best = model_es.best_iteration
print(f"Stopped at round : {best}")
print(f"Best val AUC     : {model_es.best_score:.4f}")
print(f"Final test AUC   : {roc_auc_score(y_test, model_es.predict_proba(X_test)[:, 1]):.4f}")

# Visualise the plateau
results = model_es.evals_result()
rounds  = range(len(results["validation_0"]["auc"]))
plt.figure(figsize=(10, 4))
plt.plot(rounds, results["validation_0"]["auc"], color="#1976D2", label="Training AUC", linewidth=1.5)
plt.plot(rounds, results["validation_1"]["auc"], color="#43A047", label="Validation AUC", linewidth=1.5)
plt.axvline(best, color="#E53935", linestyle="--", label=f"Best round: {best}")
plt.fill_betweenx([0.5, 1.0], best, len(rounds), alpha=0.05, color="red", label="Overfit zone")
plt.xlabel("Round"); plt.ylabel("AUC")
plt.title("Early Stopping: Validation Peaks, Training Keeps Climbing")
plt.legend(); plt.grid(alpha=0.3); plt.tight_layout(); plt.show()
```

The red shaded area is the "overfit zone" - rounds after the validation plateau where you're just memorising the training set. The gap between training and validation AUC in that region is pure overfitting.

**Try this:** Change `early_stopping_rounds=40` to `5`. The model stops much earlier, often before convergence, because 5 rounds isn't enough buffer to survive the natural noise in validation scores. Compare the test AUC to the `40`-round result.

---

## Putting It All Together: A Practical Configuration Guide

After running all these experiments, here's how I now think about each parameter when starting a new clinical dataset:

| Parameter | Start here | When to change |
|---|---|---|
| `n_estimators` | 2000 + early stopping | Let early stopping find the right count |
| `learning_rate` | 0.05 | Lower if overfitting; 0.1 is fine for quick experiments |
| `max_depth` | 4 | Increase to 6 if underfitting; decrease to 3 if overfitting |
| `subsample` | 0.8 | Rarely needs changing; try 0.6 for small datasets |
| `colsample_bytree` | 0.8 | Same as subsample |
| `reg_lambda` | 1 (default) | Increase to 3–10 if gap > 0.05 |
| `gamma` | 0 | Try 1–5 if the model still overfits after lambda tuning |

The most important thing I learned: **always watch the train-val gap, not just the test score**. A model with val AUC 0.83 and gap 0.02 will generalise to a new hospital better than one with val AUC 0.85 and gap 0.12.

---

## What's Next

Day 3 covers LightGBM - Microsoft's gradient boosting library that often trains 10–20x faster than XGBoost on large datasets. The key algorithmic differences (leaf-wise growth, GOSS, EFB) are worth understanding because they explain *when* LightGBM wins and when XGBoost is still the better choice.
