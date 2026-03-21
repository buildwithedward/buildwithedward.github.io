---
layout: single
title: "Day 1 - The Journey Begins"
excerpt: "Setting up the environment and mapping out the Deep Learning & LLM Systems landscape"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
# Day 1 of 247: Setting Up Your AI/ML Development Environment

> *This is Day 1 of my 247-day AI Engineering journey — from Python basics all the way to deploying production LLMs in the cloud. I'm documenting every day. Follow along!*

---

## 🎯 What I Learned Today

Before writing a single line of machine learning code, you need a clean, reproducible development environment. This is something many beginners skip - and it causes endless "it works on my machine" headaches later. Day 1 is all about getting this right once, so every project for the next 246 days just works.

---

## 🧠 The Core Concepts

### Why Virtual Environments Matter

When you install Python packages globally (without a virtual environment), every project on your machine shares the same packages. This breaks things fast — Project A needs `numpy==1.24`, Project B needs `numpy==1.26`, and they can't coexist.

A **virtual environment** creates an isolated bubble of packages for each project. Activate it, install what you need, and nothing leaks out to other projects.

Think of it like this: instead of one shared kitchen where everyone leaves their ingredients, each project gets its own private kitchen.

### The Standard ML Project Structure

Every professional ML project follows a similar folder layout. Having this structure from Day 1 means your code is always organised, and anyone can understand your project instantly:

```
ai-engineering/
├── data/
│   ├── raw/          # original data — never modify this
│   └── processed/    # cleaned and transformed data
├── notebooks/        # Jupyter notebooks for exploration
├── src/              # importable Python source code
├── models/           # saved model files (.pkl, .pt, etc.)
├── tests/            # unit tests
├── requirements.txt  # pinned package versions
├── .gitignore        # files Git should ignore
└── README.md         # project overview
```

The most important rule: **raw data is sacred**. You never overwrite it. Always read from `data/raw/` and write outputs to `data/processed/`.

---

## 🔥 Hands-On: Build Your ML Workspace from Scratch

Here's exactly what I did, step by step. Every command is reproducible on macOS, Linux, or Windows (PowerShell).

### Step 1 — Check Your Python Version

You need Python 3.11 or higher for this curriculum.

```bash
python3 --version
```

If you're below 3.11, download the latest from [python.org](https://www.python.org/downloads/) or use Homebrew on macOS:

```bash
brew install python@3.11
```

### Step 2 — Create the Project Folder & Structure

```bash
# Create the project directory and enter it
mkdir ai-engineering && cd ai-engineering

# Create the standard ML folder layout
mkdir -p data/raw data/processed notebooks src models tests

# Create placeholder files
touch README.md requirements.txt .gitignore src/__init__.py
```

### Step 3 — Create and Activate a Virtual Environment

```bash
# Create the virtual environment (stored in a hidden .venv folder)
python3 -m venv .venv

# Activate it — macOS / Linux
source .venv/bin/activate

# Activate it — Windows PowerShell
# .venv\Scripts\Activate.ps1
```

Once activated, your terminal prompt shows `(.venv)` — that means you're inside the isolated environment. Any packages you install now stay in this project only.

### Step 4 — Install Core ML Packages

```bash
# Upgrade pip first
pip install --upgrade pip

# Install the core ML stack
pip install numpy pandas scikit-learn matplotlib seaborn jupyterlab
```

### Step 5 — Pin Your Dependencies

```bash
# Save exact versions to requirements.txt
pip freeze > requirements.txt
```

This file is your environment's "recipe". Anyone can recreate your exact setup by running `pip install -r requirements.txt`.

### Step 6 — Verify Everything Works

```bash
python -c "import numpy, pandas, sklearn, matplotlib, seaborn; print('All imports OK ✅')"
```

**Expected output:**
```
All imports OK ✅
```

### Step 7 — Set Up .gitignore

```bash
cat > .gitignore << 'EOF'
.venv/
__pycache__/
*.pyc
.ipynb_checkpoints/
.env
models/*.pkl
data/raw/*
EOF
```

This keeps your virtual environment, cached files, secrets, and large data files out of version control.

---

## ✅ Done When…

- [ ] `python3 --version` shows 3.11 or higher
- [ ] The `ai-engineering/` folder exists with all subdirectories (`data/`, `notebooks/`, `src/`, `models/`, `tests/`)
- [ ] A virtual environment is active and `python -c "import numpy, pandas, sklearn"` runs without errors
- [ ] `requirements.txt` has been generated with `pip freeze`

---

## 📅 What's Coming on Day 2

Tomorrow I'm setting up **Git & GitHub** — version control for ML projects. Every model I train, every experiment I run, every script I write will be versioned and backed up. Never lose work again.

---

## 🗺️ About This Series

I'm following a **247-day AI Engineering curriculum** covering:

- **Phase 0–2:** Dev setup, Python, Math, ML Fundamentals
- **Phase 3–5:** Neural Networks, Deep Learning, Computer Vision
- **Phase 5.5:** GPU Architecture & CUDA
- **Phase 6–7:** NLP, Transformers, Reinforcement Learning
- **Phase 8–8.5:** LLMs, Fine-tuning, Advanced Inference (vLLM, FlashAttention, Quantization)
- **Phase 9:** Agent Engineering & Multi-Agent Systems
- **Phase 10–11:** Cloud Networking (VPC, Subnets) & MLOps (Docker, FastAPI, SageMaker, CI/CD)
- **Phase 12–13:** Ethics, Safety & Capstone Projects

Each day is 1 hour: 20 minutes of theory + 35 minutes of hands-on + 5 minutes of reflection.

*See you on Day 2!* 🚀

---

*Tags: `#MachineLearning` `#AI` `#DeepLearning` `#Python` `#100DaysOfCode` `#MLOps` `#LLM` `#AIEngineering`*
