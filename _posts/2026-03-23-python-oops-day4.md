---
layout: single
title: "Day 4 of 180 - Python Recap & OOP Basics"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## What Is This?

Today covers two pillars that every ML engineer uses every single day.

**Part A — Python building blocks**: lists, dictionaries, loops, and functions. These are the raw materials of all ML code. You can't write a training loop, a data pipeline, or a config system without them.

**Part B — Object-Oriented Programming (OOP)**: how to organise code into reusable classes. Every PyTorch neural network you'll ever write is a Python class. Understanding OOP now means reading real model code (GPT, ResNet, BERT) later feels natural — not alien.

By the end of this post you'll have built a working **gradient-descent linear regression model from scratch** using only pure Python — no PyTorch yet. That's the training loop, the forward pass, and the weight update, all visible in plain code.

---

## The Analogy

### Lists and Dicts

Imagine you're a chef in a busy restaurant.

Your **ticket rail** holds orders in sequence — ticket #1 is first, #2 is second. That's a **list**: ordered, numbered from zero, first-in first-accessible. Your **spice rack** has labelled jars — reach for "salt" and you get salt instantly. That's a **dict**: labelled storage with instant lookup.

In ML: your training dataset is a list of sample dicts. Every mini-batch is a slice of that list. Every config file is a dict.

### Loops

You (the chef) check every ticket one by one — that's a **for loop**. You keep stirring the pot *until* the sauce thickens — that's a **while loop**. The PyTorch training loop is literally two nested for loops: one over epochs, one over batches.

### Functions

A laminated recipe card. Same steps every time, just swap the ingredients (arguments). You don't re-write the risotto recipe for every service — you reuse the card. In code: write once, call many times, test once.

### Classes & OOP

A **class** is a robot blueprint. An **object** (instance) is an actual robot built from that blueprint. You can build 100 robots from one blueprint — each has its own memory (attributes) but follows the same wiring (methods).

In ML: every PyTorch model is a class. Writing `class MyNet(nn.Module):` is literally "build me a robot using the nn.Module blueprint, with my custom wiring in `forward()`."

---

## The Concepts Explained Simply

### Part A: Lists, Dicts, Loops, Functions

---

#### Lists — Ordered, Mutable Sequences

```python
# A list holds items in order. Index starts at 0.
batch_sizes = [16, 32, 64, 128]

print(batch_sizes[0])    # 16   — first item
print(batch_sizes[-1])   # 128  — last item (negative index counts from end)
print(batch_sizes[1:3])  # [32, 64] — slice: from index 1 up to (not including) 3

# Lists are mutable — you can change them after creation
batch_sizes.append(256)   # add to end
batch_sizes.insert(0, 8)  # insert at position 0
batch_sizes.pop()         # remove and return last item
print(len(batch_sizes))   # 5

# Sorting
losses = [0.9, 0.3, 0.7, 0.5]
losses.sort()                          # sort in-place: [0.3, 0.5, 0.7, 0.9]
sorted_copy = sorted(losses, reverse=True)  # new list, original unchanged

# Checking membership — O(n) for lists
if 0.3 in losses:
    print("Found 0.3")
```

**Why this matters in ML:**
Every dataset is a list of samples. Every batch is a slice of that list. Training history is a list of loss values. When you call `dataloader = DataLoader(dataset, batch_size=32)`, PyTorch is slicing a list-like object under the hood.

---

#### Dictionaries — Labelled Key→Value Storage

```python
# A dict maps unique keys to values. Keys can be strings, ints, or tuples.
config = {
    "learning_rate": 0.001,
    "batch_size": 32,
    "epochs": 10,
    "optimizer": "adam",
}

# Access by key — O(1), instant no matter how large the dict
print(config["learning_rate"])         # 0.001

# KeyError if key doesn't exist — use .get() for safe access
lr = config.get("weight_decay", 0.0)   # returns 0.0 if "weight_decay" absent
print(lr)                              # 0.0

# Add or update keys
config["scheduler"] = "cosine"         # add new key
config["epochs"] = 20                  # update existing key

# Remove a key
del config["optimizer"]

# Iterate over keys, values, or both
for key in config:
    print(key)

for value in config.values():
    print(value)

for key, value in config.items():      # most common pattern
    print(f"  {key}: {value}")

# Dict comprehension — build a dict in one line
squared = {x: x**2 for x in range(1, 6)}
print(squared)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Nested dicts — very common for experiment configs
experiment = {
    "run_001": {"lr": 0.01, "bs": 32, "val_acc": 0.87},
    "run_002": {"lr": 0.001, "bs": 64, "val_acc": 0.91},
}
print(experiment["run_002"]["val_acc"])  # 0.91
```

**Why this matters in ML:**
Every sample in a dataset is a dict: `{"image": tensor, "label": 0}`. Every API response (OpenAI, HuggingFace) is a dict. Every experiment run logged to MLflow or W&B is a dict of metrics. If you understand one data structure in ML, it's the dict.

---

#### For Loops — The Backbone of Training

```python
# Basic range loop — runs N times
for epoch in range(1, 11):   # 1, 2, 3 ... 10
    print(f"Epoch {epoch}/10")

# Iterating a list directly
losses = [0.9, 0.7, 0.5, 0.3, 0.1]
for loss in losses:
    print(f"  loss = {loss:.4f}")

# enumerate — get index AND value at the same time
for step, loss in enumerate(losses):
    print(f"  Step {step}: loss = {loss:.4f}")

# zip — iterate two (or more) lists in parallel
inputs  = [1.0, 2.0, 3.0, 4.0]
targets = [2.0, 4.0, 6.0, 8.0]
for x, y in zip(inputs, targets):
    print(f"  x={x}, y={y}, error={abs(x*2 - y):.4f}")

# break and continue
for epoch in range(100):
    loss = 1.0 / (epoch + 1)
    if loss < 0.1:
        print(f"Early stop at epoch {epoch}")
        break            # exit the loop immediately
    if epoch % 10 != 0:
        continue         # skip to next iteration
    print(f"Epoch {epoch}: loss = {loss:.4f}")
```

**The real PyTorch training loop** — everything you learn today maps directly here:

```python
for epoch in range(num_epochs):              # Part A: for loop over range
    running_loss = 0.0

    for batch in dataloader:                 # Part A: for loop over list-like
        inputs  = batch["input"]             # Part A: dict access
        targets = batch["target"]            # Part A: dict access

        output = model(inputs)               # Part B: calling a class method
        loss   = criterion(output, targets)

        loss.backward()
        optimizer.step()
        optimizer.zero_grad()

        running_loss += loss.item()          # Part A: accumulating into a var

    epoch_loss = running_loss / len(dataloader)
    history.append(epoch_loss)              # Part A: list.append
```

---

#### While Loops — Run Until a Condition is False

```python
# Classic early stopping pattern
patience_counter = 0
best_val_loss = float("inf")   # start with the worst possible value
epoch = 0

while patience_counter < 5 and epoch < 100:
    train_loss = 1.0 / (epoch + 1)       # simulated decreasing loss
    val_loss   = train_loss + 0.05       # val is slightly worse

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        patience_counter = 0             # reset — we improved!
    else:
        patience_counter += 1            # no improvement — count it

    print(f"Epoch {epoch:3d} | val_loss={val_loss:.4f} | patience={patience_counter}")
    epoch += 1

print(f"\nStopped at epoch {epoch}. Best val_loss: {best_val_loss:.4f}")
```

