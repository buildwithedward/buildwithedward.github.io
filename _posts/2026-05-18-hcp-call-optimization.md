---
layout: single
title: "How I Built an Explainable AI System to Optimize Pharma Sales Calls"
excerpt: "When a client said 'no black boxes', I discovered GAM - and it changed how I think about enterprise ML"
tags: [mlops, explainable-ai, gam, pharma, fastapi, mlflow, python]
header:
  teaser: /assets/img/bgimage.png
---

A client came to me with a deceptively simple question:

> *"How many times should our sales reps call each doctor to maximize prescriptions - without annoying them?"*

My first instinct was: XGBoost. Maybe a neural network. Something powerful.

Then the client added one constraint that changed everything:

> *"We don't want a black box. We need to explain every recommendation to our commercial team."*

That sent me down a rabbit hole of **explainable AI, causal thinking, and a model type I'd honestly underestimated** - GAM (Generalized Additive Models). I ended up building a complete end-to-end ML platform to understand it deeply. This post walks through what I built and why.

---

## The Business Problem

In the pharma world, **HCP** stands for Healthcare Provider - basically, the doctors that sales representatives visit to promote drugs.

The problem is a classic Goldilocks situation:

- **Too few calls** → the doctor forgets about your drug, you miss prescription opportunities
- **Too many calls** → the doctor gets tired of hearing from you, engagement fatigue and they stop responding

The goal isn't just "more calls = more prescriptions." The goal is finding the **sweet spot for each individual doctor** - because a cardiologist in New York and a neurologist in Texas will have completely different tolerances and response patterns.

This is what commercial analytics teams call **call optimization**, and it's a real problem at scale when you have thousands of doctors across multiple territories.

---

## Why I Didn't Use Deep Learning

When you hear "optimize for each individual", it's tempting to reach for a neural network. They're great at learning individual patterns.

But the client's commercial team had a legitimate need: they wanted to sit in a meeting and explain to their VP *why* Doctor X was getting 3 calls this quarter and Doctor Y was getting 6. A neural network can't do that. It gives you a number with no explanation.

This is a more common situation than you'd think in enterprise settings - especially in regulated industries like pharma, finance, and healthcare. Sometimes **trust and explainability matter more than squeezing out the last 1% of accuracy.**

This led me to **GAM - Generalized Additive Models.**

---

## What is a GAM?

You've probably seen linear regression:

```
Prescriptions = (Calls × 2) + (Emails × 3) + 5
```

That's a straight line. Simple and explainable - but it assumes the relationship is always a straight line. In real life, that's rarely true.

A GAM says instead:

```
Prescriptions = f(Calls) + f(Emails) + f(Recency) + ...
```

Each `f(...)` is a **smooth curve** the model learns from data - not a straight line. And here's the magic: you can **plot each curve individually** and show a business person exactly what's happening.

For this use case, the "calls" curve looks something like this:

```
Prescription Growth
        ▲
        │       ●●●
        │     ●     ●●
        │   ●          ●●●
        │ ●                ●●●●
        └──────────────────────────▶
          1   2   3   4   5   6   7
               Number of Calls

        ←── Sweet Spot ──→←─ Fatigue ─→
```

Initial calls drive growth. After a certain point, returns diminish. Too many calls and you actually hurt performance. GAM captures this naturally - and you can show this exact curve to a stakeholder and say *"this is why we're recommending 4 calls."*

---

## The Key Insight: Not All Doctors Are the Same

Before modeling, I realized there's a fundamental split in the doctor population:

| Group | Who They Are | What We're Trying to Do |
|---|---|---|
| **Breadth HCPs** | Doctors who have *never* prescribed the drug | Convert them to first-time prescribers |
| **Depth HCPs** | Doctors who *already* prescribe | Grow their volume without causing fatigue |

These two groups need completely different models. Asking "will this doctor prescribe?" is a classification problem. Asking "how much will prescriptions grow with one more call?" is a regression problem.

So I built two separate pipelines:

- **Breadth pipeline** → GAM Logistic model → outputs a probability (0–100%) of first-time conversion
- **Depth pipeline** → GAM Regression model → outputs predicted prescription count

---

## The Hidden Bias Problem (And How to Fix It)

Here's something that trips up a lot of commercial analytics work:

> Sales reps are human. They naturally spend more time visiting doctors who are already performing well.

So if you naively train a model on raw call data, it learns the wrong thing:

```
"Doctors who get more calls write more prescriptions"
```

But that's backwards. The doctor was already a top performer - the rep was visiting them *because* they were doing well, not the other way around. This is called **selection bias**, and it can make your model confidently wrong.

### Fix 1: Propensity Scoring

I built a logistic regression model that asks: *"Given this doctor's profile, how likely is a sales rep to call them frequently?"* This score captures the bias - and lets us mathematically correct for it when training the main models. Think of it like a control group in a clinical trial.

### Fix 2: Uplift Features

