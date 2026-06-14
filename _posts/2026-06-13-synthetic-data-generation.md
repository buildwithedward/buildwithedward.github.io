---
layout: single
title: "Synthetic Data Generation for Healthcare: CTGAN, TVAE, and When to Use Each"
date: 2026-06-13
categories: [machine-learning]
tags: [synthetic-data, ctgan, tvae, sdv, privacy, healthcare-ml, data-generation, clinical-ml]
excerpt: "When you can't share real patient data for model development, synthetic data is the answer - if it's done right. This post covers CTGAN, TVAE, and how to evaluate whether your synthetic data actually preserves the statistical patterns that matter."
---

## Introduction

Here's a problem I ran into on a real project: we wanted to share an EHR dataset with an external research partner, but hospital policy (and HIPAA) restricted sharing real patient records. We also wanted to test our readmission model on data from a different hospital, but they couldn't share their cohort either.

Synthetic data generation is the answer in both cases - generate a dataset that statistically resembles the real one without being recoverable back to real patients. Get it right and you have a dataset you can share, test on, and even open-source. Get it wrong and you either generate data that's too different from reality to be useful, or data that's close enough to real records that privacy isn't really protected.

This post covers the main tools: a Gaussian baseline (simple but often good enough), CTGAN (a GAN-based approach), and TVAE (a variational autoencoder approach). We'll also look at how to evaluate whether the synthetic data is actually any good.

---

## The Real Dataset We're Synthesising

Let's build a slightly richer readmission dataset to synthesise - with a mix of continuous, integer, and categorical features, plus some realistic correlations between them.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score
from scipy import stats
import xgboost as xgb

np.random.seed(42)
N = 2000

# Correlated features: sicker patients stay longer and have more comorbidities
age             = np.random.normal(65, 15, N).clip(18, 95)
severity_latent = np.random.normal(0, 1, N)   # hidden severity factor

num_comorbid    = (np.random.poisson(2.0, N) + (severity_latent > 0.5).astype(int)).clip(0, 10)
los_days        = np.random.exponential(4, N) + severity_latent.clip(0) * 2
los_days        = los_days.clip(1, 30)
creatinine      = (np.random.exponential(1.0, N) + severity_latent.clip(0) * 0.5).clip(0.5, 12)
prev_admissions = np.random.poisson(0.8, N).clip(0, 6)
bmi             = np.random.normal(28, 6, N).clip(16, 55)
heart_rate_bpm  = (np.random.normal(78, 12, N) + severity_latent * 5).clip(45, 140)
gender          = np.random.choice(["M", "F"], N, p=[0.52, 0.48])
blood_type      = np.random.choice(["A+", "A-", "B+", "B-", "O+", "O-", "AB+", "AB-"],
                                    N, p=[0.34, 0.06, 0.09, 0.02, 0.38, 0.07, 0.03, 0.01])

logit = (-3.5 + 0.020*age + 0.28*creatinine - 0.01*bmi
         + 0.38*num_comorbid + 0.07*los_days + 0.45*prev_admissions
         + 0.8*severity_latent + np.random.normal(0, 0.4, N))
readmission_30d = (1 / (1 + np.exp(-logit)) > 0.5).astype(int)

real_df = pd.DataFrame({
    "age": age.round(1), "creatinine_mg_dl": creatinine.round(2),
    "bmi": bmi.round(1), "num_comorbidities": num_comorbid.astype(int),
    "los_days": los_days.round(1), "prev_admissions": prev_admissions.astype(int),
    "heart_rate_bpm": heart_rate_bpm.round(0).astype(int),
    "gender": gender, "blood_type": blood_type,
    "readmission_30d": readmission_30d,
})

