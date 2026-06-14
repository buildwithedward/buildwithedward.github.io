---
layout: single
title: "Feature Engineering Pipelines: Building Preprocessing That Can't Leak"
date: 2026-06-12
categories: [machine-learning]
tags: [sklearn, pipeline, column-transformer, feature-engineering, custom-transformer, clinical-ml]
excerpt: "Scaling your test data with statistics from test data is a bug. So is fitting your imputer on training + test combined. sklearn Pipelines prevent both - here's how to build one that handles mixed clinical data types cleanly."
---

## Introduction

I've made both of these mistakes: scaling the test set using the test set's own mean and standard deviation, and fitting the imputer on the full dataset before the train/test split. Both are forms of data leakage - you're giving the model a sneak preview of test information during training, which inflates evaluation metrics and causes models to underperform in production.

The fix is a sklearn `Pipeline`: a container that chains all preprocessing steps together and ensures everything is fitted on training data only, then applied consistently to test data. Once built, it also makes deployment much cleaner - you pass raw patient data in, the pipeline transforms it, the model predicts.

---

## The Problem: Preprocessing Outside a Pipeline

Let me show the leakage first.

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OrdinalEncoder, OneHotEncoder
from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.metrics import roc_auc_score
import xgboost as xgb

np.random.seed(42)
N = 4000

age             = np.random.normal(65, 15, N).clip(18, 95)
creatinine      = np.random.exponential(1.2, N).clip(0.5, 12)
bmi             = np.random.normal(28, 6, N).clip(16, 55)
num_comorbid    = np.random.poisson(2.5, N).clip(0, 8)
los_days        = np.random.exponential(5, N).clip(1, 30)
prev_admissions = np.random.poisson(1.0, N).clip(0, 6)
department      = np.random.choice(["cardiology", "nephrology", "general"], N)

# Inject 5% missing values in creatinine (realistic for EHR data)
missing_mask = np.random.random(N) < 0.05
creatinine[missing_mask] = np.nan

logit = (-3.5 + 0.025*age + 0.30*np.nan_to_num(creatinine, nan=1.2) - 0.02*bmi
         + 0.40*num_comorbid + 0.08*los_days + 0.50*prev_admissions
         + np.random.normal(0, 0.5, N))
readmission_30d = (1 / (1 + np.exp(-logit)) > 0.5).astype(int)

df = pd.DataFrame({
    "age": age, "creatinine_mg_dl": creatinine, "bmi": bmi,
    "num_comorbidities": num_comorbid, "los_days": los_days,
    "prev_admissions": prev_admissions, "department": department,
})
y = readmission_30d

# ❌ WRONG: impute and scale BEFORE splitting - test data leaks into training statistics
imputer = SimpleImputer(strategy="median")
scaler  = StandardScaler()
df_imputed = pd.DataFrame(imputer.fit_transform(df.select_dtypes(include=[np.number])),
                           columns=df.select_dtypes(include=[np.number]).columns)
df_imputed = scaler.fit_transform(df_imputed)

X_train_leak, X_test_leak, y_train, y_test = train_test_split(df_imputed, y, test_size=0.2, random_state=42)
m_leak = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, eval_metric="logloss", random_state=42)
m_leak.fit(X_train_leak, y_train, verbose=False)
auc_leak = roc_auc_score(y_test, m_leak.predict_proba(X_test_leak)[:, 1])
print(f"❌ Leaky preprocessing AUC:  {auc_leak:.4f}  (inflated - test data was seen during fit)")
```

The inflated AUC here is subtle - on a large clean dataset the difference might be small. But on small clinical datasets with noisy features, this leakage can make a mediocre model look production-ready.

---

## ColumnTransformer: Different Preprocessing for Different Feature Types

Clinical datasets have mixed feature types: continuous lab values, integer counts, and categorical labels. They need different preprocessing - you wouldn't one-hot encode creatinine, and you wouldn't standardise department names.

Think of `ColumnTransformer` like a mail sorting room: each package (feature column) gets routed to the right processing station based on its type - fragile packages to the careful handler, bulk mail to the fast machine, international packages to the customs specialist.

```python
numeric_features     = ["age", "creatinine_mg_dl", "bmi", "los_days"]
count_features       = ["num_comorbidities", "prev_admissions"]
categorical_features = ["department"]

numeric_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),  # fill missing lab values with median
    ("scaler",  StandardScaler()),                  # mean=0, std=1 for gradient-based models
])

count_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),  # counts rarely missing; mode is safe
    # no scaling - XGBoost doesn't need it, and counts have natural meaning
])

