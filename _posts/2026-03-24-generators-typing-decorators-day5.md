---
layout: single
title: "Day 5 of 180 - List Comprehensions, Generators, Decorators, Typing Hints, Dataclasses & pathlib"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## What Is This?

Day 5 covers six Python superpowers that you'll use every single day as an ML engineer:

1. **List Comprehensions** - build lists in one expressive line
2. **Generators** - stream data lazily without loading it all into RAM
3. **Decorators** - wrap functions with reusable behaviour (timing, retrying, validating)
4. **Typing Hints** - label your functions so tools and teammates know what they expect
5. **Dataclasses** - auto-generate boilerplate Python classes (great for ML configs)
6. **pathlib** - handle file paths cleanly on any operating system

---

## The Analogy

**List comprehension** = an assembly line that transforms every item in one motion.

**Generator** = a popcorn machine that makes one kernel at a time, only when you ask - instead of filling a bathtub with popcorn first.

**Decorator** = gift-wrapping a present. The present (your function) stays the same; the wrapping adds a bow (timing, retry logic, validation) without touching the present.

**Typing hints** = the label on a food delivery box. You see "Burger: beef, bun, lettuce" before opening it - no surprises.

**Dataclass** = a smart form template. Fill in the field names and types; Python writes the boring __init__, __repr__, __eq__ code for you.

**pathlib** = a GPS for your file system. Describe where you want to go; it handles Mac vs Linux vs Windows road rules automatically.

---

## The Concept Explained Simply

### List Comprehensions

Old way:
```python
squares = []
for x in range(10):
    squares.append(x * x)
```

New way (list comprehension):
```python
squares = [x * x for x in range(10)]
```

With a filter (only even squares):
```python
even_squares = [x * x for x in range(10) if x % 2 == 0]
# → [0, 4, 16, 36, 64]
```

Pattern: `[expression  for  item  in  iterable  if  condition]`

---

### Generators

A list stores ALL items in memory at once. A generator computes and hands you **one item at a time**.

```python
# List - allocates ~8 MB for 1 million items
big_list = [x * x for x in range(1_000_000)]

# Generator expression - always ~104 bytes, regardless of size
big_gen  = (x * x for x in range(1_000_000))
```

Generator function (using `yield`):
```python
def batch_generator(data, batch_size=32):
    for start in range(0, len(data), batch_size):
        yield data[start : start + batch_size]  # Pause here, hand batch to caller

# Usage
for batch in batch_generator(my_data, batch_size=32):
    train_on(batch)  # Process one batch, then ask for next
```

Why it matters in ML: PyTorch's `DataLoader` is a generator. It streams training batches from disk without loading the 100 GB ImageNet dataset into your 16 GB RAM.

---

### Decorators

A decorator is a function that takes a function and returns an enhanced version of it.

```python
import functools
import time

def timer(func):
    @functools.wraps(func)   # Preserve original function name/docs
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)   # Call the original
        elapsed = time.perf_counter() - start
        print(f"⏱  {func.__name__}() took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def train_epoch(model, data):
    ...  # Your training code here
# Now every call to train_epoch() automatically prints its duration
```

The `@decorator_name` syntax is just shorthand for:
```python
train_epoch = timer(train_epoch)
```

Common decorators you'll use in production ML:
- `@torch.no_grad()` - disables gradient tracking during inference
- `@lru_cache(maxsize=128)` - caches expensive computation results
- `@app.get("/predict")` - registers a FastAPI route handler
- `@retry(max_attempts=3)` - auto-retries failed LLM API calls

---

### Typing Hints

```python
from typing import List, Dict, Optional, Tuple

def preprocess_texts(
    texts: List[str],
    max_length: int = 512,
    pad: bool = True,
) -> Dict[str, List[int]]:
    """Returns tokenised input_ids and attention_mask."""
    ...
```

Python doesn't enforce these at runtime - they're labels for humans and tools. Tools like **mypy** and **pyright** (built into VS Code) will catch type errors before you run the code.

From Python 3.10+ you can use the cleaner union syntax:
```python
def load(path: str | None = None) -> list[float]:
    ...
```

---

### Dataclasses

Without dataclass - lots of boilerplate:
```python
class TrainingConfig:
    def __init__(self, lr, batch_size, epochs):
        self.lr = lr
        self.batch_size = batch_size
        self.epochs = epochs
    def __repr__(self):
        return f"TrainingConfig(lr={self.lr}, ...)"  # tedious
```

With `@dataclass` - Python writes all of that for you:
```python
from dataclasses import dataclass, field

@dataclass
class TrainingConfig:
    lr: float = 3e-4
    batch_size: int = 32
    epochs: int = 10
    tags: list = field(default_factory=list)

cfg = TrainingConfig(lr=1e-3)
print(cfg)  # → TrainingConfig(lr=0.001, batch_size=32, epochs=10, tags=[])
```

Add `frozen=True` to make the config immutable (great for experiment reproducibility).

---

### pathlib

Old string approach - fragile:
```python
path = "data/" + "raw/" + "train.csv"  # Breaks on Windows (\\ vs /)
```

pathlib approach - clean and cross-platform:
```python
from pathlib import Path

# Build paths with / operator
data_path = Path("data") / "raw" / "train.csv"

# Common operations
data_path.exists()                          # Does it exist?
data_path.parent.mkdir(parents=True, exist_ok=True)  # Create parent dirs
data_path.suffix                            # '.csv'
data_path.stem                              # 'train'

# Find all CSV files recursively
csv_files = sorted(Path("data").rglob("*.csv"))

# Read a text file
text = Path("README.md").read_text(encoding="utf-8")
```

