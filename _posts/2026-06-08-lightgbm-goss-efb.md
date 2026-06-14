---
layout: single
title: "LightGBM: Why It's Faster Than XGBoost and When That Matters"
date: 2026-06-08
categories: [machine-learning]
tags: [lightgbm, gradient-boosting, goss, efb, leaf-wise, clinical-ml, python]
excerpt: "LightGBM trains 10–20x faster than XGBoost on large datasets. This post explains the three algorithmic tricks behind that speed - GOSS, EFB, and leaf-wise growth - with experiments to make each one observable."
---

## Introduction

If XGBoost is the reliable workhorse of tabular ML, LightGBM is its faster younger sibling. On a dataset with 100,000 patients and 50 features, XGBoost might take 10 minutes to train while LightGBM finishes in under a minute. For a long time I just accepted this speed difference and moved on - but the *why* turned out to be interesting, and understanding it helped me decide which tool to reach for.

LightGBM gets its speed from three algorithmic changes. This post covers each one with a concrete experiment so you can see the tradeoff directly.

---

## Level-Wise vs Leaf-Wise Growth: A Different Building Strategy

When XGBoost builds a tree, it grows it one *level* at a time - all nodes at depth 1, then all nodes at depth 2, and so on. Think of building a house by completing one full floor before starting the next.

LightGBM grows trees *leaf-wise* instead. It looks at all the current leaves and asks: "which single leaf, if split, would reduce the error the most?" Then it splits that one leaf, regardless of which level it's on. It's like renovating a house by always tackling whichever room is in the worst state first.

The result: LightGBM's trees are often deeper and more asymmetric than XGBoost's, but they achieve lower loss with fewer nodes. The tradeoff is that leaf-wise growth is more prone to overfitting on small datasets - the tree digs deep into the highest-error region, which might just be noise.

```python
import lightgbm as lgb
import xgboost as xgb
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import time
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score

np.random.seed(42)
N = 20_000   # larger dataset to make timing differences visible

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

# Time both on identical conditions
t0 = time.time()
xgb_model = xgb.XGBClassifier(n_estimators=300, learning_rate=0.1, max_depth=6,
                                subsample=0.8, eval_metric="logloss", random_state=42)
xgb_model.fit(X_train, y_train, verbose=False)
xgb_time = time.time() - t0
xgb_auc  = roc_auc_score(y_test, xgb_model.predict_proba(X_test)[:, 1])

t0 = time.time()
lgb_model = lgb.LGBMClassifier(n_estimators=300, learning_rate=0.1, num_leaves=63,
                                 subsample=0.8, random_state=42, verbose=-1)
lgb_model.fit(X_train, y_train)
lgb_time = time.time() - t0
lgb_auc  = roc_auc_score(y_test, lgb_model.predict_proba(X_test)[:, 1])

print(f"{'Library':<12} {'AUC':>8} {'Train time':>12}")
print(f"{'XGBoost':<12} {xgb_auc:>8.4f} {xgb_time:>10.1f}s")
print(f"{'LightGBM':<12} {lgb_auc:>8.4f} {lgb_time:>10.1f}s")
print(f"\nLightGBM speedup: {xgb_time / lgb_time:.1f}x faster")
```

**Try this:** Change `N = 200_000` and re-run. The speedup multiplier gets larger as the dataset grows - this is where LightGBM's algorithmic advantages compound.

---

## GOSS: Sampling Patients Smartly, Not Randomly

Normally, training a tree means looking at every training patient to find the best split. With 100,000 patients, that's a lot of work - most of which isn't very informative, because patients the model already predicts well don't tell you much about where to improve.