categorical_pipeline = Pipeline([
    ("imputer",  SimpleImputer(strategy="most_frequent")),
    ("encoder",  OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
])

preprocessor = ColumnTransformer([
    ("numeric",      numeric_pipeline,      numeric_features),
    ("counts",       count_pipeline,        count_features),
    ("categorical",  categorical_pipeline,  categorical_features),
])
```

**Try this:** Replace `OneHotEncoder` with `OrdinalEncoder` for `department`. For XGBoost this sometimes works just as well (the tree can learn the right splits regardless of integer encoding), but for linear models it implies an ordering that doesn't exist. Knowing which encoder to use for which downstream model is an important habit.

---

## Wrapping Everything in a Pipeline

Once you have the preprocessor, wrap it with the model into a single `Pipeline`. This is the important step - now `.fit()` on training data sets all parameters, and `.predict()` on test data applies the same transformations without any leakage.

```python
X = df.copy()
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

full_pipeline = Pipeline([
    ("preprocessing", preprocessor),
    ("model", xgb.XGBClassifier(
        n_estimators=200, learning_rate=0.1, max_depth=4,
        eval_metric="logloss", random_state=42
    )),
])

full_pipeline.fit(X_train, y_train)
auc_clean = roc_auc_score(y_test, full_pipeline.predict_proba(X_test)[:, 1])

print(f"✅ Clean pipeline AUC: {auc_clean:.4f}  (no leakage - test data never touched during fit)")
print()
# Show that it handles raw data directly - no manual preprocessing at inference
new_patient = pd.DataFrame([{
    "age": 72, "creatinine_mg_dl": np.nan,  # missing creatinine - handled automatically
    "bmi": 31, "num_comorbidities": 4, "los_days": 8,
    "prev_admissions": 2, "department": "nephrology"
}])
risk = full_pipeline.predict_proba(new_patient)[0, 1]
print(f"New patient readmission risk: {risk:.1%}")
print("(creatinine was NaN - the pipeline imputed it internally)")
```

---

## Custom Transformers: Adding Domain Logic

Sometimes you need a transformation that sklearn doesn't provide out of the box. For clinical data, a common one is a "risk score" feature - something like the Charlson Comorbidity Index that clinicians already use. You want to encode that domain knowledge as a feature.

Custom transformers inherit from `BaseEstimator` and `TransformerMixin`. They fit into any Pipeline exactly like a built-in transformer.

```python
class ClinicalRiskScoreTransformer(BaseEstimator, TransformerMixin):
    """
    Adds a derived clinical risk feature: a simple weighted combination
    of high-signal EHR fields, mimicking how clinicians think about risk.
    """
    def fit(self, X, y=None):
        # Nothing to learn from data - this is a deterministic rule
        return self

    def transform(self, X):
        X_ = pd.DataFrame(X).copy()
        # Simplified risk heuristic (in practice, use validated clinical scores)
        risk_score = (
            (X_.iloc[:, 0] > 70).astype(float) * 1.5  +  # age > 70
            (X_.iloc[:, 1] > 2.0).astype(float) * 2.0 +  # creatinine > 2.0 mg/dL (CKD)
            (X_.iloc[:, 4] > 7).astype(float) * 1.0      # los_days > 7
        )
        X_["clinical_risk_score"] = risk_score
        return X_.values

# Build a pipeline with the custom transformer
custom_pipeline = Pipeline([
    ("imputer",       SimpleImputer(strategy="median")),
    ("risk_scorer",   ClinicalRiskScoreTransformer()),
    ("scaler",        StandardScaler()),
    ("model",         xgb.XGBClassifier(n_estimators=200, learning_rate=0.1,
                                         max_depth=4, eval_metric="logloss", random_state=42)),
])

X_numeric = X_train[numeric_features + count_features]
X_numeric_test = X_test[numeric_features + count_features]

custom_pipeline.fit(X_numeric, y_train)
auc_custom = roc_auc_score(y_test, custom_pipeline.predict_proba(X_numeric_test)[:, 1])
print(f"Pipeline with custom risk feature AUC: {auc_custom:.4f}")
```

**Try this:** Change the clinical risk score thresholds - use `age > 80` instead of `age > 70`, or `creatinine > 1.5` instead of `> 2.0`. Does the pipeline AUC change? This is how you test whether your domain knowledge is actually adding signal the model couldn't learn on its own.

---

## Cross-Validation With a Pipeline

The final benefit of wrapping everything in a pipeline: cross-validation works correctly. When sklearn's `cross_val_score` splits the data, it calls `.fit()` on the training fold - which fits the imputer, scaler, and encoder on that fold only. The validation fold is transformed using those fitted parameters, with no leakage.

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
cv_scores = cross_val_score(full_pipeline, X, y, cv=cv, scoring="roc_auc", n_jobs=-1)

print("5-Fold Cross-Validation AUC (no leakage, each fold fitted independently):")
for i, score in enumerate(cv_scores):
    print(f"  Fold {i+1}: {score:.4f}")
print(f"  Mean: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")
```

The standard deviation across folds tells you how stable the model is. A large spread (e.g., ±0.05) means the model's performance is sensitive to which patients happen to be in the training set - which in clinical deployment means it might work well in one hospital cohort and poorly in another.

---

## What's Next

Day 8 covers synthetic data generation - SDV, CTGAN, and TVAE. In healthcare, where you often can't share real patient data for model development, being able to generate synthetic data that preserves statistical properties without exposing real patients is an increasingly important skill.
