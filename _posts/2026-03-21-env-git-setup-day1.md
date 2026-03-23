---
layout: single
title: "Day 1 of 180 - Dev Environment Setup + Git & GitHub"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*

---

## What Is This?

Before writing a single line of ML code, every professional engineer sets up a proper, reproducible development environment. Today I set up Python 3.11, VS Code with ML-specific extensions, Jupyter notebooks, conda/venv for environment isolation — and then immediately learned Git and GitHub so every project I build from here onwards is properly version-controlled. These two topics are combined in Day 1 of my 180-day plan because they go hand-in-hand: you set up your environment *and* track it in Git from the very first moment.

---

## The Analogy

**Dev environment** is like a carpenter's workshop. Before you build anything, you set up your tools, organise your workbench, and make sure everything is where you expect it. A messy workshop produces messy furniture. A clean, organised workspace produces clean, reliable work.

**Git** is the carpenter's job sheet — a written record of every decision made during the build. "On March 20, I added a drawer. On March 21, I sanded it down. On March 22, I decided to remove it." You can always look back at the job sheet and reconstruct exactly what happened — and undo it if needed.

---

## The Concepts Explained Simply

### Python 3.11 — Why a Specific Version?

Python is not just "Python" — there are versions: 3.9, 3.10, 3.11, 3.12. ML libraries (PyTorch, HuggingFace, scikit-learn) are tested against specific Python versions. Using the wrong version causes silent bugs or broken installs. For this curriculum, we pin **Python 3.11** throughout — it has the best library compatibility as of 2026.

### Virtual Environments — Isolated Workspaces per Project

Imagine you're working on two projects: Project A needs `numpy==1.24` and Project B needs `numpy==1.26`. If they share a single Python installation, one will always be broken.

A **virtual environment** (`.venv`) is a completely isolated copy of Python + packages for one project. You activate it, install packages, and they never interfere with any other project.

```
My Computer
├── Project-A/
│   └── .venv/  ← numpy 1.24 lives here
└── Project-B/
    └── .venv/  ← numpy 1.26 lives here
```

### VS Code Extensions Worth Installing Today

| Extension | Why |
|---|---|
| Python (Microsoft) | IntelliSense, debugging, test runner |
| Pylance | Fast type checking as you type |
| Jupyter | Run `.ipynb` notebooks inside VS Code |
| GitLens | See Git blame and history inline |
| Docker | Manage containers without leaving your editor |
| Even Better TOML | Syntax highlighting for `pyproject.toml` |

### Git — The 3-Zone Model

Every change to your code lives in one of three places:

```
Working Directory  →  Staging Area  →  Repository (History)
   (you edit)          (git add)         (git commit)
```

- **Working Directory**: the files you actually edit
- **Staging Area**: changes you've selected for the next snapshot
- **Repository**: permanent history of snapshots (commits)

### GitHub — Your Engineering Portfolio

GitHub hosts your Git history in the cloud. Every public repository you push to GitHub is part of your **engineering portfolio** — hiring managers look at it. From Day 1, every project goes into a repo with a proper `README.md`, a `.gitignore`, and meaningful commit messages.

### The ML `.gitignore` — What Never Gets Committed

```
# Never commit these
.env              ← API keys and secrets
data/             ← raw datasets (use DVC for these)
models/           ← trained model weights (*.pt, *.pkl)
*.h5              ← Keras model files
__pycache__/      ← Python bytecode cache
.venv/            ← virtual environment (recreatable from requirements.txt)
wandb/            ← W&B experiment logs
mlruns/           ← MLflow runs
.DS_Store         ← macOS metadata junk
```

---

## How Real Companies Use This

Every engineer at Google, Meta, OpenAI, and every ML startup follows this exact pattern from Day 1 of any project:

1. Create a repo with a `.gitignore` before writing a single line of code
2. Use a virtual environment per project — never install globally
3. Pin all dependency versions in `requirements.txt` (`torch==2.3.0`, not just `torch`)
4. Commit with descriptive messages: `"Add ResNet backbone to image classifier"` not `"update"`

