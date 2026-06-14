---
layout: single
title: "Class Imbalance in Clinical ML: SMOTE, ADASYN, and Why Accuracy Is Lying to You"
date: 2026-06-11
categories: [machine-learning]
tags: [class-imbalance, smote, adasyn, borderline-smote, cost-sensitive, imbalanced-learn, clinical-ml]
excerpt: "A model that predicts 'no readmission' for every patient can claim 88% accuracy. Here's how SMOTE, ADASYN, BorderlineSMOTE, and cost-sensitive learning actually fix the problem - and which metric to watch instead of accuracy."
header:
  teaser: /assets/img/banner-class-imbalance.png
---

## Introduction

Early in a project I was proud of a readmission model with 88% accuracy. My colleague asked how it performed on actual readmission cases. I ran the numbers: it was correctly identifying only 12% of patients who would be readmitted. It was effectively just predicting "no readmission" for almost everyone and calling that a win because the dataset had only 12% positives.

This is the class imbalance problem, and it's especially dangerous in clinical settings because the rare class - the sick patients, the adverse events, the early disease - is exactly the one you care most about getting right.

This post walks through four approaches to fix it, with experiments showing what each one actually does.

---

## First: See the Problem Clearly

Before reaching for any technique, it helps to see what a naive model does on imbalanced data - and why the default accuracy metric hides it.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.metrics import (classification_report, roc_auc_score,
                              average_precision_score, confusion_matrix)
from sklearn.linear_model import LogisticRegression
import xgboost as xgb
from imblearn.over_sampling import SMOTE, ADASYN, BorderlineSMOTE

np.random.seed(42)
N = 5000

age             = np.random.normal(65, 15, N).clip(18, 95)
creatinine      = np.random.exponential(1.2, N).clip(0.5, 12)
bmi             = np.random.normal(28, 6, N).clip(16, 55)
num_comorbid    = np.random.poisson(2.0, N).clip(0, 8)
los_days        = np.random.exponential(4, N).clip(1, 30)
prev_admissions = np.random.poisson(0.5, N).clip(0, 6)

# Make it severely imbalanced: ~10% positive rate
logit = (-4.5 + 0.020*age + 0.25*creatinine - 0.01*bmi
         + 0.35*num_comorbid + 0.06*los_days + 0.45*prev_admissions
         + np.random.normal(0, 0.6, N))
readmission_30d = (1 / (1 + np.exp(-logit)) > 0.5).astype(int)

print(f"Readmission rate: {readmission_30d.mean():.1%}  ({readmission_30d.sum()} positives out of {N})")

X = pd.DataFrame({
    "age": age, "creatinine_mg_dl": creatinine, "bmi": bmi,
    "num_comorbidities": num_comorbid, "los_days": los_days,
    "prev_admissions": prev_admissions,
})
y = readmission_30d
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Naive model - no imbalance handling
model_naive = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                                  eval_metric="logloss", random_state=42)
model_naive.fit(X_train, y_train, verbose=False)
y_pred_naive = model_naive.predict(X_test)
y_prob_naive = model_naive.predict_proba(X_test)[:, 1]

accuracy  = (y_pred_naive == y_test).mean()
recall_pos = (y_pred_naive[y_test == 1] == 1).mean()  # did we catch the readmissions?

