---
layout: single
title: "CatBoost: How It Handles Categorical Features Without Leaking the Answer"
date: 2026-06-09
categories: [machine-learning]
tags: [catboost, gradient-boosting, categorical-features, ordered-boosting, target-encoding, clinical-ml]
excerpt: "The standard way to encode categorical features (target encoding) secretly leaks information from the labels into the training data. CatBoost fixes this with ordered boosting. Here's what that means and why it matters for clinical datasets with ICD codes and drug names."
---

## Introduction

Clinical datasets are full of categorical features - diagnosis codes (ICD-10), drug names, hospital departments, procedure codes. The standard way to handle these is **target encoding**: replace each category with the average outcome rate for patients in that category. Sounds reasonable, right?

There's a subtle problem. If you compute "average readmission rate for ICD code E11.9 (Type 2 Diabetes)" using the *same patients you're training on*, the encoding secretly contains information about their actual outcomes. The model learns from features that already know the answer - a kind of circular reasoning. On training data it looks great; on new patients it doesn't hold up.

CatBoost solves this with *ordered boosting*, and along the way it makes categorical features first-class citizens instead of something you have to preprocess away.

---

## The Target Leakage Problem, Made Concrete

Let me show you the problem before showing the fix.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score
from sklearn.preprocessing import OrdinalEncoder
import catboost as cb
import xgboost as xgb

np.random.seed(42)
N = 4000

# Clinical dataset with categorical features
department  = np.random.choice(["cardiology", "nephrology", "oncology",
                                  "general_medicine", "pulmonology"], N,
                                 p=[0.25, 0.20, 0.15, 0.30, 0.10])
icd_chapter = np.random.choice(["circulatory", "endocrine", "neoplasm",
                                   "respiratory", "digestive"], N,
                                  p=[0.30, 0.20, 0.15, 0.20, 0.15])
prev_admissions = np.random.poisson(1.0, N).clip(0, 6)
los_days        = np.random.exponential(5, N).clip(1, 30)
age             = np.random.normal(65, 15, N).clip(18, 95)

# Readmission depends on department AND other features
dept_risk = {"cardiology": 0.6, "nephrology": 0.8, "oncology": 0.5,
             "general_medicine": 0.2, "pulmonology": 0.4}
icd_risk  = {"circulatory": 0.5, "endocrine": 0.3, "neoplasm": 0.6,
             "respiratory": 0.4, "digestive": 0.2}

base_risk = np.array([dept_risk[d] for d in department]) * 0.5 + \
            np.array([icd_risk[i]  for i in icd_chapter]) * 0.3 + \
            prev_admissions * 0.1 - 0.5 + np.random.randn(N) * 0.3

readmission_30d = (base_risk > 0.5).astype(int)

df = pd.DataFrame({
    "department": department, "icd_chapter": icd_chapter,
    "prev_admissions": prev_admissions, "los_days": los_days, "age": age,
})

X_train, X_test, y_train, y_test = train_test_split(
    df, readmission_30d, test_size=0.2, random_state=42
)

# Naive target encoding - computed on the SAME training data = leakage
for col in ["department", "icd_chapter"]:
    means = y_train.groupby(X_train[col]).mean()
    X_train[col + "_enc"] = X_train[col].map(means)
    X_test[col + "_enc"]  = X_test[col].map(means).fillna(y_train.mean())

X_train_enc = X_train.drop(["department", "icd_chapter"], axis=1)
X_test_enc  = X_test.drop(["department", "icd_chapter"],  axis=1)

m_naive = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                              eval_metric="logloss", random_state=42)
m_naive.fit(X_train_enc, y_train, verbose=False)

train_auc = roc_auc_score(y_train, m_naive.predict_proba(X_train_enc)[:, 1])
test_auc  = roc_auc_score(y_test,  m_naive.predict_proba(X_test_enc)[:, 1])
print(f"Naive target encoding:  train AUC={train_auc:.4f}, test AUC={test_auc:.4f}")
print(f"Gap (overfit signal):   {train_auc - test_auc:.4f}")
print("Note: train AUC is inflated - the encoding leaked outcome information")
```

The inflated training AUC is the leakage signal. The encoding was computed on the same patients being trained on, so the model partially memorises patient outcomes through the encoding itself.

**Try this:** Compute the target encoding on only the *first half* of training data and apply it to the second half, then train. The gap will shrink. This is leave-out encoding - a manual approximation of what CatBoost does automatically.

---

## How CatBoost Fixes It: Ordered Boosting

CatBoost uses a technique called **ordered boosting**. The analogy: imagine a hospital that trains its risk model only using *historical* patient records - when deciding how to encode a patient's department, you only use outcomes from patients who were admitted *before* this patient in time.

More precisely: CatBoost shuffles the training data into a random order, then for each patient, computes their category statistics using only the patients who appeared earlier in the order. No patient's outcome is ever used to encode itself. This is repeated with multiple random orderings to reduce variance.

The result: no leakage, no manual cross-validation tricks required. Just pass your categorical columns directly.

```python
# CatBoost - pass raw categorical columns, no encoding needed
cat_features = ["department", "icd_chapter"]

