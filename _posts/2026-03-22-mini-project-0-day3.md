---
layout: single
title: "Day 3 of 180 - ML Project Structure + NeuralCorp Scaffold (Mini-Project MP-0)"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## What Is This?

Day 3 is arguably the most important day of Phase 0 - more important than Python or Git individually. Today I built the **NeuralCorp project scaffold**: a production-standard template that every single project for the next 177 days is cloned from. Every real ML team has a scaffold like this. It enforces consistent structure, automates quality checks, and ensures that code written on Day 3 looks identical in discipline to code written on Day 180.

This is also **Mini-Project MP-0** - the first of 12 mini-projects in the curriculum.

---

## The Analogy

Think of the scaffold like the steel frame of a building before the walls go up. Every serious construction project starts with the same basic frame: foundation, beams, supports. You don't improvise the frame differently on each building - the frame is standard because it works, it's safe, and everyone on the team knows how to work with it.

Every data scientist who "just wants to explore quickly" in a Jupyter notebook has at some point caused a production incident because their exploration code made it to production without a frame. The scaffold prevents that.

---

## The Concepts Explained Simply

### Why Project Structure Matters

Without a standard structure, every project looks different. One project has `model.py`, another has `train_model.py`, another has `ModelClass.py`. A new team member (or future-you in 6 months) wastes hours just figuring out where things are.

With a standard structure, the answer is always the same:
- Config? → `src/config.py`
- Business logic? → `src/[module].py`
- Tests? → `tests/test_[module].py`
- How to run? → `make run`
- How to test? → `make test`

### The NeuralCorp Standard Project Layout

```
neuralcorp-project/
├── src/
│   ├── __init__.py        ← makes src/ a Python package
│   ├── config.py          ← ALL config loaded from .env (Pydantic)
│   ├── exceptions.py      ← custom exception classes
│   └── [module].py        ← your main logic goes here
├── tests/
│   ├── __init__.py
│   └── test_[module].py   ← pytest test file
├── notebooks/             ← exploration only, NEVER production code
├── models/                ← saved model artifacts (.pt, .pkl)
├── data/
│   ├── raw/               ← original data, never modified
│   └── processed/         ← cleaned/transformed data
├── logs/                  ← runtime log files
├── .env                   ← real secrets - NOT in Git
├── .env.example           ← template showing required vars - IS in Git
├── .gitignore             ← what Git ignores
├── Dockerfile             ← multi-stage container definition
├── Makefile               ← shortcut commands (make test, make run)
├── pyproject.toml         ← linting and formatting config
├── requirements.txt       ← pinned package versions
└── README.md              ← how to set up and run the project
```

### Pydantic BaseSettings - Config Done Right

The most common beginner mistake: hardcoding config values directly in Python files.

```python
# ❌ Wrong - hardcoded, insecure, breaks in different environments
API_KEY = "sk-abc123..."
MODEL_PATH = "/Users/edward/models/model.pt"
```

The production way is Pydantic `BaseSettings` - it reads from a `.env` file automatically:

```python
# ✅ Right - reads from .env, typed, validated
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_key: str           # must exist in .env or environment
    model_path: str = "models/model.pt"   # has a default
    batch_size: int = 32

settings = Settings()     # auto-reads .env
```

Your `.env` file (never committed to Git):
```
API_KEY=sk-abc123...
MODEL_PATH=models/my_model.pt
```

Your `.env.example` (committed to Git - shows teammates what vars are needed):
```
API_KEY=your-key-here
MODEL_PATH=models/model.pt
```

### Custom Exceptions - Self-Documenting Errors

Never raise a bare `Exception("something went wrong")`. That tells you nothing. Custom exceptions make errors self-documenting:

```python
# ❌ Wrong
raise Exception("model file not found")

# ✅ Right
raise ModelNotFoundError(f"No model file at: {path}")
```

When you see `ModelNotFoundError` in a stack trace, you know exactly what happened and where to look.