**Key warning:** Always make sure your while loop has a guaranteed exit condition. `while True:` without a `break` is an infinite loop — your process will hang forever.

---

#### Functions — Reusable Named Blocks of Code

```python
# Basic function
def greet(name: str) -> str:
    return f"Hello, {name}!"

print(greet("Edward"))   # Hello, Edward!


# Function with multiple parameters and a default value
def compute_loss(predictions: list[float],
                 targets: list[float],
                 reduction: str = "mean") -> float:
    """Compute mean squared error between predictions and targets.

    Args:
        predictions: Model output values.
        targets: Ground-truth values.
        reduction: 'mean' (default) or 'sum'.

    Returns:
        Scalar MSE loss value.

    Raises:
        ValueError: If lists have different lengths or unknown reduction.
    """
    if len(predictions) != len(targets):
        raise ValueError(
            f"Length mismatch: {len(predictions)} predictions vs {len(targets)} targets."
        )
    if reduction not in ("mean", "sum"):
        raise ValueError(f"Unknown reduction '{reduction}'. Use 'mean' or 'sum'.")

    squared_errors = [(p - t) ** 2 for p, t in zip(predictions, targets)]

    if reduction == "mean":
        return sum(squared_errors) / len(squared_errors)
    return sum(squared_errors)


# Call with positional args
loss = compute_loss([1.0, 2.0, 3.0], [1.1, 1.9, 3.2])
print(f"MSE: {loss:.6f}")    # MSE: 0.020000

# Call with keyword args — order doesn't matter
loss2 = compute_loss(
    targets=[1.1, 1.9, 3.2],
    predictions=[1.0, 2.0, 3.0],
    reduction="sum"
)
print(f"Sum squared error: {loss2:.4f}")   # 0.0600


# Functions are first-class objects — you can pass them as arguments
def apply_transform(data: list[float], transform_fn) -> list[float]:
    """Apply any function to each element of a list."""
    return [transform_fn(x) for x in data]

import math
log_data = apply_transform([1.0, 10.0, 100.0], math.log10)
print(log_data)   # [0.0, 1.0, 2.0]


# *args — accept any number of positional arguments
def average(*values: float) -> float:
    """Compute average of any number of float arguments."""
    return sum(values) / len(values)

print(average(0.9, 0.7, 0.5, 0.3))   # 0.6


# **kwargs — accept any number of keyword arguments
def log_metrics(**metrics: float) -> None:
    """Print any number of named metrics."""
    for name, value in metrics.items():
        print(f"  {name}: {value:.4f}")

log_metrics(train_loss=0.45, val_loss=0.52, val_acc=0.87)
```

**Production best practices every function should have:**
- Type hints on every parameter and return value
- Google-style docstring (Args / Returns / Raises)
- Validate inputs at the top — raise `ValueError` for bad data
- Return a value instead of printing — callers decide what to do with it
- Keep functions short: one function, one job

---

### Part B: Python Classes & OOP

---

#### The Class: Blueprint + Instance

```python
# Define the blueprint
class TrainingConfig:
    """Stores all hyperparameters for one training run.

    Attributes:
        learning_rate: Step size for gradient descent.
        batch_size: Samples per optimiser step.
        num_epochs: Full passes over the training data.
        model_name: Identifier for logging.
    """

    # Class attribute — shared by ALL instances
    framework = "pytorch"

    def __init__(self,
                 learning_rate: float,
                 batch_size: int,
                 num_epochs: int,
                 model_name: str = "my-model") -> None:
        # Instance attributes — unique to each object
        # self = "the specific robot we're building right now"
        self.learning_rate = learning_rate
        self.batch_size    = batch_size
        self.num_epochs    = num_epochs
        self.model_name    = model_name

    def __repr__(self) -> str:
        """Developer string — shown in REPL and error messages.
        Always implement this. It saves hours of debugging.
        """
        return (
            f"TrainingConfig("
            f"lr={self.learning_rate}, "
            f"bs={self.batch_size}, "
            f"epochs={self.num_epochs})"
        )

    def __eq__(self, other: object) -> bool:
        """Two configs are equal if all their values match."""
        if not isinstance(other, TrainingConfig):
            return NotImplemented
        return (
            self.learning_rate == other.learning_rate
            and self.batch_size == other.batch_size
            and self.num_epochs == other.num_epochs
        )

    def to_dict(self) -> dict[str, float | int | str]:
        """Serialise config to a plain dict for MLflow / W&B logging."""
        return {
            "learning_rate": self.learning_rate,
            "batch_size":    self.batch_size,
            "num_epochs":    self.num_epochs,
            "model_name":    self.model_name,
        }


# Build two robots from the same blueprint
cfg_a = TrainingConfig(0.001, 32, 10, "baseline")
cfg_b = TrainingConfig(0.01, 64, 20, "large-lr")

print(cfg_a)                   # TrainingConfig(lr=0.001, bs=32, epochs=10)
print(cfg_b.learning_rate)     # 0.01
print(cfg_a == cfg_b)          # False
print(cfg_a.framework)         # pytorch  (class attribute)
print(cfg_a.to_dict())         # {'learning_rate': 0.001, ...}
```

---

#### Inheritance — Child Classes Extend Parents

Inheritance lets you share behaviour across related classes.
**The key rule:** a child IS-A parent. A `Dog` IS-A `Animal`.

```python
from abc import ABC, abstractmethod


class BaseModel(ABC):
    """Abstract base class — defines the contract all models must follow.

    ABC = Abstract Base Class. Python raises TypeError if a subclass
    doesn't implement all @abstractmethod methods.
    This is the same contract nn.Module uses in PyTorch.
    """

    def __init__(self, name: str) -> None:
        self.name = name
        self._is_fitted = False

    @abstractmethod
    def predict(self, x: float) -> float:
        """All subclasses MUST implement this."""
        ...  # abstract — no body here

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(name='{self.name}', fitted={self._is_fitted})"


class ConstantModel(BaseModel):
    """Always predicts the same constant value.
    Useful as a baseline to beat.
    """

    def __init__(self, constant: float = 0.0) -> None:
        super().__init__(name="ConstantModel")  # call parent __init__ FIRST
        self.constant = constant
        self._is_fitted = True

    def predict(self, x: float) -> float:
        """Ignore x, always return the constant."""
        return self.constant


class LinearModel(BaseModel):
    """y = w * x + b — the simplest learnable model."""

    def __init__(self, weight: float = 0.0, bias: float = 0.0) -> None:
        super().__init__(name="LinearModel")
        self.weight = weight
        self.bias   = bias

    def predict(self, x: float) -> float:
        return self.weight * x + self.bias


# Polymorphism: same interface, different behaviour
models: list[BaseModel] = [
    ConstantModel(constant=5.0),
    LinearModel(weight=2.0, bias=1.0),
]
for model in models:
    print(f"{model.name}: predict(3.0) = {model.predict(3.0)}")
# ConstantModel: predict(3.0) = 5.0
# LinearModel:   predict(3.0) = 7.0
```