Instead of asking "will this doctor prescribe?", I engineered features around the question: **"will an extra call actually *cause* more prescriptions?"** This is the causal question. A doctor might have high prescriptions regardless of calls - in that case, calling them more is wasted effort.

Together, these two layers help the model measure the **true incremental effect** of a call, not just correlation.

---

## How the Full System Works

Here's the complete flow from raw data to recommendation:

```
Step 1 - Load historical HCP data
         (calls, prescriptions, emails, territory, specialty)
         ↓
Step 2 - Feature engineering
         (responsiveness score, saturation estimate, recency)
         ↓
Step 3 - Split: Breadth vs Depth doctors
         ↓
Step 4 - Build Propensity Scores (fix selection bias)
         ↓
Step 5 - Engineer Uplift Features (causal signal)
         ↓
Step 6 - Train models
         Breadth → GAM Logistic  (predicts conversion %)
         Depth   → GAM Regression (predicts script growth)
         ↓
Step 7 - Find optimal call count per doctor
         (the point on the curve before diminishing returns)
         ↓
Step 8 - Serve recommendations via FastAPI
         ↓
Step 9 - Track in MLflow, monitor for drift, retrain when needed
```

---

## The Architecture

```
┌─────────────────────────────────────────────┐
│           DATA LAYER                        │
│  20,000 HCP records (synthetic)             │
│  Calls, scripts, emails, specialty, etc.    │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         FEATURE ENGINEERING                 │
│  Propensity Scores + Uplift Features        │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────┴───────────┐
        │                      │
┌───────▼────────┐   ┌─────────▼──────────┐
│ BREADTH MODEL  │   │   DEPTH MODEL      │
│                │   │                    │
│ GAM Logistic   │   │ GAM Regression     │
│ → Conversion % │   │ → Script Growth    │
│                │   │                    │
│ XGBoost        │   │ XGBoost Regressor  │
│ (benchmark)    │   │ (benchmark)        │
└───────┬────────┘   └─────────┬──────────┘
        └──────────┬───────────┘
                   │
┌──────────────────▼──────────────────────────┐
│           MLOPS LAYER                       │
│  MLflow → FastAPI → Docker → Monitoring     │
└─────────────────────────────────────────────┘
```

---

## Step 1 - Generating the Data

I started with a synthetic dataset of 20,000 doctors. The key fields:

```python
df = pd.DataFrame({
    "hcp_id": hcp_ids,
    "specialty": specialty,          # Cardiology, Oncology, etc.
    "previous_scripts": ...,         # How many scripts before?
    "calls": ...,                    # Calls made this quarter
    "email_open_rate": ...,          # Engagement signal
    "responsiveness": ...,           # How much does this doctor respond?
    "saturation": ...,               # How quickly do they tire?
    "estimated_uplift": ...,         # Causal signal
    "converted": ...,                # Did they prescribe? (0 or 1)
    "hcp_type": ...                  # "Breadth" or "Depth"
})
```

The synthetic script growth formula captures the saturation curve:

```python
script_growth = (
    responsiveness * np.log1p(calls)   # diminishing returns
    - saturation * (calls ** 2)        # fatigue penalty
)
```

This means early calls help, but the squared penalty kicks in as calls increase. That's the curve GAM will learn.

---

## Step 2 - Training the Breadth Model

For doctors who've never prescribed, we want to predict the probability they'll convert:

```python
from pygam import LogisticGAM, s

model = LogisticGAM(
    s(0) + s(1) + s(2) + s(3)
)
model.fit(X_train, y_train)

pred_probs = model.predict_proba(X_test)
auc = roc_auc_score(y_test, pred_probs)
```

`s()` is a spline term - it tells GAM "learn a smooth curve for this feature." Each feature gets its own curve. You can plot them individually and explain exactly what's happening.

---

## Step 3 - Training the Depth Model

For existing prescribers, we predict how many scripts they'll write:

```python
from pygam import LinearGAM, s

model = LinearGAM(
    s(0) + s(1) + s(2) + s(3) + s(4)
)
model.fit(X_train, y_train)

preds = model.predict(X_test)
rmse = mean_squared_error(y_test, preds) ** 0.5
```

The partial dependence plot for "calls" is the response curve - it shows exactly where the saturation point is. That's the number you put in your quarterly call plan.

---

## Step 4 - Tracking with MLflow

Without experiment tracking, you're flying blind. Every training run logs its metrics and artifacts:

```python
import mlflow

mlflow.set_experiment("hcp-call-optimization")

with mlflow.start_run():
    model.fit(X_train, y_train)
    rmse = mean_squared_error(y_test, preds) ** 0.5

    mlflow.log_metric("rmse", rmse)
    mlflow.log_param("model_type", "Depth_GAM")
    mlflow.log_artifact("models/depth_gam.pkl")
```

Run `mlflow ui` and you get a full dashboard showing every experiment - which parameters you used, what the metrics were, and which model artifacts were saved.

---

## Step 5 - The Inference API

Once models are trained, they're wrapped in a FastAPI service. Any downstream system - a CRM, a planning tool, a dashboard - can call it:

