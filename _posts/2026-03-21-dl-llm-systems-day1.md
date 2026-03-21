---
layout: single
title: "Day 1 - How Real AI Companies Set Up Their Dev Environment"
excerpt: "Setting up the environment and mapping out the Deep Learning & LLM Systems landscape"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
# Day 1 of 247: How Real AI Companies Set Up Their Dev Environment (And Why You Should Too)

> *Part of my 247-day AI Engineering journey — learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*

---

## 🤔 What Is This and Why Does It Matter at Real Companies?

Imagine joining a new company on your first day. Your manager says "just clone the repo and get started." You clone it, try to run it, and spend three hours debugging because some package version on your machine doesn't match what everyone else uses. Sound familiar? This is one of the most common time-wasters in real engineering teams.

Today's lesson is about eliminating that problem entirely — before it ever happens. Before writing any machine learning code, professional AI engineers set up a **clean, reproducible development environment**. It sounds boring. It is the most important thing you'll do.

At companies like Google, Meta, and every serious AI startup, no one ever writes code without this foundation. Today you'll build the exact same setup they use.

---

## 🧠 The Concept, Explained Simply

### What Is a Virtual Environment?

Think of your computer like a shared kitchen. If everyone cooks in the same kitchen and leaves their ingredients lying around, things get messy fast. Someone uses up the last of the flour, someone else reorganises the spice rack — chaos.

A **virtual environment** gives each project its own private kitchen. The packages (ingredients) you install for Project A stay in Project A's kitchen and never interfere with Project B. When you're done, you just close the kitchen door.

In Python, this means running:
```bash
python3 -m venv .venv
source .venv/bin/activate  # "open the kitchen door"
```

Now anything you install with `pip install` only goes into this project's private space.

### Why Do We Pin Exact Versions?

When you write `numpy==1.26.4` in your requirements file, you're saying "use *this exact version*, nothing else." If you write `numpy>=1.24` instead, six months later pip might install `numpy==2.0` which has breaking changes — and your code silently fails in production on a Friday night. Pinning versions is what makes your environment identical on your laptop, your teammate's laptop, and the CI/CD server.

### What Is a Pydantic Config and Why Not Just Use `os.getenv()`?

Many beginners do this:
```python
import os
api_key = os.getenv("API_KEY")  # returns None if missing — silent failure!
```

If `API_KEY` is missing, you get `None` and your app crashes later with a confusing error deep inside some function. In production, you want the app to **fail immediately at startup** with a clear message: "API_KEY is required but not set."

**Pydantic Settings** does exactly that — it reads all your config from environment variables, validates the types, and crashes loudly at startup if anything is missing. Fail fast, fail clearly.

### What Is a Custom Exception?

Instead of:
```python
raise Exception("something went wrong")  # useless in production logs
```

Production code uses:
```python
raise ModelLoadError("model artifact not found at path: models/v2.pkl")
```

Now your monitoring system can filter logs by `ModelLoadError`, set up specific alerts, and your on-call engineer knows exactly what broke without reading the entire stack trace.

---

## 🏭 What This Looks Like at a Real Company

