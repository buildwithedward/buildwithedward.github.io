---
layout: single
title: "Day 2 of 180 - Linux Terminal + Google Colab & Kaggle Setup"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## What Is This?

Day 2 covers the two environments every ML engineer lives in: the **Linux terminal** (the text-based interface that controls every production server on Earth) and **Google Colab + Kaggle** (free cloud GPU environments that let you train models your laptop can't handle). These go together naturally — you use the terminal on your Mac for local work, and you use Colab/Kaggle for anything that needs a CUDA GPU (which your M1 doesn't have).

---

## The Analogy

**The terminal** is the steering wheel of a race car. A Finder window is more like a taxi — comfortable, automatic, but slow and limited. The moment your model lives on an AWS EC2 server or RunPod GPU pod, there's no taxi. There's only the steering wheel. The sooner you get comfortable with it, the faster you move.

**Google Colab** is like renting a Formula 1 car for free for a few hours. Your MacBook Air M1 is a great car for most things, but it has no NVIDIA GPU — it can't run CUDA-based training. Colab gives you a free Tesla T4 GPU (15GB VRAM) in the cloud. You open a browser, click Runtime → T4 GPU, and you're training.

---

## The Concepts Explained Simply

### Linux Terminal — Why It Matters for ML

Every ML model deployed in production runs on a Linux server. When you SSH into RunPod to run vLLM, when you connect to an AWS EC2 instance to deploy your FastAPI model — you land in a terminal. There's no GUI. You navigate, manage files, run scripts, and monitor processes entirely through typed commands.

macOS uses **Zsh** by default; Linux servers use **Bash**. They're 99% identical for the commands you'll use in ML engineering.

### The 15 Commands You'll Use Every Day

**Navigation:**
```bash
pwd           # Where am I? (print working directory)
ls -la        # What's here? (-l = details, -a = hidden files)
cd models/    # Move into a folder
cd ..         # Go up one level
cd ~          # Go home, no matter where you are
```

**File Operations:**
```bash
mkdir -p src/utils        # Create folder + parents (no error if exists)
touch src/__init__.py     # Create an empty file
cp model.pkl models/      # Copy file to a folder
mv old.py new.py          # Rename a file (mv also moves)
rm -rf old_experiments/   # Delete folder recursively (no undo!)
```

**Peeking Inside Files:**
```bash
cat requirements.txt      # Print entire file
head -5 data.csv          # First 5 lines (check CSV headers)
tail -f logs/app.log      # Last lines, live-updating (follow mode — watch logs in real time)
wc -l dataset.csv         # Count lines = row count
```

**Searching:**
```bash
grep "learning_rate" config.yaml       # Find lines with that text
grep -rn "import torch" src/           # Search recursively, show line numbers
find . -name "*.py"                    # Find all Python files
find . -name "*.pt" -size +100M        # Find model files > 100MB
```

**Power Combos — Pipes and Redirects:**
```bash
# | pipe: feed output of one command into the next
cut -d',' -f5 data.csv | sort | uniq -c   # count unique values in column 5
ps aux | grep python                        # find running Python processes

# > redirect: write output to a file
pip freeze > requirements.txt    # save installed packages to file
python train.py > logs/run.log   # save all training output to a log file

# && AND: run second command only if first succeeds
mkdir project && cd project      # create folder, then enter it
make test && make docker-build   # only build if tests pass
```

**Downloading Data:**
```bash
# curl: "client URL" — fetch anything from the internet
curl -L -o data/iris.csv "https://raw.githubusercontent.com/.../iris.csv"
# -L = follow redirects   -o = save to file
```

### Google Colab — Free T4 GPU in a Browser

Google Colab is a hosted Jupyter notebook environment with free GPU access. For this curriculum:

- **Free tier**: Tesla T4 (15GB VRAM) — perfect for QLoRA fine-tuning of 7B models, CUDA experiments
- **How to activate GPU**: Runtime → Change runtime type → T4 GPU → Save
- **Storage**: Files are lost when session ends. Always save outputs to Google Drive or download them.
- **Time limit**: ~12 hours per session. Save checkpoints often.

Key Colab patterns you'll use throughout the curriculum:

```python
# Always run this first to check your GPU
import torch
print(torch.cuda.is_available())      # True = you have a GPU
print(torch.cuda.get_device_name(0))  # e.g., Tesla T4

# Mount Google Drive to persist files across sessions
from google.colab import drive
drive.mount('/content/drive')
# Your Drive is now accessible at /content/drive/MyDrive/
```

### Kaggle — Datasets + Free GPU Notebooks

Kaggle is the ML community platform. Two things matter for this curriculum:

1. **Datasets**: 50,000+ free public datasets, downloadable via CLI
2. **Free GPU notebooks**: 30 hrs/week of P100 GPU (16GB) — useful backup when Colab is busy

**Kaggle API setup** (do this once):
1. Go to kaggle.com → Settings → API → **Create New Token** → downloads `kaggle.json`
2. Store it: `mkdir -p ~/.kaggle && mv ~/Downloads/kaggle.json ~/.kaggle/ && chmod 600 ~/.kaggle/kaggle.json`
3. Now you can download any dataset with one command

### M1 + Colab/Kaggle — The Split Workflow

This is the workflow you'll use for the next 179 days:

```
MacBook Air M1 (local)          Google Colab / Kaggle (cloud)
────────────────────────        ──────────────────────────────
✅ Data exploration             ✅ QLoRA fine-tuning (7B models)
✅ Preprocessing pipelines      ✅ CUDA experiments
✅ FastAPI development           ✅ bitsandbytes quantisation
✅ Ollama / MLX inference       ✅ FlashAttention benchmarks
✅ RAG systems (Qdrant)         ✅ Large batch training
✅ Agent development            ✅ HuggingFace model downloads
```