### Makefile - One-Word Commands for Everything

A `Makefile` is a file of named shortcuts. Instead of:
```bash
pytest tests/ -v --cov=src --cov-report=term-missing --cov-fail-under=80
```

You type:
```bash
make test
```

Every teammate and every CI/CD pipeline uses the same commands. `make install`, `make test`, `make lint`, `make docker-build`, `make run` - these are standard across every NeuralCorp project.

### Multi-Stage Dockerfile - Small, Secure, Production-Ready

A single-stage Dockerfile is easy to write but produces huge images (often 2–4GB) that are slow to push and have a large attack surface.

A multi-stage Dockerfile has two stages:
1. **Builder**: installs all dependencies (can be large)
2. **Runtime**: copies only the installed packages into a minimal image (usually 200–400MB)

```dockerfile
FROM python:3.11-slim AS builder    # Stage 1: heavy, install everything
RUN pip install --prefix=/install -r requirements.txt

FROM python:3.11-slim AS runtime    # Stage 2: light, copy only what's needed
COPY --from=builder /install /usr/local
COPY src/ ./src/
USER appuser                         # Never run as root in production
```

### Code Quality Tools - Automated Discipline

| Tool | What It Does | When It Runs |
|---|---|---|
| `black` | Auto-formats your code to a consistent style | `make format` |
| `ruff` | Lints for errors, bad patterns, unused imports | `make lint` |
| `mypy` | Checks type hints are correct | `make lint` |
| `pytest` | Runs your test suite | `make test` |
| `pre-commit` | Runs all checks automatically before every `git commit` | On every commit |

---

## How Real Companies Use This

- **Every ML team at a serious company** (Stripe, Airbnb, Cohere) uses a project scaffold like this. New engineers clone the template, not a blank folder.
- **Code reviews at Google** will reject code that lacks type hints, docstrings, or tests - these aren't optional extras, they're table stakes.
- **The Makefile pattern** is how GitHub Actions CI/CD pipelines work: `run: make test && make docker-build`. If `make test` fails, the build stops.
- **Multi-stage Dockerfiles** are a production requirement - single-stage images routinely fail security scans at companies with security review gates.

---

## Step-by-Step: Build Your NeuralCorp Scaffold (Mini-Project MP-0)

### 1. Create the scaffold structure

```bash
# Create and enter the project folder
mkdir -p neuralcorp-scaffold && cd neuralcorp-scaffold

# Create all directories
mkdir -p src tests notebooks models data/raw data/processed logs

# Create placeholder files (git needs at least one file to track a folder)
touch src/__init__.py tests/__init__.py logs/.gitkeep

# Set up virtual environment
python3 -m venv .venv && source .venv/bin/activate
pip install --upgrade pip

# Install all dev tools with exact versions
pip install \
    pydantic==2.7.1 \
    pydantic-settings==2.3.0 \
    black==24.4.2 \
    ruff==0.4.4 \
    mypy==1.10.0 \
    pytest==8.2.0 \
    pytest-cov==5.0.0 \
    pre-commit==3.7.1

# Save pinned versions
pip freeze > requirements.txt
```

### 2. Create `src/config.py`

```python
"""config.py - Application configuration via Pydantic BaseSettings.

All configuration is loaded from environment variables / .env file.
Never hardcode values anywhere else in the codebase.
"""
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    """Project settings loaded from the .env file at runtime.

    Pydantic BaseSettings reads environment variables automatically.
    Type annotations enforce that values are the right type.
    Default values are used when the env var is not set.
    """

    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    # Project identity
    project_name: str = "NeuralCorp Project"
    environment: str = "development"    # development | staging | production
    log_level: str = "INFO"

    # ML config (override per project)
    model_path: str = "models/model.pkl"
    batch_size: int = 32
    random_seed: int = 42
    learning_rate: float = 1e-3


# Single global instance - import this everywhere: from src.config import settings
settings = Settings()
```