At NeuralCorp (the fictional AI company I'm working through in this series), every single ML project starts from the same template. When a new engineer joins, they clone the template, run `make install`, run `make test`, and they're productive in minutes — not hours.

This template enforces the rules automatically: secrets can't be committed to Git because `.env` is in `.gitignore`. Config can't be hardcoded because Pydantic reads everything from environment variables. Bad code can't be pushed because a linting check runs before every commit.

The folder structure tells a story that every engineer can read instantly:
- `src/` — the actual code
- `tests/` — the safety net
- `data/raw/` — original data, never touched
- `data/processed/` — outputs of our transformations
- `models/` — saved model files

---

## 💻 The Code — Line by Line

Here's the production-grade config module I built today. Every line is explained.

```python
# src/config.py
# This file manages ALL configuration for the project
# Rule: no hardcoded values anywhere else in the codebase

import logging
from pydantic_settings import BaseSettings  # reads from .env automatically
from pydantic import Field                  # lets us set defaults and add docs

# Set up structured logging — every log line has a timestamp, level, and module name
# This format works with log aggregation tools like Datadog, CloudWatch, Splunk
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
)
logger = logging.getLogger(__name__)  # creates a logger named after this file


class Settings(BaseSettings):
    """All application configuration, loaded from environment variables.

    If a required variable is missing, Pydantic raises an error at startup.
    This is intentional — we want to know immediately, not after 10 minutes.
    """

    app_name: str = Field(default="neuralcorp-ml-template")  # identifies this app in logs
    environment: str = Field(default="development")           # dev / staging / production
    log_level: str = Field(default="INFO")                    # how verbose logs should be
    random_seed: int = Field(default=42)                      # same seed = same results

    class Config:
        env_file = ".env"       # look for a .env file in the project root
        case_sensitive = False  # APP_NAME and app_name are treated the same


def get_settings() -> Settings:  # -> Settings means this function returns a Settings object
    """Load settings and log that startup succeeded.

    Returns:
        Settings: the validated configuration object
    """
    settings = Settings()  # Pydantic reads env vars and validates them here
    logger.info("Settings loaded successfully | env=%s", settings.environment)
    return settings
```

And here's a custom exception — small but critical:

```python
# src/exceptions.py
# Never raise bare Exception() in production — use specific named exceptions
# Specific exceptions make monitoring, alerting, and debugging 10x easier

class NeuralCorpBaseError(Exception):
    """Every custom exception in this project inherits from this.
    Lets callers catch all NeuralCorp errors with one except clause if needed."""
    pass

class ConfigurationError(NeuralCorpBaseError):
    """Raised when a required config value is missing or wrong type."""
    pass

class ModelLoadError(NeuralCorpBaseError):
    """Raised when a model file can't be found or loaded from disk."""
    pass
```

**What you'll see when you run `make test`:**
```
tests/test_config.py::TestSettings::test_default_settings_load_successfully PASSED
tests/test_config.py::TestSettings::test_settings_override_via_env_vars PASSED
tests/test_config.py::TestSettings::test_random_seed_is_integer PASSED
tests/test_config.py::TestSettings::test_settings_returns_settings_instance PASSED

4 passed in 0.42s
```

Four green tests. That's your proof the foundation is solid.

---

## ✅ Today's One-Sentence Lesson

A reproducible environment, typed configuration, custom exceptions, and tests from Day 1 eliminate an entire category of bugs before they can ever happen in production.

---

## 🔗 Up Next

**Day 2:** Git & GitHub for ML — set up branch protection, commit conventions, a `.gitignore` tuned for ML artifacts, and a pre-commit hook so broken code can never reach `main`.

---

## 📚 About This Series

I'm learning AI Engineering from scratch — 1 hour a day, 247 days, building everything to **production standards** (not just PoC scripts). Every day I get a real engineering ticket from a fictional company called NeuralCorp and implement it the way a real junior ML engineer would.

**The full roadmap:**
- 🐍 Python, Math & ML Fundamentals (Days 1–52)
- 🧠 Neural Networks & Deep Learning (Days 53–89)
- 👁️ Computer Vision & Generative AI (Days 90–104)
- ⚡ GPU Architecture & CUDA (Days 105–115)
- 📝 NLP, Transformers & LLMs (Days 116–164)
- 🚀 Advanced LLM Inference & LLMOps (Days 165–184)
- 🤖 AI Agents (Days 185–197)
- ☁️ Cloud Networking & MLOps (Days 198–234)
- 🛡️ Ethics, Safety & Capstones (Days 235–247)

*One day at a time. Follow along!* 🚀

---

*Tags: `#AI` `#MachineLearning` `#Python` `#AIEngineering` `#100DaysOfCode` `#Beginners` `#MLOps` `#ProductionML` `#DevSetup`*
