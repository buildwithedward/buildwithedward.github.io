---
layout: single
title: "Day 1 of 290 - Setting Up Your ML Dev Environment (The Right Way)"
excerpt: "Part of my 290-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 290-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*

---

## What Is This?

Before you write a single line of machine learning code, you need the right tools installed correctly. This might sound boring, but it's actually one of the most important things a professional ML engineer does - because a bad setup wastes *days* of debugging time later.

At companies like Google, Meta, and every AI startup you've heard of, every engineer's machine is set up the same way. When you join a team, you run one setup script and your machine matches theirs exactly. Today you'll learn how to do that.

---

## The Analogy

Think of your computer as a kitchen. You could throw all your ingredients - flour, spices, oil - onto one counter and mix everything together on the same surface. It works, but it's chaos. Ingredients from one recipe contaminate another.

A professional chef uses **separate, labelled containers** for each recipe. In Python, we call these **virtual environments**. Each project gets its own isolated container of Python libraries. When you're done, you delete the container - nothing breaks.

---

## The Concept Explained Simply

### Why Not Just Use System Python?

Your Mac comes with Python already installed. But that Python is used by macOS itself. If you install packages into it and something conflicts, macOS can break in strange ways.

Always use a virtual environment. Always.

### What Is conda?

**conda** is a tool that creates and manages virtual environments. We use **Miniforge** - the Apple Silicon (M1) native version. It's free and much faster on your Mac than the official Anaconda.

### What Is VS Code?

**VS Code** (Visual Studio Code) is a text editor designed for writing code. It has autocomplete, syntax highlighting, a built-in terminal, and extensions for Python, Git, and everything else you'll use. It's the most popular editor in ML engineering.

### What Is Jupyter?

**Jupyter Notebook** lets you write code in small "cells" and run them one at a time. You see the output (numbers, charts, text) right below each cell. This is ideal for experiments - you can run one step at a time, inspect results, and continue.

---

## How Real Companies Use This

When a new engineer joins NeuralCorp, they run `conda activate project-env` before they do anything else. This ensures they're using the exact same Python version and library versions as everyone else. If the model trains with loss=0.34 on the lead engineer's machine, it should train with loss=0.34 on yours too.

This reproducibility is what separates amateur ML code from production ML code.

---

## Step-by-Step: Try It Yourself

### 1. Install the tools

```bash
# Install Homebrew (Mac package manager)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Miniforge (Apple Silicon conda)
brew install miniforge
conda init zsh   # restarts your shell config

# Close and reopen terminal, then verify - you should see (base) in your prompt

# Install VS Code
brew install --cask visual-studio-code
```

### 2. Create your first environment

```bash
# Create a Python 3.11 environment called ml-fundamentals
conda create -n ml-fundamentals python=3.11 -y

# Activate it - run this every time you start working
conda activate ml-fundamentals

# Verify you're using the right Python
which python   # should show a path containing ml-fundamentals
python --version   # should show Python 3.11.x
```

### 3. Install Jupyter and write your first script

```bash
pip install jupyter notebook ipykernel

# Register the environment so Jupyter can use it
python -m ipykernel install --user --name=ml-fundamentals --display-name "ML Fundamentals (3.11)"

# Create a project folder
mkdir ~/neuralcorp-setup
cd ~/neuralcorp-setup
```

Create `hello_neuralcorp.py`:
```python
# hello_neuralcorp.py - first Python script at NeuralCorp
import sys
import platform

print(f"Hello, NeuralCorp!")
print(f"Python version: {sys.version}")     # shows exact version
print(f"Machine: {platform.machine()}")     # 'arm64' on M1 Mac
```

```bash
python hello_neuralcorp.py
```

**What you'll see:**
```
Hello, NeuralCorp!
Python version: 3.11.x ...
Machine: arm64
```

Create `check_environment.py`:
```python
# check_environment.py - confirms the environment is correctly set up
import sys

REQUIRED_MAJOR = 3
REQUIRED_MINOR = 11   # minimum Python version NeuralCorp requires

current_major = sys.version_info.major
current_minor = sys.version_info.minor

print("=" * 40)
print("NeuralCorp Environment Check")
print("=" * 40)

if current_major == REQUIRED_MAJOR and current_minor >= REQUIRED_MINOR:
    print(f"✅ Python {current_major}.{current_minor} - OK")
else:
    print(f"❌ Python {current_major}.{current_minor} found - need 3.11+")
    print(f"   Fix: conda create -n ml-fundamentals python=3.11")

print(f"\nEnvironment path:\n  {sys.executable}")

# Detect if we're inside a virtual environment (not bare system Python)
in_venv = sys.prefix != sys.base_prefix
if in_venv:
    print("✅ Running inside a virtual environment - good!")
else:
    print("⚠️  Not in a virtual environment. Run: conda activate ml-fundamentals")
```

```bash
python check_environment.py
```

**What you'll see:**
```
========================================
NeuralCorp Environment Check
========================================
✅ Python 3.11 - OK

Environment path:
  /opt/homebrew/.../ml-fundamentals/bin/python
✅ Running inside a virtual environment - good!
```

### 4. Start Jupyter

```bash
jupyter notebook
```

Your browser opens automatically. Click **New → ML Fundamentals (3.11)**. In the first cell type `print("Jupyter works!")` and press `Shift+Enter`.

---

## Common Mistakes

```
❌ conda: command not found
✅ Close terminal, reopen it (conda init needs a fresh shell to take effect)

❌ which python shows /usr/bin/python (system Python)
✅ Run: conda activate ml-fundamentals

❌ Jupyter doesn't show "ML Fundamentals" kernel
✅ Run: python -m ipykernel install --user --name=ml-fundamentals --display-name "ML Fundamentals (3.11)"
```

---

## ✅ Today's One-Sentence Lesson

The two minutes you spend creating a virtual environment saves you two days of "it worked on my machine" debugging later.

---

## 🔗 Up Next

**Day 2:** Git & GitHub - version control for ML projects. You'll learn how to save every version of your code so you can always go back, and how to share code with teammates.

---

## 📚 About This Series

I'm learning AI Engineering from scratch - 1 hour a day, 290 days, everything documented in plain English.

*Tags: `#AI` `#MachineLearning` `#Python` `#AIEngineering` `#Beginners` `#DevSetup` `#conda` `#VSCode` `#Jupyter`*