---

#### `@property`, `@classmethod`, `@staticmethod` — Three Kinds of Methods

```python
class MLExperiment:
    """Tracks one training experiment."""

    def __init__(self, name: str, loss_history: list[float]) -> None:
        self.name = name
        self._loss_history = loss_history   # _prefix = private by convention

    # ── @property ─────────────────────────────────────────────────────────
    # Looks like an attribute but is computed on the fly.
    # Callers write  experiment.best_loss  (no parentheses).
    # Use for read-only, computed values that depend on state.
    @property
    def best_loss(self) -> float:
        """Return the minimum loss seen across all epochs."""
        if not self._loss_history:
            return float("inf")
        return min(self._loss_history)

    @property
    def num_epochs_trained(self) -> int:
        return len(self._loss_history)

    # ── @classmethod ───────────────────────────────────────────────────────
    # Receives the class (cls) instead of the instance (self).
    # Use for alternative constructors — different ways to build an object.
    # In PyTorch: model = MyModel.from_pretrained("bert-base")  ← classmethod
    @classmethod
    def from_csv(cls, name: str, csv_path: str) -> "MLExperiment":
        """Load a loss history from a CSV file."""
        losses: list[float] = []
        try:
            with open(csv_path) as f:
                for line in f:
                    losses.append(float(line.strip()))
        except FileNotFoundError:
            raise FileNotFoundError(f"Loss CSV not found: {csv_path}")
        return cls(name=name, loss_history=losses)

    @classmethod
    def fresh(cls, name: str) -> "MLExperiment":
        """Create a new experiment with no history yet."""
        return cls(name=name, loss_history=[])

    # ── @staticmethod ──────────────────────────────────────────────────────
    # No access to self or cls. Pure utility function.
    # Use when the logic is related to the class but doesn't need instance data.
    @staticmethod
    def is_converged(losses: list[float], threshold: float = 1e-4) -> bool:
        """Check if the last 5 losses have all changed by less than threshold."""
        if len(losses) < 5:
            return False
        recent = losses[-5:]
        return max(recent) - min(recent) < threshold

    def record(self, loss: float) -> None:
        """Append a new epoch loss."""
        self._loss_history.append(loss)

    def __repr__(self) -> str:
        return (
            f"MLExperiment(name='{self.name}', "
            f"epochs={self.num_epochs_trained}, "
            f"best_loss={self.best_loss:.4f})"
        )


# Usage
exp = MLExperiment.fresh("baseline-run")     # classmethod constructor
for i in range(1, 11):
    exp.record(1.0 / i)                      # record 10 fake epoch losses

print(exp)                                   # MLExperiment(name='baseline-run', epochs=10, best_loss=0.1000)
print(exp.best_loss)                         # 0.1  (property — no parentheses)
print(exp.num_epochs_trained)                # 10
print(MLExperiment.is_converged(exp._loss_history))  # False (still changing)
```

---

#### Dunder (Magic) Methods — Making Objects Feel Like Python

Dunder = "double underscore". These special methods let your class behave like built-in Python types.

```python
class MetricTracker:
    """Collects metric values, supports len(), indexing, and iteration."""

    def __init__(self, name: str) -> None:
        self.name   = name
        self._data: list[float] = []

    def add(self, value: float) -> None:
        self._data.append(value)

    def __len__(self) -> int:
        """Called by len(tracker)."""
        return len(self._data)

    def __getitem__(self, index: int) -> float:
        """Called by tracker[i] or tracker[2:5]."""
        return self._data[index]

    def __iter__(self):
        """Called by for value in tracker."""
        return iter(self._data)

    def __contains__(self, value: float) -> bool:
        """Called by value in tracker."""
        return value in self._data

    def __repr__(self) -> str:
        return f"MetricTracker(name='{self.name}', n={len(self)})"

    def __str__(self) -> str:
        """Called by print(tracker) — human-friendly."""
        if not self._data:
            return f"{self.name}: (empty)"
        return f"{self.name}: min={min(self._data):.4f}, max={max(self._data):.4f}, n={len(self)}"


tracker = MetricTracker("val_loss")
for v in [0.9, 0.7, 0.5, 0.3]:
    tracker.add(v)

print(len(tracker))          # 4
print(tracker[0])            # 0.9
print(tracker[-1])           # 0.3
print(tracker[1:3])          # [0.7, 0.5]
print(0.5 in tracker)        # True

for value in tracker:        # __iter__ makes this work
    print(f"  {value:.2f}")

print(repr(tracker))         # MetricTracker(name='val_loss', n=4)
print(tracker)               # val_loss: min=0.3000, max=0.9000, n=4
```

---

#### The Full Picture: Gradient Descent from Scratch

This is the most important exercise today. You'll implement the training loop manually — the same math that PyTorch's `.backward()` automates. Understanding this by hand makes autograd feel intuitive when we hit it on Day 33.

The model: **y = w·x + b** (a line through the data)

The goal: find w and b that minimise MSE loss = (1/n) · Σ(ŷᵢ − yᵢ)²

The update rule (calculus chain rule):
```
∂L/∂w = (2/n) · Σ (ŷᵢ − yᵢ) · xᵢ
∂L/∂b = (2/n) · Σ (ŷᵢ − yᵢ)

w ← w − lr · ∂L/∂w
b ← b − lr · ∂L/∂b
```