---

## How Real Companies Use This

- **Every ML engineer's first action on a new server**: SSH in via terminal, `ls -la`, `ps aux | grep python`, `tail -f logs/app.log`
- **Startups with small GPU budgets** use Colab Pro (~$10/month for A100 access) for model fine-tuning before investing in cloud infrastructure
- **Kaggle competitions** are how many ML engineers built their first portfolio — companies like DoorDash, Booking.com actively recruit from Kaggle leaderboards
- **The `&&` pattern** (`make test && make deploy`) is how every CI/CD pipeline works — don't deploy if tests fail

---

## Step-by-Step: Try It Yourself

### Part A — Terminal Practice (45 min)

Open Terminal on your Mac (Spotlight → "Terminal") and run through this:

```bash
# 1. Navigate and explore
pwd                              # note your home directory path
ls -la ~                        # list everything in home folder
ls -la ~/projects/ai-engineering-journey/   # check your project from Day 1

# 2. Create an ML project mini-structure
mkdir -p ~/terminal-practice/{data,models,logs,src}
cd ~/terminal-practice
ls -la                           # confirm all folders created

# 3. Download a real dataset with curl
curl -L -o data/iris.csv \
  "https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv"

# 4. Explore it with terminal commands — no Python needed!
head -5 data/iris.csv           # peek at the headers
wc -l data/iris.csv             # count rows: should be 151 (150 + header)

# 5. Use pipes to analyse without opening the file
cut -d',' -f5 data/iris.csv | sort | uniq -c
# Expected:
#   1 species       ← the header row
#  50 setosa
#  50 versicolor
#  50 virginica

# 6. Search inside the file
grep "virginica" data/iris.csv | wc -l    # count virginica rows → 50
grep "virginica" data/iris.csv | head -3  # show first 3 virginica rows

# 7. Save a filtered result to a new file
grep "setosa" data/iris.csv > data/setosa_only.csv
wc -l data/setosa_only.csv      # should be 50

# 8. Create a requirements file
cat > requirements.txt << 'EOF'
pandas==2.2.2
numpy==1.26.4
scikit-learn==1.4.2
EOF
cat requirements.txt            # verify it

# 9. Find all CSV files you just created
find . -name "*.csv"

# 10. Process management check
ps aux | grep python            # any Python running?
```

### Part B — Google Colab Setup (20 min)

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Click **New notebook**
3. Runtime → Change runtime type → **T4 GPU** → Save
4. Run this verification cell:

```python
# Cell 1: Verify GPU access
import torch
import subprocess

print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"VRAM: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")

# Check nvidia-smi (shows GPU utilisation)
result = subprocess.run(['nvidia-smi'], capture_output=True, text=True)
print(result.stdout)
```

Expected output:
```
PyTorch version: 2.x.x+cu121
CUDA available: True
GPU: Tesla T4
VRAM: 15.8 GB
```

```python
# Cell 2: Mount Google Drive (persist your work)
from google.colab import drive
drive.mount('/content/drive')

# Create a persistent folder for this curriculum
import os
os.makedirs('/content/drive/MyDrive/ai-engineering-180/', exist_ok=True)
print("Persistent folder ready at: /content/drive/MyDrive/ai-engineering-180/")
```

```python
# Cell 3: Test a quick GPU computation
import torch
import time

# Create large tensors on GPU
a = torch.randn(5000, 5000).cuda()
b = torch.randn(5000, 5000).cuda()

# Time a matrix multiply (this is what neural network layers do constantly)
start = time.time()
c = torch.matmul(a, b)
torch.cuda.synchronize()   # wait for GPU to finish
elapsed = time.time() - start

print(f"5000×5000 matrix multiply on T4: {elapsed*1000:.1f}ms")
# Your M1 CPU would take ~10× longer for this
```

### Part C — Kaggle Setup (15 min)

```bash
# On your Mac terminal:

# Install Kaggle CLI
pip install kaggle==1.6.17

# Move your kaggle.json to the right place
mkdir -p ~/.kaggle
mv ~/Downloads/kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json   # only you can read it

# Verify it works
kaggle datasets list --sort-by votes | head -5

# Download a dataset (Titanic — you'll use this in Phase 2)
kaggle competitions download -c titanic -p ~/terminal-practice/data/
ls ~/terminal-practice/data/
```
---

## Common Mistakes

**Mistake 1: `rm -rf` without verifying the path first**
There is no recycle bin in the terminal. `rm -rf wrong_folder/` is permanent. Always run `ls wrong_folder/` first to confirm you're deleting what you think you're deleting.

**Mistake 2: Colab session timeouts losing your work**
Colab disconnects after ~12 hours of inactivity and erases all files. Every Colab notebook that produces outputs (trained models, processed datasets) should save them to Google Drive:
```python
model.save('/content/drive/MyDrive/ai-engineering-180/my_model.pt')
```

**Mistake 3: Forgetting to activate the T4 GPU before training**
If you forget to switch to T4 runtime, your training runs on Colab's CPU — 10× slower. Always check: Runtime → Change runtime type → confirm T4 is selected.

**Mistake 4: Installing packages globally in Colab and losing them on reconnect**
In Colab, every `!pip install` runs fresh each session. Put your installs in the first cell of every notebook so they re-run automatically when you reconnect:
```python
# First cell — always
!pip install -q unsloth peft trl  # -q = quiet mode (less noise)
```

---

## One-Sentence Lesson

The terminal is the only interface on every production ML server, and Colab/Kaggle are the free GPU environments that let you train models your laptop can't handle — mastering both today means you're ready for every hands-on session from here to Day 180.

---

*Day 2 / 180 complete ✅ | Tomorrow: ML Project Structure + NeuralCorp Scaffold — the template every project in this curriculum is built on*