print(f"Real dataset: {real_df.shape}")
print(f"Readmission rate: {real_df.readmission_30d.mean():.1%}")
print(f"Creatinine-comorbidity correlation: {real_df.creatinine_mg_dl.corr(real_df.num_comorbidities):.3f}")
print(real_df.head(3))
```

The `severity_latent` variable creates realistic correlations: sicker patients have longer stays, higher creatinine, and faster heart rates. Good synthetic data should preserve these correlations, not just the marginal distributions of each feature.

---

## Baseline: Gaussian Sampling (Simple but Correlation-Blind)

The simplest approach is to fit a Gaussian distribution to each numerical feature independently and sample from it. Think of it like making a copy of a painting by mixing colours that match each individual colour in the painting - you'll get the same colours, but the spatial relationships between them (the composition) will be random.

```python
def gaussian_synthetic(df, n_samples, random_state=42):
    """Generate synthetic data by sampling each column independently."""
    rng = np.random.default_rng(random_state)
    synth = {}
    for col in df.columns:
        if df[col].dtype in [np.float64, np.float32]:
            mean, std = df[col].mean(), df[col].std()
            synth[col] = rng.normal(mean, std, n_samples).clip(df[col].min(), df[col].max())
        elif df[col].dtype in [np.int64, np.int32]:
            mean, std = df[col].mean(), df[col].std()
            vals = rng.normal(mean, std, n_samples).round().astype(int)
            synth[col] = vals.clip(df[col].min(), df[col].max())
        else:
            # Categorical: sample from observed frequency distribution
            freq = df[col].value_counts(normalize=True)
            synth[col] = rng.choice(freq.index, n_samples, p=freq.values)
    return pd.DataFrame(synth)

synth_gaussian = gaussian_synthetic(real_df, N)

# Compare correlations
real_corr   = real_df[["creatinine_mg_dl", "num_comorbidities", "los_days"]].corr()
synth_corr  = synth_gaussian[["creatinine_mg_dl", "num_comorbidities", "los_days"]].corr()

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
import seaborn as sns
sns.heatmap(real_corr,  annot=True, fmt=".2f", ax=axes[0], cmap="coolwarm", vmin=-1, vmax=1)
sns.heatmap(synth_corr, annot=True, fmt=".2f", ax=axes[1], cmap="coolwarm", vmin=-1, vmax=1)
axes[0].set_title("Real data correlations")
axes[1].set_title("Gaussian synthetic correlations")
plt.tight_layout()
plt.show()
```

The correlation matrices will look very different. Gaussian sampling destroys the structure between features - the correlation between creatinine and comorbidities that exists in real patients disappears because we sampled each column independently. If your downstream model relies on these relationships (which readmission models do), Gaussian synthetic data will train a worse model.

**Try this:** Train a readmission XGBoost model on the Gaussian synthetic data and test it on the real held-out data. Compare AUC to a model trained on real data. The performance gap tells you how much the Gaussian approach missed.

---

## CTGAN: Learning the Full Joint Distribution

CTGAN (Conditional Tabular GAN) takes a fundamentally different approach. Instead of sampling each column independently, it trains a GAN - two neural networks competing against each other - to learn the *joint* distribution of all features together.

The analogy: instead of copying individual paint colours, CTGAN studies thousands of real portraits and learns the patterns - how brush strokes cluster, how highlights relate to shadows, how composition works. Then it paints new portraits from scratch that follow the same patterns, without copying any specific portrait.

```python
# pip install sdv
from sdv.single_table import CTGANSynthesizer
from sdv.metadata import SingleTableMetadata

# Tell SDV about your data types
metadata = SingleTableMetadata()
metadata.detect_from_dataframe(real_df)

# Specify which columns are categorical (SDV may not detect all correctly)
metadata.update_column("gender",          sdtype="categorical")
metadata.update_column("blood_type",      sdtype="categorical")
metadata.update_column("readmission_30d", sdtype="categorical")

ctgan = CTGANSynthesizer(
    metadata,
    epochs=200,          # more epochs = better quality, slower
    batch_size=500,
    verbose=True,
)
ctgan.fit(real_df)
synth_ctgan = ctgan.sample(num_rows=N)