print(f"\nNaive model (no imbalance handling):")
print(f"  Accuracy:                 {accuracy:.1%}  ← sounds great!")
print(f"  Recall on readmissions:   {recall_pos:.1%}  ← we're missing this many actual cases")
print(f"  ROC-AUC:                  {roc_auc_score(y_test, y_prob_naive):.4f}")
print(f"  Average Precision (AUPRC):{average_precision_score(y_test, y_prob_naive):.4f}")
print()
print("Confusion matrix:")
cm = confusion_matrix(y_test, y_pred_naive)
print(f"  True negatives:  {cm[0,0]}   False positives: {cm[0,1]}")
print(f"  False negatives: {cm[1,0]}   True positives:  {cm[1,1]}")
```

The confusion matrix tells the real story. Most of the "accuracy" comes from correctly predicting the majority class (no readmission). The patients who were actually readmitted - the ones a care team would act on - are being missed at an alarming rate.

**Try this:** Swap XGBoost for a `LogisticRegression()` (no class weight). The recall on readmissions will drop further. Now try `LogisticRegression(class_weight='balanced')` - a simple fix that already helps a lot.

---

## SMOTE: Creating Plausible New Cases

SMOTE (Synthetic Minority Over-sampling Technique) doesn't just copy existing positive cases - it creates new synthetic ones by interpolating between existing ones.

The analogy: imagine you have 10 examples of a rare disease presentation in your textbook. Instead of photocopying those 10 pages, you blend adjacent examples: "Patient A had creatinine 3.2 and 4 comorbidities; Patient B had creatinine 2.8 and 5 comorbidities. Synthesise a new patient at creatinine 3.0 and 4.5 comorbidities." The result is a richer, more varied representation of what "high-risk" looks like.

```python
smote = SMOTE(random_state=42)
X_smote, y_smote = smote.fit_resample(X_train, y_train)

print(f"Before SMOTE: {y_train.sum()} positives / {len(y_train)} total ({y_train.mean():.1%})")
print(f"After SMOTE:  {y_smote.sum()} positives / {len(y_smote)} total ({y_smote.mean():.1%})")

model_smote = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                                  eval_metric="logloss", random_state=42)
model_smote.fit(X_smote, y_smote, verbose=False)
y_prob_smote = model_smote.predict_proba(X_test)[:, 1]

# Use a lower threshold: default 0.5 is calibrated for balanced data
threshold = 0.3
y_pred_smote = (y_prob_smote >= threshold).astype(int)

print(f"\nSMOTE model (threshold={threshold}):")
print(classification_report(y_test, y_pred_smote, target_names=["No Readmit", "Readmit"]))
print(f"ROC-AUC: {roc_auc_score(y_test, y_prob_smote):.4f}")
```

Notice I used `threshold=0.3` instead of the default `0.5`. After SMOTE, the model's probability scale shifts - it's more willing to predict positive. The right threshold depends on the clinical context: how costly is a false alarm vs a missed case?

**Try this:** Try `SMOTE(k_neighbors=3)` vs `k_neighbors=10`. Fewer neighbours creates more localised synthetic patients (close to existing ones); more neighbours blends further across the feature space. On a noisy dataset, `k_neighbors=3` tends to be safer.

---

## ADASYN: Focusing on the Hard Cases

ADASYN (Adaptive Synthetic Sampling) is an improvement on SMOTE. Instead of generating the same number of synthetic samples near every positive patient, it generates *more* synthetic samples near the positive patients that are hardest to classify - the ones surrounded by negative cases.

The analogy: a medical school creates more practice cases around the disease presentations that students most often confuse with other conditions. The easy-to-recognise presentations get fewer examples; the ambiguous borderline cases get many more.

```python
adasyn = ADASYN(random_state=42)
X_adasyn, y_adasyn = adasyn.fit_resample(X_train, y_train)

model_adasyn = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                                   eval_metric="logloss", random_state=42)
model_adasyn.fit(X_adasyn, y_adasyn, verbose=False)
y_prob_adasyn = model_adasyn.predict_proba(X_test)[:, 1]
y_pred_adasyn = (y_prob_adasyn >= 0.3).astype(int)

print("ADASYN model:")
print(classification_report(y_test, y_pred_adasyn, target_names=["No Readmit", "Readmit"]))
print(f"ROC-AUC: {roc_auc_score(y_test, y_prob_adasyn):.4f}")
print(f"AUPRC:   {average_precision_score(y_test, y_prob_adasyn):.4f}")
```

ADASYN generally performs similarly to SMOTE but can excel when the positive class has distinct "easy" and "hard" subgroups. In clinical data, "easy" cases might be patients with extreme lab values; "hard" cases are borderline presentations where readmission risk is genuinely ambiguous.

---

## Cost-Sensitive Learning: The Simplest Fix That Often Wins

All the oversampling approaches (SMOTE, ADASYN, BorderlineSMOTE) modify the training data. There's a simpler alternative: tell the model directly that getting a readmission case wrong is more costly than getting a no-readmission case wrong.

XGBoost's `scale_pos_weight` does exactly this. Setting it to 5 means: "treat each positive case as if it were 5 patients." The model will try harder not to miss them.

```python
# scale_pos_weight = (number of negatives) / (number of positives)
# This is the "perfectly balanced" version; you can tune it
n_neg = (y_train == 0).sum()
n_pos = (y_train == 1).sum()
auto_weight = n_neg / n_pos