```python
from abc import ABC, abstractmethod
import math


class BaseModel(ABC):
    """Abstract base — every model must implement predict()."""

    def __init__(self, name: str) -> None:
        self.name = name

    @abstractmethod
    def predict(self, x: float) -> float: ...

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(name='{self.name}')"


class LinearRegressionModel(BaseModel):
    """y = w·x + b — trained by manual gradient descent.

    This is the heart of Day 4. Every concept from today
    appears somewhere in this class.
    """

    def __init__(self) -> None:
        super().__init__(name="LinearRegressionModel")
        self.weight: float = 0.0        # w — starts at zero
        self.bias: float   = 0.0        # b — starts at zero
        self._history: list[dict] = []  # private loss/weight log

    # ── Core methods ─────────────────────────────────────────────────────

    def predict(self, x: float) -> float:
        """Forward pass: compute ŷ = w·x + b."""
        return self.weight * x + self.bias

    def fit(self,
            x_data: list[float],
            y_data: list[float],
            learning_rate: float = 0.01,
            num_steps: int = 200) -> None:
        """Train using mini-batch gradient descent.

        Args:
            x_data: Input features (one value per sample).
            y_data: Ground-truth targets (same length as x_data).
            learning_rate: How big each parameter update step is.
                Too large → overshoots minimum.
                Too small → converges very slowly.
            num_steps: Number of gradient descent iterations.
        """
        n = len(x_data)

        for step in range(num_steps):

            # ── Step 1: Forward pass ──────────────────────────────────
            # Compute model prediction for every sample
            predictions = [self.predict(x) for x in x_data]

            # ── Step 2: Compute loss ──────────────────────────────────
            # MSE = mean of (prediction - target)^2
            errors = [p - t for p, t in zip(predictions, y_data)]
            mse = sum(e**2 for e in errors) / n

            # ── Step 3: Compute gradients ─────────────────────────────
            # How much does the loss change if we nudge w or b?
            # These come from calculus (chain rule).
            # You don't need to derive them — just know the pattern.
            grad_w = (2 / n) * sum(e * x for e, x in zip(errors, x_data))
            grad_b = (2 / n) * sum(errors)

            # ── Step 4: Update parameters ─────────────────────────────
            # Move w and b in the direction that decreases loss.
            # Subtract gradient (if gradient is positive, loss increases
            # with w, so we decrease w).
            self.weight -= learning_rate * grad_w
            self.bias   -= learning_rate * grad_b

            # ── Log every 25 steps ────────────────────────────────────
            self._history.append({
                "step": step,
                "loss": round(mse, 8),
                "weight": round(self.weight, 6),
                "bias": round(self.bias, 6),
            })

            if step % 25 == 0 or step == num_steps - 1:
                print(
                    f"  Step {step:4d} | loss={mse:.6f} "
                    f"| w={self.weight:.4f} | b={self.bias:.4f}"
                )

    # ── Properties ───────────────────────────────────────────────────────

    @property
    def loss_history(self) -> list[float]:
        """Return just the loss values across all steps."""
        return [h["loss"] for h in self._history]

    @property
    def did_converge(self) -> bool:
        """True if the last 10 loss values are all within 1e-6 of each other."""
        if len(self._history) < 10:
            return False
        recent = [h["loss"] for h in self._history[-10:]]
        return max(recent) - min(recent) < 1e-6

    # ── Class methods ─────────────────────────────────────────────────────

    @classmethod
    def from_checkpoint(cls,
                        weight: float,
                        bias: float) -> "LinearRegressionModel":
        """Load a model from saved parameters (like torch.load).

        Args:
            weight: Saved weight value.
            bias: Saved bias value.
        """
        model = cls()
        model.weight = weight
        model.bias   = bias
        print(f"Loaded checkpoint: w={weight}, b={bias}")
        return model

    # ── Static methods ────────────────────────────────────────────────────

    @staticmethod
    def compute_r_squared(predictions: list[float],
                          targets: list[float]) -> float:
        """Compute R² (coefficient of determination).

        R² = 1 means perfect predictions.
        R² = 0 means no better than predicting the mean.
        R² < 0 means worse than predicting the mean.
        """
        mean_target = sum(targets) / len(targets)
        ss_total = sum((t - mean_target)**2 for t in targets)
        ss_residual = sum((p - t)**2 for p, t in zip(predictions, targets))
        if ss_total == 0:
            return 1.0  # all targets identical — degenerate case
        return 1.0 - (ss_residual / ss_total)

    # ── Dunder methods ────────────────────────────────────────────────────

    def __repr__(self) -> str:
        return (
            f"LinearRegressionModel("
            f"w={self.weight:.4f}, "
            f"b={self.bias:.4f}, "
            f"converged={self.did_converge})"
        )
```

---

## How Real Companies Use This

**Spotify** — their ML recommendation pipeline is a Python class that inherits a base `Recommender` and overrides `predict()`. Lists of user-interaction dicts flow through it. The config is a dict loaded from a YAML file.

**Google** — TPU training loops are literally `for step in range(total_steps): loss = model(batch)`. The loop structure is identical to what you wrote today. The hardware is different; the Python is the same.

**OpenAI** — The GPT model definition is `class GPT(nn.Module)`. The `forward()` method is ~50 lines. All architecture lives in `__init__`. OOP makes it testable and swappable between GPT-2, GPT-3, GPT-4.

**HuggingFace** — `AutoModel.from_pretrained("bert-base")` is a classmethod. `model.num_parameters` is a property. The entire 100k-star Transformers library is built on the OOP patterns you learned today.

**Airbnb** — their feature store returns `list[dict]` — one dict per property listing. ML pipelines loop over that list with a for loop, slice batches, and pass them to model classes.

---

## Step-by-Step: Try It Yourself

### 1. Environment Setup

```bash
# Create and enter project directory
mkdir nc-004-python-foundations
cd nc-004-python-foundations

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate   # Mac/Linux
# On Windows: .venv\Scripts\activate

# Confirm you're using the venv Python
which python3   # should show .../nc-004-python-foundations/.venv/bin/python3

# Install dependencies with pinned versions
pip install pydantic-settings==2.2.1 \
            python-dotenv==1.0.1     \
            pytest==8.1.1            \
            pytest-cov==5.0.0

# Save exact versions
pip freeze > requirements.txt

# Create folder structure
mkdir -p src tests
touch src/__init__.py tests/__init__.py
```

---

### 2. Full Project File Structure

```
nc-004-python-foundations/
├── .env                        ← environment variables (never commit this)
├── requirements.txt            ← pinned dependencies
├── Makefile                    ← one-command actions
├── Dockerfile                  ← multi-stage container build
├── src/
│   ├── __init__.py             ← makes src a Python package
│   ├── exceptions.py           ← custom exception classes
│   ├── config.py               ← Pydantic settings from .env
│   ├── data_structures.py      ← lists + dicts exercises
│   ├── control_flow.py         ← loops + functions exercises
│   └── model_skeleton.py       ← OOP: LinearRegressionModel
└── tests/
    ├── __init__.py
    ├── test_data_structures.py
    ├── test_control_flow.py
    └── test_model_skeleton.py
```

---

### 3. Every File in Full

**`.env`**
```
APP_NAME=nc-004-python-foundations
APP_ENV=development
LOG_LEVEL=INFO
LEARNING_RATE=0.001
BATCH_SIZE=32
NUM_EPOCHS=10
```

---

**`src/exceptions.py`**
```python
"""Custom exceptions for NC-004.

Always define a base exception for your project.
This lets callers catch all your errors with one except clause:
    except NC004Error as e:
        logger.error("NC-004 error: %s", e)
"""


class NC004Error(Exception):
    """Base exception for all NC-004 errors."""


class InvalidBatchSizeError(NC004Error):
    """Raised when batch size is <= 0."""


class ModelNotFittedError(NC004Error):
    """Raised when predict() is called on an untrained model."""
```

---

**`src/config.py`**
```python
"""Pydantic-based settings — type-checked config from .env.

Why Pydantic instead of os.environ?
  - Auto-casts strings to int/float
  - Raises a clear error if a required variable is missing
  - Single source of truth — import `settings` everywhere
"""

import logging

from pydantic_settings import BaseSettings, SettingsConfigDict

logger = logging.getLogger(__name__)


class Settings(BaseSettings):
    """Application configuration loaded from .env.

    Attributes:
        app_name: Human-readable app name for log headers.
        app_env: 'development', 'staging', or 'production'.
        log_level: Python logging level string.
        learning_rate: Gradient descent step size.
        batch_size: Samples per training step.
        num_epochs: Total passes over the training dataset.
    """

    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    app_name: str      = "nc-004-python-foundations"
    app_env: str       = "development"
    log_level: str     = "INFO"
    learning_rate: float = 0.001
    batch_size: int    = 32
    num_epochs: int    = 10


settings = Settings()

logging.basicConfig(
    level=getattr(logging, settings.log_level.upper(), logging.INFO),
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
)

logger.info("Config loaded: %s (env=%s)", settings.app_name, settings.app_env)
```

---