GOSS (Gradient-based One-Side Sampling) exploits this insight. It works like a teacher reviewing exam papers: you definitely re-read the papers with the highest errors (the students who got things very wrong), and you randomly sample a smaller fraction of the easy papers (just to make sure you haven't forgotten those patterns). Papers where the student got a perfect score get skipped entirely.

Technically: patients with large gradients (the model is very wrong about them) are always kept. A random sample of small-gradient patients is kept. Everyone else is skipped. The result: you train on a fraction of the data but focus your effort on the patients where improvement is possible.

```python
# LightGBM exposes GOSS via the data_sample_strategy parameter
# (older versions: boosting_type is "goss")

lgb_full = lgb.LGBMClassifier(
    n_estimators=300, learning_rate=0.1, num_leaves=63,
    data_sample_strategy="bagging",  # random sampling (like XGBoost's subsample)
    subsample=0.8, random_state=42, verbose=-1
)
lgb_goss = lgb.LGBMClassifier(
    n_estimators=300, learning_rate=0.1, num_leaves=63,
    data_sample_strategy="goss",     # smart sampling: keep high-gradient patients
    top_rate=0.2,                    # keep top 20% highest-gradient patients
    other_rate=0.1,                  # randomly sample 10% of the rest
    random_state=42, verbose=-1
)

for name, m in [("Bagging (random 80%)", lgb_full), ("GOSS (gradient-aware)", lgb_goss)]:
    t0 = time.time()
    m.fit(X_train, y_train)
    elapsed = time.time() - t0
    auc = roc_auc_score(y_test, m.predict_proba(X_test)[:, 1])
    print(f"{name:<30}  AUC={auc:.4f}  time={elapsed:.2f}s")
```

GOSS typically achieves similar AUC to full sampling but trains faster because it processes fewer patients per split-finding step. The tradeoff: it can miss rare but important patient subgroups that happen to have low gradients (because the model accidentally learned them from noise in early rounds).

**Try this:** Lower `top_rate=0.05, other_rate=0.02` - aggressive GOSS that only looks at 7% of patients per round. Training gets faster, but AUC may drop on this noisy clinical dataset because important rare patterns get missed.

---

## EFB: Merging Features That Never Conflict

Here's a clever one. In clinical data, you often have features that are never both non-zero at the same time. For example, a patient might have either a high "sepsis_score" or a high "cardiac_risk_score" - but rarely both elevated simultaneously. These features are what LightGBM calls *mutually exclusive*.

EFB (Exclusive Feature Bundling) finds groups of such features and merges them into a single feature. Instead of scanning 50 features to find the best split, LightGBM might scan only 30 "bundled" features. This is a pure speed win with (ideally) no accuracy loss.

The analogy: imagine a hospital form with 50 fields, but you notice that fields 12–18 are from an old respiratory protocol and only one of them ever has a value at a time. Bundle them into one "respiratory_indicator" field. You haven't lost any information - you just read fewer fields.

```python
# EFB is enabled by default in LightGBM; the `max_bin` parameter affects bundle quality

lgb_no_efb = lgb.LGBMClassifier(
    n_estimators=300, learning_rate=0.1, num_leaves=63,
    subsample=0.8,
    max_bin=255,    # default
    random_state=42, verbose=-1
)

# Demonstrate on a dataset with artificially sparse/exclusive features
N2 = 10_000
feature_matrix = np.zeros((N2, 20))
for i in range(0, 20, 4):
    idx = np.random.choice(N2, N2 // 4, replace=False)
    feature_matrix[idx, i:i+4] = np.random.randn(N2 // 4, 4)

X_sparse = pd.DataFrame(feature_matrix, columns=[f"feat_{i}" for i in range(20)])
X_sparse["prev_admissions"] = np.random.poisson(1.0, N2)
X_sparse["age"] = np.random.normal(65, 15, N2).clip(18, 95)
y_sparse = (X_sparse["feat_0"] + X_sparse["feat_4"] + X_sparse["prev_admissions"] * 0.3
             + np.random.randn(N2) * 0.5 > 0.8).astype(int)

Xs_tr, Xs_te, ys_tr, ys_te = train_test_split(X_sparse, y_sparse, test_size=0.2, random_state=42)

t0 = time.time()
lgb_no_efb.fit(Xs_tr, ys_tr)
t_no_efb = time.time() - t0
auc_no_efb = roc_auc_score(ys_te, lgb_no_efb.predict_proba(Xs_te)[:, 1])

print(f"Without EFB awareness: AUC={auc_no_efb:.4f}, time={t_no_efb:.2f}s")
print("EFB is on by default in LightGBM - it found and bundled exclusive features automatically.")
print(f"Feature count: {X_sparse.shape[1]} original → fewer effective bins scanned per split")
```

EFB works automatically - you don't configure it. The benefit shows up most when you have wide sparse datasets (many features, most zero for any given patient), which is common in clinical data derived from EHR systems where most coded diagnoses and procedures are absent for any single patient.

---

## num_leaves: LightGBM's max_depth Equivalent

Because LightGBM grows trees leaf-wise instead of level-wise, it uses `num_leaves` rather than `max_depth` to control tree complexity. `num_leaves` is the maximum number of terminal nodes the tree can have.

A level-wise tree of depth 4 has at most 2⁴ = 16 leaves. A leaf-wise tree can have any number of leaves regardless of depth. Setting `num_leaves=63` is a common default (roughly equivalent to depth 6 in level-wise terms).

```python
print(f"{'num_leaves':<14} {'AUC':>8}  note")
print("-" * 50)
notes = {7: "very shallow (≈depth 3)", 31: "moderate (≈depth 5)",
         63: "LightGBM default", 127: "deeper", 511: "overfitting risk"}
for nl in [7, 31, 63, 127, 511]:
    m = lgb.LGBMClassifier(n_estimators=300, learning_rate=0.1, num_leaves=nl,
                            subsample=0.8, random_state=42, verbose=-1)
    m.fit(X_train, y_train)
    auc = roc_auc_score(y_test, m.predict_proba(X_test)[:, 1])
    print(f"{nl:<14} {auc:>8.4f}  {notes[nl]}")
```

**Try this:** Set `num_leaves=511` on a small dataset (N=1000) and compare train vs test AUC. With enough leaves and few patients, the tree can create one leaf per patient group - perfect training AUC, terrible generalisation.

---

## XGBoost vs LightGBM: When to Use Which

After running these experiments, here's my practical decision rule:

| Situation | Reach for |
|---|---|
| Dataset < 10K rows | XGBoost - LightGBM's leaf-wise growth overfits more on small data |
| Dataset > 50K rows | LightGBM - speed advantage is significant and worth it |
| Lots of categorical features | LightGBM - has native categorical support (Day 4 CatBoost does this better still) |
| Need interpretability (SHAP) | Either - both support `shap.TreeExplainer` |
| Production with tight latency | XGBoost - marginally faster at inference time |
| Hyperparameter tuning budget | LightGBM - faster training = more Optuna trials in the same time |

For our readmission dataset at 5,000 patients, the difference is small. At 500,000 patients, LightGBM is the clear default.

---

## What's Next

Day 4 covers CatBoost - Yandex's gradient boosting library that handles categorical features natively without the target-leakage problem that plagues the usual encoding approaches. If your clinical dataset has ICD codes, drug names, or hospital unit labels as features, CatBoost's approach to these is worth knowing.