### 3. Create `src/exceptions.py`

```python
"""exceptions.py - Custom exception hierarchy for NeuralCorp projects.

Using named exceptions makes errors self-documenting.
Callers can catch exactly the exception they expect without
accidentally swallowing unrelated errors.
"""


class NeuralCorpBaseError(Exception):
    """Base exception - all NeuralCorp errors inherit from this."""


class DataValidationError(NeuralCorpBaseError):
    """Raised when input data fails schema or value validation."""


class ModelNotFoundError(NeuralCorpBaseError):
    """Raised when a model file does not exist at the expected path."""


class InferenceError(NeuralCorpBaseError):
    """Raised when model inference fails at runtime."""


class ConfigurationError(NeuralCorpBaseError):
    """Raised when required configuration is missing or invalid."""
```

### 4. Create `tests/test_config.py`

```python
"""test_config.py - Tests for configuration loading."""
import pytest
from src.config import Settings
from src.exceptions import NeuralCorpBaseError, DataValidationError


class TestSettings:
    """Tests for the Settings class."""

    def test_default_values(self) -> None:
        """Settings loads with correct defaults when no .env is present."""
        s = Settings()
        assert s.environment == "development"
        assert s.batch_size == 32
        assert s.random_seed == 42
        assert s.log_level == "INFO"

    def test_batch_size_is_int(self) -> None:
        """batch_size must be an integer."""
        s = Settings()
        assert isinstance(s.batch_size, int)

    def test_learning_rate_is_float(self) -> None:
        """learning_rate must be a float."""
        s = Settings()
        assert isinstance(s.learning_rate, float)


class TestExceptions:
    """Tests for the custom exception hierarchy."""

    def test_data_validation_error_is_base_error(self) -> None:
        """DataValidationError should be catchable as NeuralCorpBaseError."""
        with pytest.raises(NeuralCorpBaseError):
            raise DataValidationError("bad data")

    def test_exception_message_preserved(self) -> None:
        """Exception message should be retrievable."""
        msg = "model.pkl not found at models/model.pkl"
        try:
            from src.exceptions import ModelNotFoundError
            raise ModelNotFoundError(msg)
        except NeuralCorpBaseError as exc:
            assert str(exc) == msg
```

### 5. Create `pyproject.toml`

```toml
[tool.black]
line-length = 88
target-version = ["py311"]

[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "W", "UP"]
ignore = ["E501"]

[tool.mypy]
python_version = "3.11"
strict = true
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=80"
```

### 6. Create `Makefile`

```makefile
.PHONY: install test lint format docker-build run clean

install:          ## Install all dependencies
	pip install -r requirements.txt

test:             ## Run tests with coverage (must be ≥80%)
	pytest tests/ -v

lint:             ## Run ruff + mypy type checks
	ruff check src/ tests/
	mypy src/

format:           ## Auto-format code with black
	black src/ tests/

docker-build:     ## Build the Docker image
	docker build -t neuralcorp-scaffold:latest .

run:              ## Start the application
	uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload

clean:            ## Remove all build artefacts and caches
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -name "*.pyc" -delete
	rm -rf .pytest_cache .mypy_cache .ruff_cache htmlcov
```

### 7. Create `Dockerfile`

```dockerfile
# ── Stage 1: Builder ─────────────────────────────────────────────────────────
# Uses a full image to install dependencies efficiently
FROM python:3.11-slim AS builder
WORKDIR /app

# Copy only requirements first - Docker caches this layer if requirements.txt
# doesn't change, so rebuilds are fast when you only change src/ code
COPY requirements.txt .
RUN pip install --prefix=/install --no-cache-dir -r requirements.txt

# ── Stage 2: Runtime ─────────────────────────────────────────────────────────
# Uses the same slim base - final image is much smaller than a single-stage build
FROM python:3.11-slim AS runtime
WORKDIR /app

# Copy only the installed packages from builder - not the full pip cache
COPY --from=builder /install /usr/local

# Copy application source code
COPY src/ ./src/

# Security: never run as root in production containers
RUN useradd -m appuser && chown -R appuser /app
USER appuser

EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 8. Create `.env.example` and `.env`

```bash
# .env.example (committed to Git - shows what vars are needed)
cat > .env.example << 'EOF'
PROJECT_NAME=NeuralCorp Project
ENVIRONMENT=development
LOG_LEVEL=INFO
MODEL_PATH=models/model.pkl
BATCH_SIZE=32
RANDOM_SEED=42
LEARNING_RATE=0.001
EOF