---

## How Real Companies Use This

- **Netflix / Spotify** data pipelines: list comprehensions to build feature vectors from event logs in seconds
- **OpenAI / Anthropic** training loops: generator-based data loaders stream terabytes of text without memory overflow
- **Every FastAPI endpoint at any ML company**: uses `@app.post("/predict")` - a decorator registering the route
- **Google DeepMind** configs: Pydantic models (dataclass-like) store all hyperparameters, logged to W&B as a dict
- **AWS SageMaker** training scripts: pathlib used to construct paths to S3-mounted training data directories

---

## Step-by-Step: Try It Yourself

### Environment Setup

```bash
mkdir nc_005_pythonic_utils
cd nc_005_pythonic_utils
python3 -m venv .venv
source .venv/bin/activate        # Mac/Linux
# .venv\Scripts\activate         # Windows

pip install pydantic==2.6.4 pydantic-settings==2.2.1 pytest==8.1.1 python-dotenv==1.0.1
echo 'APP_ENV=development
DATA_DIR=./data
LOG_LEVEL=INFO' > .env
```

### Quick Demo - Try Each Concept

**1. List comprehensions:**
```python
# normalise scores from 0–100 to 0–1
scores = [78.5, 92.0, 45.0, 100.0]
normalised = [s / 100.0 for s in scores]
print(normalised)
# → [0.785, 0.92, 0.45, 1.0]

# filter: only high-confidence predictions
preds = [("cat", 0.95), ("dog", 0.60), ("bird", 0.85)]
high_conf = [label for label, conf in preds if conf >= 0.80]
print(high_conf)
# → ['cat', 'bird']
```

**2. Generator memory comparison:**
```python
import sys

big_list = [x * x for x in range(1_000_000)]
big_gen  = (x * x for x in range(1_000_000))

print(f"List:      {sys.getsizeof(big_list):>10,} bytes")
print(f"Generator: {sys.getsizeof(big_gen):>10,} bytes")
# List:       8,697,464 bytes
# Generator:        104 bytes
```

**3. Write and use a batch generator:**
```python
def batch_generator(data, batch_size=3):
    for i in range(0, len(data), batch_size):
        yield data[i : i + batch_size]

training_data = list(range(10))
for batch_num, batch in enumerate(batch_generator(training_data)):
    print(f"Batch {batch_num}: {batch}")
# Batch 0: [0, 1, 2]
# Batch 1: [3, 4, 5]
# Batch 2: [6, 7, 8]
# Batch 3: [9]
```

**4. Build and use decorators:**
```python
import functools, time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"⏱  {func.__name__}() = {time.perf_counter()-t0:.4f}s")
        return result
    return wrapper

@timer
def compute_mean(values):
    return sum(values) / len(values)

print(compute_mean(list(range(100_000))))
# ⏱  compute_mean() = 0.0031s
# 49999.5
```

**5. Typing hints and dataclasses:**
```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class ModelConfig:
    architecture: str = "resnet50"
    hidden_dim: int = 512
    dropout: float = 0.1
    tags: List[str] = field(default_factory=list)

cfg = ModelConfig(architecture="vit-base", dropout=0.2)
print(cfg)
# ModelConfig(architecture='vit-base', hidden_dim=512, dropout=0.2, tags=[])
```

**6. pathlib for ML project setup:**
```python
from pathlib import Path

root = Path("my_ml_project")
dirs = ["data/raw", "data/processed", "models/checkpoints", "logs"]

for d in dirs:
    (root / d).mkdir(parents=True, exist_ok=True)
    print(f"Created: {root / d}")

# Build a checkpoint filename
epoch, val_loss = 12, 0.2341
ckpt = root / "models" / "checkpoints" / f"resnet50_epoch{epoch:03d}_loss{val_loss:.4f}.pt"
print(f"Checkpoint: {ckpt}")
# Checkpoint: my_ml_project/models/checkpoints/resnet50_epoch012_loss0.2341.pt
```

### Expected Output (all 6 demos)
All code above runs without errors. Memory demo shows generator is ~80,000x smaller. Timer shows sub-millisecond timing. Directories are created.

---

## ☁️ Cloud Note

Everything in Day 5 is pure Python - no GPU needed. Runs perfectly on your M1 MacBook Air. No Colab or RunPod required today.

---

## Common Mistakes (3 errors + fixes)

**Error 1:**
```
ModuleNotFoundError: No module named 'src'
```
Fix: Always run `pytest tests/ -v` from the project root directory (`nc_005_pythonic_utils/`), not from inside `src/` or `tests/`.
Why: Python adds the current directory to `sys.path`. The `src` package is only visible from the root.

---

**Error 2:**
```
RecursionError / infinite loop when consuming a generator
```
Fix: Infinite generators (`while True` + `yield`) need you to stop them. Use:
```python
import itertools
first_10_batches = list(itertools.islice(infinite_gen, 10))
```
Why: Generators don't stop themselves - the caller controls when to stop.

---

**Error 3:**
```
pathlib.Path.glob() returns [] but files exist
```
Fix: Use `rglob()` for recursive search across all subdirectories.
```python
# Wrong - only immediate directory
files = Path("data").glob("*.csv")

# Correct - searches all subdirectories too
files = Path("data").rglob("*.csv")
```
Why: `glob()` is one level deep; `rglob()` adds `**/` recursion automatically.

---

## One-Sentence Lesson

List comprehensions, generators, decorators, type hints, dataclasses, and pathlib are the six Pythonic tools you'll use in nearly every single ML engineering file you write - master them now and every phase ahead gets dramatically easier.