```
POST /predict/breadth
~ Send: { calls, email_open_rate, call_recency_days, estimated_uplift }
~ Get:  { "conversion_probability": 0.73 }

POST /predict/depth
~ Send: { calls, previous_scripts, email_open_rate, ... }
~ Get:  { "predicted_scripts": 14.2 }
```

```python
from fastapi import FastAPI
from pydantic import BaseModel
import joblib

app = FastAPI()

breadth_model = joblib.load("models/breadth_gam.pkl")
depth_model = joblib.load("models/depth_gam.pkl")

@app.post("/predict/breadth")
def predict_breadth(hcp: BreadthInput):
    row = pd.DataFrame([[hcp.calls, hcp.email_open_rate, ...]])
    pred = breadth_model.predict_proba(row)[0]
    return {"conversion_probability": float(pred)}
```

Start the server with `uvicorn api.app:app --reload` and the interactive docs are available at `http://127.0.0.1:8000/docs`.

---

## Step 6 - Monitoring and Retraining

The monitoring script checks whether real outcomes match predictions:

```python
mean_error = predictions["error"].mean()

if mean_error > 5:
    print("Retraining recommended")
else:
    print("Model stable")
```

Simple, but the idea scales. In production, you'd feed actual prescription data back in, calculate drift, and trigger a retraining pipeline automatically.

---

## Why XGBoost Stayed as a Benchmark

I also trained XGBoost models - both a classifier for breadth and a regressor for depth. They scored similarly to the GAM models.

But the decision was clear: **explainability beats a marginal accuracy bump.** In a commercial setting, a model that a business analyst can point to and explain is infinitely more adoptable than a black box with slightly better AUC.

XGBoost stayed in the project as a challenger - useful for validation, not for production.

---

## Project Structure

```
hcp-call-optimization/
│
├── data/
│   └── synthetic_hcp_data.csv
│
├── src/
│   ├── generate_data.py            ← Synthetic dataset
│   ├── propensity_model.py         ← Bias correction
│   ├── breadth_gam_model.py        ← Breadth training
│   ├── depth_gam_model.py          ← Depth training
│   ├── train_breadth_pipeline.py   ← MLflow breadth run
│   └── train_pipeline.py           ← MLflow depth run
│
├── models/
│   ├── breadth_gam.pkl
│   └── depth_gam.pkl
│
├── api/
│   └── app.py                      ← FastAPI service
│
├── monitoring/
│   └── monitor.py
│
└── docker/
    └── Dockerfile
```

---

## Running the Project

```bash
# Generate data
python src/generate_data.py

# Build bias-correction scores
python src/propensity_model.py

# Train models
python src/breadth_gam_model.py
python src/depth_gam_model.py

# MLflow-tracked training runs
python src/train_breadth_pipeline.py
python src/train_pipeline.py

# View experiment history
mlflow ui
#  http://localhost:5000

# Start the prediction API
uvicorn api.app:app --reload
#  http://127.0.0.1:8000/docs

# Run monitoring check
python monitoring/monitor.py
```

---

## Key Takeaways

**1. Explainability is a feature, not a consolation prize**

In enterprise AI - especially pharma, finance, or healthcare - a model your stakeholders trust will always outperform a model they don't. GAM gave me comparable accuracy to XGBoost and full transparency. In regulated industries, that's not a tradeoff. It's the right answer.

**2. Always ask the causal question**

"Will this doctor prescribe?" and "Did our call *cause* more prescriptions?" are completely different questions. Getting that distinction right changes your feature engineering, your model architecture, and the quality of your recommendations. Propensity scoring and uplift modeling exist to close that gap.

**3. GAM is underrated for commercial analytics**

I came in expecting GAM to be a fallback option. I left with a lot of respect for it. For problems involving saturation curves, diminishing returns, and engagement optimization, GAM is genuinely the right tool - not just a compromise.

**4. MLOps is what makes ML real**

A trained model file sitting on your laptop is not a product. The FastAPI layer, MLflow tracking, Docker container, and monitoring pipeline are what turn an experiment into something a business can actually use.

---

## What's Next

If I were to extend this into a production system, the next steps would be:

- **SHAP values** - per-prediction feature attribution to explain individual recommendations
- **CausalML / EconML** - more rigorous uplift modeling with proper treatment/control framing
- **Airflow** - scheduled retraining pipelines triggered by drift alerts
- **AWS SageMaker** - scalable cloud deployment with auto-scaling endpoints
- **Feature Store** - centralized, versioned features shared across models
- **Drift detection** - automated alerts when input data distribution shifts post-deployment

---

The project started as a way to understand a client use case. It ended up being one of the most interesting explorations I've done - touching causal ML, response curve optimization, and the full MLOps stack. Sometimes the most valuable thing a constraint gives you is a reason to think differently.

If you're working on commercial analytics, HCP engagement, or explainable ML in regulated industries - I'd love to hear how you're approaching it.