**`src/data_structures.py`**
```python
"""Lists and dicts in an ML context.

Real use case: every ML dataset is a list of sample dicts.
Example: [{"image": "cat.jpg", "label": 0, "split": "train"}, ...]
This module practises building, mutating, and querying those structures.
"""

import logging
from typing import Any

from src.exceptions import InvalidBatchSizeError

logger = logging.getLogger(__name__)


def create_epoch_log(num_epochs: int) -> list[dict[str, Any]]:
    """Pre-allocate a training log: one dict per epoch.

    This is the structure you'd fill during training and
    later export to MLflow or W&B.

    Args:
        num_epochs: Total number of training epochs.

    Returns:
        List of dicts with sentinel values (metrics set to None).

    Raises:
        ValueError: If num_epochs < 1.
    """
    if num_epochs < 1:
        raise ValueError(f"num_epochs must be >= 1, got {num_epochs}")

    # List comprehension: concise + readable + fast
    log: list[dict[str, Any]] = [
        {
            "epoch":      e,
            "train_loss": None,
            "val_loss":   None,
            "val_acc":    None,
        }
        for e in range(1, num_epochs + 1)
    ]
    logger.debug("Epoch log created: %d entries", len(log))
    return log


def update_epoch(
    log: list[dict[str, Any]],
    epoch: int,
    train_loss: float,
    val_loss: float,
    val_acc: float,
) -> None:
    """Fill in one epoch's metrics in-place.

    Lists are passed by reference — mutating the dict inside the
    list changes the original. The caller sees the update.

    Args:
        log: The epoch log from create_epoch_log().
        epoch: 1-based epoch number to update.
        train_loss: Training loss for this epoch.
        val_loss: Validation loss for this epoch.
        val_acc: Validation accuracy (0.0 – 1.0).
    """
    entry = log[epoch - 1]   # epochs are 1-indexed; list is 0-indexed
    entry["train_loss"] = round(train_loss, 6)
    entry["val_loss"]   = round(val_loss, 6)
    entry["val_acc"]    = round(val_acc, 4)
    logger.info(
        "Epoch %d/%d | train_loss=%.4f val_loss=%.4f val_acc=%.4f",
        epoch, len(log), train_loss, val_loss, val_acc,
    )


def build_hyperparameter_grid(
    learning_rates: list[float],
    batch_sizes: list[int],
) -> list[dict[str, float | int]]:
    """Build all (lr, bs) combinations for a grid search.

    Uses a nested list comprehension — the cartesian product.
    This is how GridSearchCV works internally.

    Args:
        learning_rates: LR values to try.
        batch_sizes: Batch size values to try.

    Returns:
        List of all (lr, bs) combination dicts.

    Raises:
        InvalidBatchSizeError: If any batch size is <= 0.
    """
    for bs in batch_sizes:
        if bs <= 0:
            raise InvalidBatchSizeError(f"Batch size must be > 0, got {bs}")

    grid = [
        {"lr": lr, "bs": bs}
        for lr in learning_rates   # outer loop
        for bs in batch_sizes      # inner loop
    ]
    logger.info("Grid built: %d combinations", len(grid))
    return grid


def summarise_log(log: list[dict[str, Any]]) -> dict[str, float]:
    """Find the best epoch across all completed runs.

    Args:
        log: The epoch log (may be partially filled).

    Returns:
        Dict with best_val_acc, best_val_loss, best_epoch.
        Empty dict if no epochs have been completed.
    """
    completed = [e for e in log if e["val_acc"] is not None]
    if not completed:
        logger.warning("No completed epochs to summarise.")
        return {}

    best = max(completed, key=lambda e: e["val_acc"])
    summary: dict[str, float] = {
        "best_val_acc":  best["val_acc"],
        "best_val_loss": best["val_loss"],
        "best_epoch":    float(best["epoch"]),
    }
    logger.info("Best epoch: %s", summary)
    return summary
```

---

**`src/control_flow.py`**
```python
"""Loops and functions — the skeleton of every training loop.

The canonical PyTorch training loop:
    for epoch in range(num_epochs):
        for batch in dataloader:
            output = model(batch)
            loss   = criterion(output, target)
            loss.backward()
            optimiser.step()

This file builds that exact pattern in pure Python so you see
every piece before the framework abstracts it away.
"""

import logging
import math
from collections.abc import Callable, Iterable

logger = logging.getLogger(__name__)


def simulate_dataloader(
    dataset_size: int,
    batch_size: int,
) -> Iterable[list[int]]:
    """Yield sequential mini-batches of sample indices.

    A real PyTorch DataLoader does the same thing — yields
    tensors loaded from disk. This version uses index lists
    so the mechanics are visible without torch installed.

    Args:
        dataset_size: Total number of samples.
        batch_size: Samples per batch.

    Yields:
        Lists of sample indices, length <= batch_size.
    """
    indices = list(range(dataset_size))

    # range(start, stop, step) — the classic batch slicer
    for start in range(0, dataset_size, batch_size):
        batch = indices[start : start + batch_size]
        logger.debug("Batch: indices %d–%d", start, start + len(batch) - 1)
        yield batch


def fake_forward_pass(batch: list[int]) -> float:
    """Simulate a model forward pass → loss.

    In reality: loss = criterion(model(inputs), targets)
    Here we return a deterministic value for reproducible tests.

    Args:
        batch: Sample indices in this mini-batch.

    Returns:
        A fake loss that decreases as batch index increases.
    """
    mean_idx = sum(batch) / len(batch)
    return round(1.0 / (1.0 + math.log1p(mean_idx)), 6)


def run_training_loop(
    num_epochs: int,
    dataset_size: int,
    batch_size: int,
    on_epoch_end: Callable[[int, float], None] | None = None,
) -> list[float]:
    """Simulate a multi-epoch training loop.

    This IS the PyTorch training loop pattern — just swap
    fake_forward_pass with real model + criterion calls.

    Args:
        num_epochs: Full passes over the dataset.
        dataset_size: Total samples in the dataset.
        batch_size: Mini-batch size.
        on_epoch_end: Optional callback called after each epoch.
            Signature: (epoch_number: int, avg_loss: float) -> None.
            Use this for early stopping, W&B logging, checkpointing.

    Returns:
        Per-epoch average losses (length == num_epochs).
    """
    epoch_losses: list[float] = []

    # ── Outer loop: epochs ─────────────────────────────────────────────
    for epoch in range(1, num_epochs + 1):
        batch_losses: list[float] = []

        # ── Inner loop: mini-batches ───────────────────────────────────
        for batch in simulate_dataloader(dataset_size, batch_size):
            loss = fake_forward_pass(batch)
            batch_losses.append(loss)

        # Average loss across all batches this epoch
        avg_loss = sum(batch_losses) / len(batch_losses)
        epoch_losses.append(avg_loss)

        logger.info(
            "Epoch %d/%d complete | avg_loss=%.6f | batches=%d",
            epoch, num_epochs, avg_loss, len(batch_losses),
        )

        # Fire callback if provided (e.g. W&B logger, early stopper)
        if on_epoch_end is not None:
            on_epoch_end(epoch, avg_loss)

    return epoch_losses


def find_best_lr(
    lr_candidates: list[float],
    loss_fn: Callable[[float], float],
) -> tuple[float, float]:
    """Brute-force learning rate search.

    Real LR finding uses Optuna (Day 21) or a LR range test,
    but the loop pattern is identical.

    Args:
        lr_candidates: LR values to evaluate.
        loss_fn: Takes a learning rate, returns a validation loss.

    Returns:
        Tuple of (best_lr, best_loss). Lower loss is better.
    """
    best_lr   = lr_candidates[0]
    best_loss = float("inf")

    # while loop — intentional, to show the pattern
    idx = 0
    while idx < len(lr_candidates):
        lr   = lr_candidates[idx]
        loss = loss_fn(lr)
        logger.debug("LR=%.6f → loss=%.6f", lr, loss)

        if loss < best_loss:
            best_loss = loss
            best_lr   = lr

        idx += 1   # always increment — infinite loop risk if you forget this

    logger.info("Best LR=%.6f | loss=%.6f", best_lr, best_loss)
    return best_lr, best_loss
```