X_train_cat = X_train[["department", "icd_chapter", "prev_admissions", "los_days", "age"]].copy()
X_test_cat  = X_test[["department",  "icd_chapter", "prev_admissions", "los_days", "age"]].copy()

# CatBoost requires categoricals as strings (not ints)
for col in cat_features:
    X_train_cat[col] = X_train_cat[col].astype(str)
    X_test_cat[col]  = X_test_cat[col].astype(str)

model_cb = cb.CatBoostClassifier(
    iterations=300,
    learning_rate=0.1,
    depth=4,
    cat_features=cat_features,
    eval_metric="AUC",
    random_seed=42,
    verbose=0,
)
model_cb.fit(X_train_cat, y_train, eval_set=(X_test_cat, y_test))

train_auc_cb = roc_auc_score(y_train, model_cb.predict_proba(X_train_cat)[:, 1])
test_auc_cb  = roc_auc_score(y_test,  model_cb.predict_proba(X_test_cat)[:, 1])
print(f"\nCatBoost (ordered boosting):  train AUC={train_auc_cb:.4f}, test AUC={test_auc_cb:.4f}")
print(f"Gap:                           {train_auc_cb - test_auc_cb:.4f}")
print("No manual encoding. No leakage. Categories passed directly.")
```

**Try this:** Add a new categorical feature with high cardinality - say, a `doctor_id` with 200 unique values. Naive target encoding will dramatically overfit (each doctor appears few times, their average is noisy). CatBoost handles this gracefully because it averages across multiple orderings.

---

## Feature Importance in CatBoost

CatBoost has richer feature importance options than XGBoost. The most useful for clinical work is **SHAP-based importance** (the same SHAP we'll cover in depth on Day 5), but CatBoost also has its own native importance types.

```python
# PredictionValuesChange: average change in prediction when this feature changes
importance_df = pd.DataFrame({
    "feature":    model_cb.feature_names_,
    "importance": model_cb.get_feature_importance(),
}).sort_values("importance", ascending=True)

importance_df.plot(
    kind="barh", x="feature", y="importance",
    figsize=(8, 4), color="#FF6F00", legend=False,
    title="CatBoost Feature Importance - 30-Day Readmission"
)
plt.xlabel("Importance score")
plt.tight_layout()
plt.show()

# Which categories within a feature matter most?
print("\nTop department risk ranking:")
dept_stats = pd.DataFrame({
    "department":  list(dept_risk.keys()),
    "true_risk":   list(dept_risk.values()),
}).sort_values("true_risk", ascending=False)
print(dept_stats.to_string(index=False))
```

---

## XGBoost vs LightGBM vs CatBoost: The Decision Guide

After building all three this week, here's my honest take on when each wins:

| Situation | Best choice | Why |
|---|---|---|
| Small dataset (<10K), no categoricals | XGBoost | Most stable on small data |
| Large dataset (>50K), few categoricals | LightGBM | Speed advantage is real |
| Dataset has high-cardinality categoricals | CatBoost | Ordered encoding eliminates leakage |
| ICD codes, drug names, hospital names | CatBoost | Native handling, no preprocessing |
| Need SHAP explainability | All three support it | TreeExplainer works on all |
| Competition / general default | LightGBM or XGBoost | Faster hyperparameter search |

For a real clinical readmission model with ICD codes and department information, I'd start with CatBoost. The leakage problem in naive encoding is real, and CatBoost's native handling is much simpler than the correct cross-validation-based encoding workaround you'd need with the other two.

---

## What's Next

Day 5 is SHAP from scratch - moving from `feature_importances_` (which tells you which features were *used* most) to SHAP values (which tell you *how much* each feature changed each individual patient's prediction). The difference matters a lot when you're trying to explain a specific readmission alert to a physician.
