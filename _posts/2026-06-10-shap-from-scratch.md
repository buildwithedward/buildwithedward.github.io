---
layout: single
title: "SHAP From Scratch: From Feature Importance to Per-Patient Explanations"
date: 2026-06-10
categories: [machine-learning]
tags: [shap, explainability, feature-importance, xgboost, clinical-ml, waterfall, beeswarm]
excerpt: "feature_importances_ tells you which features the model used most globally. SHAP tells you why the model made a specific prediction for a specific patient. Here's the difference and how to use both."
---

## Introduction

When I first built a readmission risk model, my team asked: "Why did it flag this patient?" I pointed at the feature importance chart - "prev_admissions is the top feature." They asked again: "Yes, but *this* patient - why is their score 78%?"

Feature importance can't answer that question. It's a global summary across all predictions. SHAP can answer it - it gives each feature a score for each individual prediction that says "this feature pushed this patient's risk up by X%."

This post covers both tools, why the difference matters in clinical settings, and how to build useful visualisations with SHAP.

---

## Why feature_importances_ Is Not Enough

The built-in `feature_importances_` in XGBoost measures how often each feature was used as a split point across all trees. Think of it like measuring a basketball player's contribution by counting how many times they touched the ball - it tells you who was involved, but not whether they helped or hurt, or by how much.

A feature can be used in many splits and still have little net effect on predictions if it splits in conflicting directions in different trees. Conversely, a feature used in few splits might dominate predictions in the cases where it appears.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import shap
import xgboost as xgb
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score

np.random.seed(42)
N = 3000

age             = np.random.normal(65, 15, N).clip(18, 95)
creatinine      = np.random.exponential(1.2, N).clip(0.5, 12)
bmi             = np.random.normal(28, 6, N).clip(16, 55)
num_comorbid    = np.random.poisson(2.5, N).clip(0, 8)
los_days        = np.random.exponential(5, N).clip(1, 30)
prev_admissions = np.random.poisson(1.0, N).clip(0, 6)
heart_rate_bpm  = np.random.normal(78, 15, N).clip(45, 140)

logit = (-3.5 + 0.025*age + 0.30*creatinine - 0.02*bmi
         + 0.40*num_comorbid + 0.08*los_days + 0.50*prev_admissions
         + np.random.normal(0, 0.5, N))
readmission_30d = (1 / (1 + np.exp(-logit)) > 0.5).astype(int)

X = pd.DataFrame({
    "age": age, "creatinine_mg_dl": creatinine, "bmi": bmi,
    "num_comorbidities": num_comorbid, "los_days": los_days,
    "prev_admissions": prev_admissions, "heart_rate_bpm": heart_rate_bpm,
})
y = readmission_30d
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                            eval_metric="logloss", random_state=42)
model.fit(X_train, y_train, verbose=False)

# Standard feature importance
fi = pd.Series(model.feature_importances_, index=X.columns).sort_values()
fig, axes = plt.subplots(1, 2, figsize=(13, 4))
fi.plot(kind="barh", ax=axes[0], color="#90CAF9", title="feature_importances_\n(how often each feature was split on)")
axes[0].set_xlabel("Split frequency")

# Already shows limitation: doesn't tell us direction or magnitude per patient
axes[1].text(0.1, 0.5,
    "feature_importances_ answers:\n'Which features did the model use most?'\n\n"
    "It does NOT answer:\n'How much did creatinine push THIS\npatient's readmission risk up?'",
    fontsize=11, transform=axes[1].transAxes, va="center",
    bbox=dict(boxstyle="round", facecolor="#FFF9C4", alpha=0.8))
axes[1].axis("off")
plt.tight_layout()
plt.show()
```

**Try this:** Add a random noise feature `noise = np.random.randn(N)` to the dataset and retrain. Check where it lands in `feature_importances_` - it might rank surprisingly high just because it appeared in many splits, even though it has no real predictive value.

---

## What SHAP Values Actually Are

SHAP comes from game theory - specifically from a concept called Shapley values, which answers the question: "In a group project, how do you fairly credit each person for the final result?"

The fair answer: try all possible orderings of team members, and measure how much the final result changes when you add each person to the group. Average that change across all orderings - that's their fair contribution.

SHAP does the same for features. For each prediction, it asks: "How much does the final readmission probability change when we include `creatinine` vs exclude it?" This is computed across all possible subsets of features, giving a fair credit score per feature per prediction.

The key property: **all SHAP values for a prediction always sum to the difference between that prediction and the average prediction.** This makes the numbers interpretable - they're not arbitrary scores, they're actual probability contributions.

```python
explainer   = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

print("SHAP values shape:", shap_values.shape)  # (n_patients, n_features)
print(f"\nAverage readmission probability (test set): {model.predict_proba(X_test)[:, 1].mean():.3f}")
print(f"SHAP base value (same thing):               {explainer.expected_value:.3f}")

# For one patient: all SHAP values sum to prediction - base_value
patient_idx   = 0
patient_pred  = model.predict_proba(X_test)[patient_idx, 1]
shap_sum      = shap_values[patient_idx].sum()

print(f"\nFor patient {patient_idx}:")
print(f"  Prediction:          {patient_pred:.4f}")
print(f"  Base value:          {explainer.expected_value:.4f}")
print(f"  Sum of SHAP values:  {shap_sum:.4f}")
print(f"  Base + SHAP sum:     {explainer.expected_value + shap_sum:.4f}  ← should match prediction")
```

This additivity is what makes SHAP values trustworthy. The numbers are grounded: they literally describe how much the prediction changed from the average because of each feature.

---

## The Waterfall Plot: Explaining One Patient

A waterfall plot shows the full SHAP breakdown for a single patient - which features pushed their risk up, which pushed it down, and by how much. Think of it like an itemised hospital bill: the total is on the bottom, each line item explains a piece of it.

```python
# Find the highest-risk patient in the test set
high_risk_idx  = np.argmax(model.predict_proba(X_test)[:, 1])
high_risk_prob = model.predict_proba(X_test)[high_risk_idx, 1]