# Check correlations preserved
ctgan_corr = synth_ctgan[["creatinine_mg_dl", "num_comorbidities", "los_days"]].corr()
print("CTGAN synthetic correlations:")
print(ctgan_corr.round(3))
print("\nReal correlations:")
print(real_corr.round(3))
```

CTGAN generally preserves inter-feature correlations much better than Gaussian sampling. The tradeoff: it requires a GPU for fast training, takes minutes to hours on large datasets, and can struggle with rare categories (low-frequency categories may be under-represented or distorted in the synthetic output).

**Try this:** Reduce `epochs=20` and re-run. The synthetic data will be lower quality - marginal distributions may look similar but correlations will be weaker. This shows how much the training duration matters for GAN quality.

---

## TVAE: A Faster, More Stable Alternative

TVAE (Tabular Variational Autoencoder) uses a different architecture - an encoder-decoder neural network rather than a GAN. The intuition: the encoder compresses each patient row into a compact "latent" representation (a small vector capturing the patient's core characteristics), and the decoder reconstructs a synthetic patient from a random point in that latent space.

Think of it like learning a map of "patient space." Once you have the map, you can pick any point on it and generate a patient that would live there - even if no real patient existed at exactly that location.

TVAE is typically faster and more stable to train than CTGAN (GANs can be finicky - they sometimes collapse or oscillate). The quality tradeoff depends on your data; neither is universally better.

```python
from sdv.single_table import TVAESynthesizer

tvae = TVAESynthesizer(
    metadata,
    epochs=200,
    batch_size=500,
)
tvae.fit(real_df)
synth_tvae = tvae.sample(num_rows=N)

# Quick comparison of marginal distributions
fig, axes = plt.subplots(2, 4, figsize=(16, 6))
cols_to_check = ["age", "creatinine_mg_dl", "num_comorbidities", "los_days",
                 "prev_admissions", "heart_rate_bpm", "bmi"]

for ax, col in zip(axes.flat, cols_to_check):
    real_vals  = real_df[col].dropna()
    ctgan_vals = synth_ctgan[col].dropna()
    tvae_vals  = synth_tvae[col].dropna()
    ax.hist(real_vals,  bins=25, alpha=0.5, color="#1976D2", label="Real",  density=True)
    ax.hist(ctgan_vals, bins=25, alpha=0.4, color="#E53935", label="CTGAN", density=True)
    ax.hist(tvae_vals,  bins=25, alpha=0.4, color="#43A047", label="TVAE",  density=True)
    ax.set_title(col, fontsize=9)
    ax.legend(fontsize=7)

axes.flat[-1].axis("off")
plt.suptitle("Real vs Synthetic Marginal Distributions", y=1.02, fontsize=12)
plt.tight_layout()
plt.show()
```

---

## Evaluating Synthetic Data: The Train-Synthetic-Test-Real Approach

The best way to evaluate whether your synthetic data is useful is to train a model on it and test on real held-out data. If the model performs nearly as well as one trained on real data, the synthetic data preserved the patterns that matter. This is called the **TSTR (Train on Synthetic, Test on Real)** evaluation.

```python
X_real = real_df.drop("readmission_30d", axis=1).select_dtypes(include=[np.number])
y_real = real_df["readmission_30d"]
X_real_train, X_real_test, y_real_train, y_real_test = train_test_split(
    X_real, y_real, test_size=0.2, random_state=42
)

results = {}

for name, synth_df in [("Gaussian", synth_gaussian),
                        ("CTGAN",    synth_ctgan),
                        ("TVAE",     synth_tvae)]:
    X_synth = synth_df.select_dtypes(include=[np.number]).drop("readmission_30d", axis=1, errors="ignore")
    y_synth = synth_df["readmission_30d"].astype(int)

    # Align columns
    common_cols = [c for c in X_real_train.columns if c in X_synth.columns]
    m = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                           eval_metric="logloss", random_state=42)
    m.fit(X_synth[common_cols], y_synth, verbose=False)
    auc = roc_auc_score(y_real_test, m.predict_proba(X_real_test[common_cols])[:, 1])
    results[name] = auc

# Baseline: train on real, test on real
m_real = xgb.XGBClassifier(n_estimators=200, learning_rate=0.1, max_depth=4,
                             eval_metric="logloss", random_state=42)