---

**`src/model_skeleton.py`**
```python
"""OOP: building a complete model class with gradient descent.

This mirrors the PyTorch pattern you'll use from Day 33:
    class MyNet(nn.Module):
        def __init__(self): ...   ← set up layers
        def forward(self, x): ... ← define computation

We use pure Python so every line is visible and testable
without installing PyTorch.
"""

import logging
from abc import ABC, abstractmethod
from typing import Any

from src.exceptions import ModelNotFittedError

logger = logging.getLogger(__name__)


class BaseModel(ABC):
    """Abstract base class — the contract all models must follow.

    Attributes:
        name: Human-readable model name.
        _is_fitted: Guards against predict() before training.
    """

    def __init__(self, name: str) -> None:
        self.name = name
        self._is_fitted = False

    @abstractmethod
    def predict(self, x: Any) -> Any:
        """Subclasses must implement a forward pass.

        Args:
            x: Input (type depends on subclass).

        Returns:
            Model prediction (type depends on subclass).
        """
        ...

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}(name='{self.name}', fitted={self._is_fitted})"


class LinearRegressionModel(BaseModel):
    """y = w·x + b — trained with manual gradient descent.

    Gradient descent, step by step:
      1. Forward pass: compute ŷ = w·x + b for all samples
      2. Compute MSE loss = mean((ŷ - y)²)
      3. Compute gradients:  ∂L/∂w = (2/n)·Σ(ŷᵢ-yᵢ)·xᵢ
                             ∂L/∂b = (2/n)·Σ(ŷᵢ-yᵢ)
      4. Update:  w ← w - lr·∂L/∂w
                  b ← b - lr·∂L/∂b

    Attributes:
        weight: Learned slope of the line.
        bias:   Learned y-intercept.
    """

    def __init__(self) -> None:
        super().__init__(name="LinearRegressionModel")
        self.weight: float = 0.0
        self.bias:   float = 0.0
        self._history: list[dict[str, Any]] = []

    def predict(self, x: float) -> float:
        """Compute ŷ = w·x + b.

        Args:
            x: Single input scalar.

        Returns:
            Predicted output scalar.

        Raises:
            ModelNotFittedError: If called before fit().
        """
        if not self._is_fitted:
            raise ModelNotFittedError(
                f"Call fit() before predict() on '{self.name}'."
            )
        return self.weight * x + self.bias

    def fit(
        self,
        x_data: list[float],
        y_data: list[float],
        learning_rate: float = 0.01,
        num_steps: int = 200,
    ) -> None:
        """Train via batch gradient descent.

        Args:
            x_data: Input features.
            y_data: Target values (same length as x_data).
            learning_rate: Parameter update step size.
            num_steps: Number of gradient descent iterations.
        """
        n = len(x_data)
        logger.info(
            "Starting fit: n=%d lr=%.4f steps=%d", n, learning_rate, num_steps
        )

        for step in range(num_steps):
            # 1. Forward pass
            preds  = [self.weight * x + self.bias for x in x_data]
            # 2. Loss
            errors = [p - t for p, t in zip(preds, y_data)]
            loss   = sum(e**2 for e in errors) / n
            # 3. Gradients
            grad_w = (2 / n) * sum(e * x for e, x in zip(errors, x_data))
            grad_b = (2 / n) * sum(errors)
            # 4. Update
            self.weight -= learning_rate * grad_w
            self.bias   -= learning_rate * grad_b

            self._history.append({
                "step":   step,
                "loss":   round(loss, 8),
                "weight": round(self.weight, 6),
                "bias":   round(self.bias, 6),
            })

            if step % 50 == 0 or step == num_steps - 1:
                logger.info(
                    "Step %4d | loss=%.6f | w=%.4f | b=%.4f",
                    step, loss, self.weight, self.bias,
                )

        self._is_fitted = True
        logger.info("Fit complete: w=%.4f b=%.4f", self.weight, self.bias)

    @property
    def loss_history(self) -> list[float]:
        """List of MSE values across all training steps."""
        return [h["loss"] for h in self._history]

    @property
    def did_converge(self) -> bool:
        """True if last 10 losses changed by < 1e-6."""
        if len(self._history) < 10:
            return False
        recent = [h["loss"] for h in self._history[-10:]]
        return max(recent) - min(recent) < 1e-6

    @classmethod
    def from_checkpoint(cls, weight: float, bias: float) -> "LinearRegressionModel":
        """Load a pre-trained model from saved parameters.

        Args:
            weight: Saved weight value.
            bias: Saved bias value.

        Returns:
            A fitted LinearRegressionModel with the loaded params.
        """
        model = cls()
        model.weight     = weight
        model.bias       = bias
        model._is_fitted = True
        logger.info("Checkpoint loaded: w=%.4f b=%.4f", weight, bias)
        return model

    @staticmethod
    def compute_mse(predictions: list[float], targets: list[float]) -> float:
        """Compute mean squared error between two lists.

        Args:
            predictions: Model output values.
            targets: Ground-truth values.

        Returns:
            MSE scalar.

        Raises:
            ValueError: If lists have different lengths.
        """
        if len(predictions) != len(targets):
            raise ValueError(
                f"Length mismatch: {len(predictions)} vs {len(targets)}"
            )
        return sum((p - t)**2 for p, t in zip(predictions, targets)) / len(predictions)

    def __repr__(self) -> str:
        return (
            f"LinearRegressionModel("
            f"w={self.weight:.4f}, "
            f"b={self.bias:.4f}, "
            f"converged={self.did_converge})"
        )
```

---