# .env (NOT committed - your real values)
cp .env.example .env
```

### 9. Create `.gitignore`

```bash
cat > .gitignore << 'EOF'
# Python
.venv/
__pycache__/
*.pyc
*.pyo
*.egg-info/

# Environment
.env

# ML artifacts
models/*.pkl
models/*.pt
models/*.onnx
data/raw/
data/processed/

# Logs
logs/*.log

# Build / test caches
.pytest_cache/
.mypy_cache/
.ruff_cache/
htmlcov/
.coverage

# IDE
.vscode/settings.json
.idea/

# macOS
.DS_Store
EOF
```

### 10. Verify everything works

```bash
# Run linter
make lint
# Expected: no errors

# Run formatter (auto-fixes style issues)
make format

# Run tests
make test
# Expected:
# tests/test_config.py::TestSettings::test_default_values PASSED
# tests/test_config.py::TestSettings::test_batch_size_is_int PASSED
# ...
# Coverage: 85%+ ✅

# Build Docker image
make docker-build
# Expected: Successfully built ...

# Confirm image exists
docker images | grep neuralcorp-scaffold
```

### Definition of Done - MP-0 Checklist

- [ ] `make lint` passes with zero errors
- [ ] `make test` runs - all tests PASSED, coverage ≥80%
- [ ] `make docker-build` produces a working image
- [ ] `src/config.py` uses Pydantic `BaseSettings` - zero hardcoded values
- [ ] `src/exceptions.py` has ≥4 custom exception classes
- [ ] `.env` exists but is in `.gitignore` - not committed
- [ ] `.env.example` is committed - shows all required vars
- [ ] `requirements.txt` has pinned versions (`pip freeze > requirements.txt`)
- [ ] Multi-stage `Dockerfile` with non-root `USER appuser`
- [ ] `Makefile` has: `install`, `test`, `lint`, `format`, `docker-build`, `clean`

---

## Common Mistakes

**Mistake 1: Using bare `Exception` instead of custom exceptions**
`raise Exception("error")` tells you nothing in a stack trace. `raise ModelNotFoundError("models/model.pt does not exist")` tells you exactly what went wrong and where to look. Write custom exceptions from Day 1.

**Mistake 2: Skipping `make lint` before committing**
If you haven't run `make lint` locally, CI will fail on GitHub Actions and your PR gets blocked. Make it a habit: `make lint && make test` before every `git push`.

**Mistake 3: Single-stage Dockerfile**
A single-stage Dockerfile that does `pip install -r requirements.txt` and then `COPY . .` will produce a 2–4GB image. The multi-stage pattern above produces 200–400MB. On AWS, you pay for every GB stored in ECR and every GB transferred. Multi-stage is not optional at production scale.

**Mistake 4: Hardcoding any value that might change between environments**
Development, staging, and production have different database URLs, API keys, model paths, and batch sizes. If any value is hardcoded in Python, you need a code change (and a deployment) to update it. If it's in `.env`, you just update the env var - no deployment needed.

---

## One-Sentence Lesson

The 20 minutes spent building a proper scaffold on Day 3 saves 20 hours of refactoring later - every professional ML project is built on a template like this, and now so is mine.

---

*Day 3 / 180 complete ✅ - Mini-Project MP-0 shipped 🎉 | Tomorrow: Python recap + OOP basics - the foundation of every PyTorch nn.Module*