print(f"Highest-risk patient readmission probability: {high_risk_prob:.1%}")
print(f"\nFeature values for this patient:")
print(X_test.iloc[high_risk_idx].round(2).to_string())

shap.waterfall_plot(
    shap.Explanation(
        values      = shap_values[high_risk_idx],
        base_values = explainer.expected_value,
        data        = X_test.iloc[high_risk_idx].values,
        feature_names = list(X_test.columns),
    ),
    show=False,
)
plt.title(f"Why is this patient at {high_risk_prob:.0%} readmission risk?")
plt.tight_layout()
plt.show()
```

Read the waterfall from bottom to top: start at the base value (average risk across all patients), then each bar adds or subtracts the SHAP contribution of one feature. The final bar lands at the patient's actual prediction.

**Try this:** Find a patient whose `prev_admissions` is 0 but whose overall risk is still high. Their waterfall will show `prev_admissions` contributing near zero, but other features (creatinine, comorbidities) carrying the weight. This is the "story" of why a low-history patient is still flagged.

---

## The Beeswarm Plot: Global Patterns Across All Patients

A waterfall is for one patient. A beeswarm shows the SHAP behaviour for every patient at once - it's a scatter plot where each dot is one patient, positioned by their SHAP value for that feature, and coloured by the feature's actual value.

```python
shap.summary_plot(shap_values, X_test, show=False, plot_size=(10, 5))
plt.title("SHAP Beeswarm: How Each Feature Affects Readmission Risk Across All Patients")
plt.tight_layout()
plt.show()
```

How to read this:
- **Feature order**: features at the top have the widest overall impact
- **Dot position**: far right = this feature strongly increased risk for this patient; far left = strongly decreased it
- **Dot colour**: red = high feature value, blue = low feature value

So for `prev_admissions`: red dots on the right, blue dots on the left - patients with many prior admissions (red) are pushed toward higher risk, patients with few (blue) toward lower risk. That's exactly the relationship we encoded.

**Try this:** Look at `bmi`. The SHAP values for BMI should be mostly small and centred near zero, with a slight lean left (reducing risk slightly) - consistent with the `-0.02 * bmi` term we used. If you made BMI more protective (e.g., `-0.08`), the blue dots would cluster more to the right.

---

## SHAP Dependence Plot: Feature Value vs Impact

The beeswarm shows all features at once but not the relationship between a feature's value and its impact in detail. A dependence plot isolates one feature and shows exactly how the SHAP value changes as the feature value increases.

```python
fig, axes = plt.subplots(1, 2, figsize=(13, 5))

# Creatinine dependence
creat_vals = X_test["creatinine_mg_dl"].values
creat_shap = shap_values[:, list(X_test.columns).index("creatinine_mg_dl")]
axes[0].scatter(creat_vals, creat_shap, alpha=0.4, c=creat_vals,
                cmap="RdYlBu_r", s=12)
axes[0].axhline(0, color="black", linestyle="--", alpha=0.4)
axes[0].set_xlabel("creatinine_mg_dl")
axes[0].set_ylabel("SHAP value (impact on log-odds)")
axes[0].set_title("Creatinine: As kidneys worsen, readmission risk rises")

# BMI dependence
bmi_vals = X_test["bmi"].values
bmi_shap = shap_values[:, list(X_test.columns).index("bmi")]
axes[1].scatter(bmi_vals, bmi_shap, alpha=0.4, c=bmi_vals,
                cmap="RdYlBu_r", s=12)
axes[1].axhline(0, color="black", linestyle="--", alpha=0.4)
axes[1].set_xlabel("bmi")
axes[1].set_ylabel("SHAP value")
axes[1].set_title("BMI: Slight protective effect at moderate values")

plt.tight_layout()
plt.show()
```

These plots let you spot nonlinear relationships and threshold effects. If creatinine below 1.0 has near-zero SHAP but above 2.0 has sharply positive SHAP, the model has learned a threshold - which you can validate against clinical guidelines (elevated creatinine for CKD staging is typically >1.2 mg/dL).

---

## Practical Notes for Clinical Use

A few things that tripped me up when first using SHAP for clinical reporting:

**SHAP values are in log-odds space, not probability space.** A SHAP value of +0.5 doesn't mean "+50% readmission risk." It means "+0.5 log-odds," which translates to a probability change that depends on the base rate. Use the waterfall plot to communicate probability directly - it handles this conversion.

**Correlation between features can distribute SHAP values unpredictably.** If `num_comorbidities` and `los_days` are correlated (sicker patients stay longer), their SHAP values will share credit. Neither may look dominant individually even though together they explain most of the prediction. A dependence plot with colour-coding by the correlated feature will reveal this.

**TreeExplainer is fast; KernelExplainer is not.** For XGBoost, LightGBM, and CatBoost, always use `shap.TreeExplainer`. It uses the tree structure directly and computes exact SHAP values in seconds. `KernelExplainer` works for any model but approximates SHAP via sampling and can take minutes on a few hundred patients.

---

## What's Next

Day 6 is class imbalance - what happens when only 10% of patients in your dataset are actually readmitted, and how SMOTE, ADASYN, and cost-sensitive learning each try to fix it in different ways. Spoiler: accuracy is the wrong metric to watch when your classes are imbalanced.