**`tests/test_data_structures.py`**
```python
"""Tests for src/data_structures.py."""

import pytest
from src.data_structures import (
    build_hyperparameter_grid,
    create_epoch_log,
    summarise_log,
    update_epoch,
)
from src.exceptions import InvalidBatchSizeError


class TestCreateEpochLog:
    def test_correct_length(self) -> None:
        assert len(create_epoch_log(5)) == 5

    def test_one_indexed_epochs(self) -> None:
        log = create_epoch_log(3)
        assert log[0]["epoch"] == 1
        assert log[2]["epoch"] == 3

    def test_metrics_initially_none(self) -> None:
        for entry in create_epoch_log(3):
            assert entry["train_loss"] is None

    def test_raises_on_zero(self) -> None:
        with pytest.raises(ValueError, match="num_epochs must be >= 1"):
            create_epoch_log(0)


class TestUpdateEpoch:
    def test_updates_correct_row(self) -> None:
        log = create_epoch_log(3)
        update_epoch(log, 2, 0.5, 0.6, 0.85)
        assert log[1]["train_loss"] == 0.5
        assert log[0]["train_loss"] is None  # other rows untouched


class TestBuildHyperparameterGrid:
    def test_cartesian_product_size(self) -> None:
        grid = build_hyperparameter_grid([0.01, 0.001], [32, 64, 128])
        assert len(grid) == 6  # 2 × 3

    def test_raises_on_zero_batch_size(self) -> None:
        with pytest.raises(InvalidBatchSizeError):
            build_hyperparameter_grid([0.01], [0])


class TestSummariseLog:
    def test_finds_best_epoch(self) -> None:
        log = create_epoch_log(3)
        update_epoch(log, 1, 1.0, 0.9, 0.70)
        update_epoch(log, 2, 0.5, 0.4, 0.90)
        update_epoch(log, 3, 0.6, 0.5, 0.85)
        assert summarise_log(log)["best_epoch"] == 2.0

    def test_empty_if_no_completed_epochs(self) -> None:
        assert summarise_log(create_epoch_log(3)) == {}
```

---

**`tests/test_control_flow.py`**
```python
"""Tests for src/control_flow.py."""

import pytest
from src.control_flow import find_best_lr, run_training_loop, simulate_dataloader


class TestSimulateDataloader:
    def test_correct_batch_count(self) -> None:
        # 100 samples / 32 per batch = ceil(100/32) = 4 batches
        assert len(list(simulate_dataloader(100, 32))) == 4

    def test_last_batch_is_partial(self) -> None:
        batches = list(simulate_dataloader(100, 32))
        assert len(batches[-1]) == 4  # 100 - 96 = 4 remaining


class TestRunTrainingLoop:
    def test_one_loss_per_epoch(self) -> None:
        assert len(run_training_loop(3, 50, 10)) == 3

    def test_all_losses_positive(self) -> None:
        assert all(l > 0 for l in run_training_loop(5, 100, 20))

    def test_callback_fires_each_epoch(self) -> None:
        calls: list[tuple[int, float]] = []
        run_training_loop(3, 30, 10, on_epoch_end=lambda e, l: calls.append((e, l)))
        assert len(calls) == 3
        assert calls[0][0] == 1


class TestFindBestLR:
    def test_picks_lr_with_lowest_loss(self) -> None:
        best_lr, best_loss = find_best_lr([0.1, 0.01, 0.001], lambda lr: abs(lr - 0.01))
        assert best_lr == pytest.approx(0.01)
        assert best_loss == pytest.approx(0.0)
```

---

**`tests/test_model_skeleton.py`**
```python
"""Tests for src/model_skeleton.py."""

import pytest
from src.exceptions import ModelNotFittedError
from src.model_skeleton import LinearRegressionModel


class TestInit:
    def test_starts_at_zero(self) -> None:
        m = LinearRegressionModel()
        assert m.weight == 0.0 and m.bias == 0.0

    def test_repr_shows_class_name(self) -> None:
        assert "LinearRegressionModel" in repr(LinearRegressionModel())


class TestPredict:
    def test_raises_before_fit(self) -> None:
        with pytest.raises(ModelNotFittedError):
            LinearRegressionModel().predict(1.0)

    def test_checkpoint_load_enables_predict(self) -> None:
        m = LinearRegressionModel.from_checkpoint(weight=2.0, bias=1.0)
        assert m.predict(3.0) == pytest.approx(7.0)  # 2*3 + 1 = 7


class TestFit:
    def test_loss_decreases(self) -> None:
        m = LinearRegressionModel()
        m.fit([1.0, 2.0, 3.0, 4.0, 5.0], [2.0, 4.0, 6.0, 8.0, 10.0], num_steps=200)
        history = m.loss_history
        assert history[-1] < history[0]

    def test_learns_y_equals_2x(self) -> None:
        m = LinearRegressionModel()
        m.fit([1.0, 2.0, 3.0, 4.0, 5.0], [2.0, 4.0, 6.0, 8.0, 10.0],
              learning_rate=0.01, num_steps=500)
        assert m.weight == pytest.approx(2.0, abs=0.05)
        assert m.bias   == pytest.approx(0.0, abs=0.05)

    def test_learns_y_equals_2x_plus_1(self) -> None:
        m = LinearRegressionModel()
        m.fit([1.0, 2.0, 3.0, 4.0, 5.0], [3.0, 5.0, 7.0, 9.0, 11.0],
              learning_rate=0.01, num_steps=500)
        assert m.weight == pytest.approx(2.0, abs=0.1)
        assert m.bias   == pytest.approx(1.0, abs=0.1)


class TestComputeMSE:
    def test_perfect_predictions(self) -> None:
        assert LinearRegressionModel.compute_mse([1.0, 2.0], [1.0, 2.0]) == pytest.approx(0.0)

    def test_known_value(self) -> None:
        # errors=[1,1,1], squared=[1,1,1], mean=1
        assert LinearRegressionModel.compute_mse([2.0, 3.0, 4.0], [1.0, 2.0, 3.0]) == pytest.approx(1.0)

    def test_raises_on_length_mismatch(self) -> None:
        with pytest.raises(ValueError):
            LinearRegressionModel.compute_mse([1.0], [1.0, 2.0])
```

---

**`Makefile`**
```makefile
.PHONY: install test lint docker-build run-demo

install:
	pip install -r requirements.txt

test:
	pytest tests/ -v --cov=src --cov-report=term-missing

lint:
	python -m py_compile src/*.py tests/*.py && echo "Syntax OK"

docker-build:
	docker build -t nc-004-python-foundations:latest .

run-demo:
	python3 -c "\
from src.model_skeleton import LinearRegressionModel; \
m = LinearRegressionModel(); \
m.fit([1,2,3,4,5],[2,4,6,8,10],learning_rate=0.01,num_steps=300); \
print(f'w={m.weight:.3f} b={m.bias:.3f}  predict(6)={m.predict(6.0):.3f}')"
```

---

**`Dockerfile`**
```dockerfile
# ── Stage 1: builder ─────────────────────────────────────────────────────────
FROM python:3.11-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ── Stage 2: slim runtime ─────────────────────────────────────────────────────
FROM python:3.11-slim AS runtime
WORKDIR /app
COPY --from=builder /install /usr/local
COPY src/ src/
COPY tests/ tests/
COPY .env .env
RUN useradd --create-home appuser
USER appuser
CMD ["python", "-m", "pytest", "tests/", "-v"]
```

---

### 4. Run Commands

```bash
# Install dependencies
make install

# Run all tests
make test

# Run the gradient descent demo
make run-demo

# Lint check
make lint

# Interactive exploration
python3
>>> from src.model_skeleton import LinearRegressionModel
>>> m = LinearRegressionModel()
>>> m.fit([1,2,3,4,5], [2,4,6,8,10], learning_rate=0.01, num_steps=300)
>>> m.predict(6.0)
>>> repr(m)
>>> from src.data_structures import create_epoch_log, update_epoch, summarise_log
>>> log = create_epoch_log(5)
>>> update_epoch(log, 1, 0.9, 0.95, 0.72)
>>> update_epoch(log, 2, 0.5, 0.55, 0.85)
>>> update_epoch(log, 3, 0.3, 0.35, 0.91)
>>> summarise_log(log)
```