A repo without these fundamentals is the first thing a senior engineer will flag in a code review.

---

## Step-by-Step: Try It Yourself

### 1. Install Python 3.11

```bash
# macOS — install pyenv to manage Python versions
brew install pyenv

# Install Python 3.11
pyenv install 3.11.9
pyenv global 3.11.9

# Confirm version
python3 --version  # should print Python 3.11.9
```

### 2. Install VS Code and Extensions

Download VS Code from https://code.visualstudio.com/

Then install extensions from the terminal:
```bash
# Install all recommended ML extensions at once
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-toolsai.jupyter
code --install-extension eamodio.gitlens
code --install-extension ms-azuretools.vscode-docker
code --install-extension tamasfe.even-better-toml
```

### 3. Set up your first project with a virtual environment

```bash
# Create a project folder
mkdir -p ~/projects/ai-engineering-journey && cd ~/projects/ai-engineering-journey

# Create an isolated virtual environment
python3 -m venv .venv

# Activate it (your prompt will show (.venv) when active)
source .venv/bin/activate

# Upgrade pip to latest
pip install --upgrade pip

# Install core ML libraries with exact versions
pip install numpy==1.26.4 pandas==2.2.2 matplotlib==3.8.4 jupyter==1.0.0

# Save exact versions to requirements.txt — this lets anyone recreate your environment
pip freeze > requirements.txt

# Confirm it works
python3 -c "import numpy; print('NumPy:', numpy.__version__)"
python3 -c "import pandas; print('Pandas:', pandas.__version__)"
```

### 4. Configure Git and create your journey repo

```bash
# One-time identity setup
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Check version
git --version
```

On GitHub.com:
- Click **+** → **New repository**
- Name: `ai-engineering-journey`
- ✅ Add a README file
- .gitignore template: **Python**
- Click **Create repository**

```bash
# Clone it to your machine
git clone https://github.com/YOUR_USERNAME/ai-engineering-journey.git
cd ai-engineering-journey

# Create a .venv inside it
python3 -m venv .venv && source .venv/bin/activate
pip install numpy==1.26.4 pandas==2.2.2
pip freeze > requirements.txt
```

### 5. Your first meaningful commit

```bash
# Check what Git sees
git status

# Stage requirements.txt
git add requirements.txt

# Commit with a clear message (imperative tense, under 72 chars)
git commit -m "Add pinned requirements for Day 1 environment"

# Push to GitHub
git push origin main
```

### 6. Verify your setup

```bash
# Every item on this checklist should pass before you move on
python3 --version          # Python 3.11.x
pip list | grep numpy      # numpy  1.26.4
pip list | grep pandas     # pandas 2.2.2
git --version              # git version 2.x.x
git log --oneline          # shows your commit
cat .gitignore | grep .env # confirms .env is ignored
```
---

## Common Mistakes

**Mistake 1: Installing packages globally (without activating .venv first)**
If your terminal prompt doesn't start with `(.venv)`, you're installing into the global Python — which will conflict with other projects. Always check the prompt before running `pip install`.

**Mistake 2: Committing `.venv/` to Git**
The `.venv/` folder contains thousands of files and is 100–500MB. It should be in `.gitignore` (Python template adds it automatically). Anyone can recreate it from `requirements.txt` with `pip install -r requirements.txt`.

**Mistake 3: Not pinning versions in `requirements.txt`**
`pip freeze > requirements.txt` gives you exact versions: `numpy==1.26.4`. If you just write `numpy` without a version, the next person who installs it might get a different version that breaks your code. Always pin.

**Mistake 4: Vague commit messages**
`git commit -m "stuff"` or `git commit -m "update"` is useless. Six months from now you won't know what changed. Write: `git commit -m "Pin numpy to 1.26.4 for M1 compatibility"`.

---

## One-Sentence Lesson

A properly configured environment and version-controlled codebase from Day 1 is the difference between a toy project and a professional one — everything you build for the next 179 days starts here.

---

*Day 1 / 180 complete ✅ | Tomorrow: Linux Terminal + Google Colab & Kaggle — the two environments where all your ML work will run*