print(f"Class ratio (neg/pos): {auto_weight:.1f}")
print(f"Setting scale_pos_weight = {auto_weight:.1f} balances the classes\n")

for spw, label in [(1, "No weighting (default)"),
                   (auto_weight, "Auto-balanced"),
                   (auto_weight * 2, "2x emphasis on positives")]:
    m = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                           scale_pos_weight=spw, eval_metric="logloss", random_state=42)
    m.fit(X_train, y_train, verbose=False)
    prob = m.predict_proba(X_test)[:, 1]
    pred = (prob >= 0.5).astype(int)
    tp   = ((pred == 1) & (y_test == 1)).sum()
    fn   = ((pred == 0) & (y_test == 1)).sum()
    recall = tp / (tp + fn) if (tp + fn) > 0 else 0
    prec   = tp / pred.sum() if pred.sum() > 0 else 0
    print(f"spw={spw:<6.1f}  {label:<35}  recall={recall:.2f}  precision={prec:.2f}  AUC={roc_auc_score(y_test, prob):.4f}")
```

**Try this:** Set `scale_pos_weight = auto_weight * 5` - very aggressive weighting. Recall (catching actual readmissions) will approach 0.95, but precision (how often your alerts are real) will drop sharply. This is the clinical tradeoff: catching more real cases means more false alarms. The right balance depends on the cost of each type of error in your deployment context.

---

## Comparison: Which Technique Actually Wins?

```python
results = []

for name, X_res, y_res, threshold in [
    ("Naive (no handling)",        X_train,   y_train,   0.5),
    ("SMOTE",                      X_smote,   y_smote,   0.3),
    ("ADASYN",                     X_adasyn,  y_adasyn,  0.3),
    ("scale_pos_weight (auto)",    X_train,   y_train,   0.5),
]:
    spw = auto_weight if "scale_pos_weight" in name else 1
    m = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                           scale_pos_weight=spw, eval_metric="logloss", random_state=42)
    m.fit(X_res, y_res, verbose=False)
    prob = m.predict_proba(X_test)[:, 1]
    pred = (prob >= threshold).astype(int)
    tp  = ((pred == 1) & (y_test == 1)).sum()
    fn  = ((pred == 0) & (y_test == 1)).sum()
    fp  = ((pred == 1) & (y_test == 0)).sum()
    rec = tp / (tp + fn) if (tp + fn) > 0 else 0
    pre = tp / (tp + fp) if (tp + fp) > 0 else 0
    f1  = 2 * pre * rec / (pre + rec) if (pre + rec) > 0 else 0
    auc = roc_auc_score(y_test, prob)
    results.append({"Method": name, "Recall": rec, "Precision": pre, "F1": f1, "AUC": auc})

print(pd.DataFrame(results).to_string(index=False))
```

In practice, cost-sensitive learning (`scale_pos_weight`) is often my first choice - it requires no preprocessing, works with any GBDT, and the tradeoff between recall and precision is directly tunable. SMOTE is worth trying when you have a very small minority class (< 1%) and need more synthetic examples to learn from.

The one metric I always watch with imbalanced data: **Recall on the positive class**. That's the answer to "how many readmissions are we actually catching?" Accuracy hides this; ROC-AUC partially captures it; but the confusion matrix shows it directly.

---

## What's Next

Day 7 covers feature engineering pipelines - ColumnTransformer, custom transformers, and how to bundle all the preprocessing (scaling, encoding, imputation) into a single sklearn Pipeline so your model is never trained on transformed data and tested on raw data.