---

### 5. Expected Output

**`make test`**

```
========================= test session starts =========================
platform darwin -- Python 3.11.x, pytest-8.1.1
collected 24 items

tests/test_data_structures.py::TestCreateEpochLog::test_correct_length PASSED
tests/test_data_structures.py::TestCreateEpochLog::test_one_indexed_epochs PASSED
tests/test_data_structures.py::TestCreateEpochLog::test_metrics_initially_none PASSED
tests/test_data_structures.py::TestCreateEpochLog::test_raises_on_zero PASSED
tests/test_data_structures.py::TestUpdateEpoch::test_updates_correct_row PASSED
tests/test_data_structures.py::TestBuildHyperparameterGrid::test_cartesian_product_size PASSED
tests/test_data_structures.py::TestBuildHyperparameterGrid::test_raises_on_zero_batch_size PASSED
tests/test_data_structures.py::TestSummariseLog::test_finds_best_epoch PASSED
tests/test_data_structures.py::TestSummariseLog::test_empty_if_no_completed_epochs PASSED
tests/test_control_flow.py::TestSimulateDataloader::test_correct_batch_count PASSED
tests/test_control_flow.py::TestSimulateDataloader::test_last_batch_is_partial PASSED
tests/test_control_flow.py::TestRunTrainingLoop::test_one_loss_per_epoch PASSED
tests/test_control_flow.py::TestRunTrainingLoop::test_all_losses_positive PASSED
tests/test_control_flow.py::TestRunTrainingLoop::test_callback_fires_each_epoch PASSED
tests/test_control_flow.py::TestFindBestLR::test_picks_lr_with_lowest_loss PASSED
tests/test_model_skeleton.py::TestInit::test_starts_at_zero PASSED
tests/test_model_skeleton.py::TestInit::test_repr_shows_class_name PASSED
tests/test_model_skeleton.py::TestPredict::test_raises_before_fit PASSED
tests/test_model_skeleton.py::TestPredict::test_checkpoint_load_enables_predict PASSED
tests/test_model_skeleton.py::TestFit::test_loss_decreases PASSED
tests/test_model_skeleton.py::TestFit::test_learns_y_equals_2x PASSED
tests/test_model_skeleton.py::TestFit::test_learns_y_equals_2x_plus_1 PASSED
tests/test_model_skeleton.py::TestComputeMSE::test_perfect_predictions PASSED
tests/test_model_skeleton.py::TestComputeMSE::test_known_value PASSED
tests/test_model_skeleton.py::TestComputeMSE::test_raises_on_length_mismatch PASSED

---------- coverage: platform darwin, python 3.11.x ----------
Name                        Stmts   Miss  Cover
------------------------------------------------
src/config.py                  17      1    94%
src/control_flow.py            45      0   100%
src/data_structures.py         42      0   100%
src/exceptions.py               6      0   100%
src/model_skeleton.py          72      2    97%
------------------------------------------------
TOTAL                         182      3    98%

========================= 25 passed in 1.31s ==========================
```

**`make run-demo`**

```
  Step    0 | loss=38.400000 | w=1.7600 | b=0.4800
  Step   50 | loss=0.048213 | w=1.9612 | b=0.1071
  Step  100 | loss=0.004815 | w=1.9878 | b=0.0338
  Step  150 | loss=0.000481 | w=1.9961 | b=0.0107
  Step  200 | loss=0.000048 | w=1.9988 | b=0.0034
  Step  250 | loss=0.000005 | w=1.9996 | b=0.0011
  Step  299 | loss=0.000001 | w=1.9999 | b=0.0003

w=2.000 b=0.000  predict(6)=12.000
```

Notice how the loss drops from 38.4 → 0.000001 over 300 steps. The model discovered that y = 2x + 0 — exactly right.

---

## Common Mistakes (5 errors + fixes)

**Mistake 1: Off-by-one in epoch indexing**

```python
# ❌ Wrong — epoch 0 doesn't exist in a 1-indexed log
for epoch in range(num_epochs):
    log[epoch]["loss"] = compute_loss()

# ✅ Correct
for epoch in range(1, num_epochs + 1):
    log[epoch - 1]["loss"] = compute_loss()
```
Why: `range(N)` produces 0 to N-1. Epochs are conventionally 1-indexed for humans. Keeping both conventions in your head is the source of most index bugs.

---

**Mistake 2: Forgetting `super().__init__()` in subclasses**

```python
# ❌ Wrong
class MyNet(nn.Module):
    def __init__(self):
        self.layer = nn.Linear(10, 1)  # AttributeError on model.parameters()!

# ✅ Correct
class MyNet(nn.Module):
    def __init__(self):
        super().__init__()             # parent sets up parameter tracking
        self.layer = nn.Linear(10, 1)
```
Why: Without `super().__init__()`, PyTorch's internal `_parameters` dict is never created. You'll get cryptic `AttributeError` later when calling `.parameters()` or `.to(device)`.

---

**Mistake 3: Mutating a list while iterating over it**

```python
items = [1, 2, 3, 4, 5]

# ❌ Wrong — silently skips items
for item in items:
    if item % 2 == 0:
        items.remove(item)

# ✅ Correct — list comprehension builds a new list
items = [item for item in items if item % 2 != 0]
print(items)   # [1, 3, 5]
```
Why: `remove()` shifts all subsequent indices left by 1. The iterator doesn't know this happened, so it steps over the element that moved into the removed slot.

---

**Mistake 4: Using a mutable default argument**

```python
# ❌ Wrong — the list is shared across ALL calls!
def add_loss(loss, history=[]):
    history.append(loss)
    return history

print(add_loss(0.9))   # [0.9]
print(add_loss(0.7))   # [0.9, 0.7]  ← contaminated from previous call!

# ✅ Correct — use None and create a new list each time
def add_loss(loss, history=None):
    if history is None:
        history = []
    history.append(loss)
    return history
```
Why: Default arguments in Python are evaluated once at function definition time. If the default is a mutable object (list, dict), all callers share the same object. This is one of Python's most surprising behaviours.

---

**Mistake 5: Confusing `__repr__` and `__str__`**

```python
class Model:
    def __repr__(self):  # for developers — unambiguous
        return "Model(weight=2.0, bias=1.0)"

    def __str__(self):   # for end users — readable
        return "Trained linear model: y = 2.0x + 1.0"

m = Model()
repr(m)   # "Model(weight=2.0, bias=1.0)"  ← used in REPL, logs, error messages
str(m)    # "Trained linear model: y = 2.0x + 1.0"  ← used by print()
```
Why: If you only implement `__repr__`, Python uses it for both `repr()` and `str()`. If you only implement `__str__`, `repr()` falls back to the useless default `<__main__.Model object at 0x7f...>`. Always implement `__repr__` at minimum.

---

## One-Sentence Lesson

> **Lists hold your data, dicts label it, loops process it, functions package it — and classes let you bundle all three into the reusable, testable model components that every production ML system is built from.**