m_real.fit(X_real_train, y_real_train, verbose=False)
results["Real (upper bound)"] = roc_auc_score(y_real_test, m_real.predict_proba(X_real_test)[:, 1])

print("TSTR Evaluation - How Well Does Synthetic Data Substitute for Real Training Data?")
print(f"{'Method':<25} {'AUC (tested on real data)':>28}")
print("-" * 55)
for name, auc in sorted(results.items(), key=lambda x: -x[1]):
    gap = results["Real (upper bound)"] - auc
    bar = "█" * int((1 - gap) * 30)
    print(f"{name:<25} {auc:.4f}  (gap from real: {gap:+.4f})")
```

The "gap from real" tells you how much you're giving up by using synthetic training data. A gap under 0.03 AUC is usually acceptable for model development and sharing purposes. A gap over 0.08 means the synthetic data is missing important structural patterns.

**Try this:** Evaluate the Kolmogorov-Smirnov statistic for each feature between real and synthetic data. A KS statistic near 0 means the distributions match well; near 1 means they're very different. This gives a column-by-column audit of where the synthetic generation is failing.

---

## Privacy Check: Are Any Real Patients Recoverable?

Synthetic data is only safe to share if real patients can't be identified from it. A basic check: for each synthetic patient, find the nearest real patient (by Euclidean distance in feature space). If synthetic patients are consistently very close to real patients, the generator is memorising instead of generalising.

```python
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import NearestNeighbors

# Compute nearest-neighbour distances: synthetic → real
scaler = StandardScaler()
X_real_scaled  = scaler.fit_transform(X_real)
X_synth_scaled = scaler.transform(synth_ctgan.select_dtypes(include=[np.number])
                                              .drop("readmission_30d", axis=1, errors="ignore")
                                              .reindex(columns=X_real.columns, fill_value=0))

nn = NearestNeighbors(n_neighbors=1)
nn.fit(X_real_scaled)
distances, _ = nn.kneighbors(X_synth_scaled)
min_distances = distances[:, 0]

print(f"Nearest-neighbour distance (synthetic → real):")
print(f"  Mean:   {min_distances.mean():.3f}")
print(f"  Median: {np.median(min_distances):.3f}")
print(f"  % within distance 0.5: {(min_distances < 0.5).mean():.1%}  ← memorisation risk if high")

plt.figure(figsize=(8, 4))
plt.hist(min_distances, bins=40, color="#FF6F00", edgecolor="white", alpha=0.8)
plt.xlabel("Distance to nearest real patient (standardised)")
plt.ylabel("Number of synthetic patients")
plt.title("Privacy Check: Distribution of Distances from Synthetic to Real Patients")
plt.axvline(0.5, color="red", linestyle="--", label="Memorisation risk threshold")
plt.legend()
plt.tight_layout()
plt.show()
```

If most synthetic patients are very close to specific real patients (distance < 0.5 in standardised space), the model is memorising. You want a spread of distances showing the synthetic patients are blended across the space, not anchored to individual real records.

---

## When to Use Each Approach

| Situation | Reach for |
|---|---|
| Quick prototyping, marginal distributions matter | Gaussian (simple, fast, correlations lost) |
| Sharing data externally, correlations must hold | CTGAN or TVAE |
| Unstable training, GPU not available | TVAE (more stable than CTGAN) |
| Large dataset (>50K rows), complex categoricals | CTGAN (better capacity) |
| Privacy audit required | Run nearest-neighbour check regardless of method |
| Rare disease / very imbalanced dataset | Both can struggle - oversample minority class first |

The core takeaway: Gaussian synthetic data is fine for testing pipelines. For anything involving training a real model that will be evaluated on real patient outcomes, use CTGAN or TVAE and validate with the TSTR evaluation.

---

## What's Next

Day 9 is the first mini-project: a full clinical trial drug response prediction system using XGBoost + SHAP + MLflow + Optuna - everything from Phase 1 combined into a production-grade pipeline.
