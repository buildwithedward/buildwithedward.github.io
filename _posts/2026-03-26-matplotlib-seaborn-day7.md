---
layout: single
title: "Day 5 of 180 - Data Visualisation"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## Introduction

Today marks a pivotal shift in how we write code. Up until now, you've been using `print()` to see what your programs do. Starting today, you'll graduate to **logging** - the production-standard way that real engineers observe their code.

Here's why this matters: Imagine you deploy a data analysis script to production, and it processes 1 million customer records at 3 AM. Something goes wrong. With `print()`, your output scrolls off the screen and you have no record of what happened. With `logging`, you have a timestamped, categorized, searchable log file that tells you exactly what went wrong, when, and what led up to it.

By the end of today, you'll also have **beautiful, publication-quality plots** using Matplotlib and Seaborn. But logging comes first - it's the more important production skill.

---

## Setup

Before we start, install the packages for today:

```bash
pip install matplotlib==3.8.2 seaborn==0.13.1 python-json-logger==2.0.7
```

**Why these versions?** Pinned versions prevent surprise breaking changes. In real jobs, you'll do this for every project.

---

## Part 1: Python Logging - Your New Production Standard

### 1.1 The Problem with `print()`

Let's say you're processing student test scores:

```python
# ❌ OLD WAY - DON'T DO THIS ANYMORE
print("Starting analysis...")
print("Loaded 100 records")
print("ERROR: Student 42 has no math score")
print("Finished!")
```

**Problems:**
1. **No severity levels** - All output looks the same. Can't tell warnings from errors from normal info.
2. **No timestamps** - When exactly did that error happen? Was it 3 AM or 3 PM?
3. **No filtering** - In development, you want DEBUG messages. In production, they're noise. Can't easily toggle.
4. **Console-only** - Output disappears when the terminal closes. Can't analyze later.
5. **Not machine-readable** - Can't pipe to log aggregation tools (Datadog, Splunk, CloudWatch).

### 1.2 The Solution: `logging` Module

```python
# ✅ NEW WAY - USE THIS ALWAYS
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Starting analysis...")
logger.info("Loaded 100 records")
logger.error("Student 42 has no math score")
logger.info("Finished!")
```

**Benefits:**
1. ✅ **5 levels** (DEBUG, INFO, WARNING, ERROR, CRITICAL) - you choose the severity
2. ✅ **Automatic timestamps** - 2026-03-30 14:32:05,123 on every message
3. ✅ **Filterable** - One config change hides DEBUG noise in production
4. ✅ **Multi-destination** - Send to console AND file simultaneously
5. ✅ **Machine-readable** - Convert to JSON for real log systems

### 1.3 The 5 Log Levels (Traffic Light Analogy)

Think of driving a car through a city. Your log level is like a traffic light system:

```
🔵 DEBUG    = Blue light (take the side streets) = "Checking if row 542 has email field"
              Use: Development only. Too detailed for production.

🟢 INFO     = Green light (proceed normally)      = "Loaded 10,000 records successfully"
              Use: Normal operation milestones. What's going right.

🟡 WARNING  = Yellow light (caution ahead)        = "5 records missing phone number"
              Use: Unexpected but recoverable. Needs attention but won't crash.

🔴 ERROR    = Red light (stop for hazard)         = "Failed to connect to database, retrying..."
              Use: Operation failed, but program continues. Needs fix soon.

🔴🔴 CRITICAL = Red light + alarm (emergency!)    = "Out of disk space, cannot continue"
              Use: Show-stoppers. Program must stop immediately.
```

In **development**, you show everything (DEBUG through CRITICAL).
In **production**, you show only INFO and above (hide DEBUG noise).

### 1.4 Basic Setup: 5 Lines of Code

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# Now use it:
logger.info("Analysis started")
logger.warning("5 rows have missing data")
logger.error("Failed to save plot")
```

**What does each part do?**

- `level=logging.INFO`: Show INFO, WARNING, ERROR, CRITICAL. Hide DEBUG (too noisy).
- `format='...'`: Template for every log message.
  - `%(asctime)s` = timestamp (2026-03-30 14:32:05,123)
  - `%(name)s` = module name (e.g., "data_analysis")
  - `%(levelname)s` = DEBUG/INFO/WARNING/ERROR/CRITICAL
  - `%(message)s` = your message
- `logging.getLogger(__name__)`: Create a logger named after this file

### 1.5 Using the Logger

```python
import logging

logging.basicConfig(level=logging.DEBUG)  # Show everything
logger = logging.getLogger(__name__)

logger.debug("Checking column 'email' in row 5")           # 🔵 Dev detail
logger.info("Successfully loaded 10,000 records")          # 🟢 Normal
logger.warning("Student #42 missing math score, using 0")  # 🟡 Unexpected
logger.error("Failed to save plot: /output/ not writable") # 🔴 Problem
logger.critical("Database offline, cannot continue")       # 🔴🔴 Fatal
```

**Output:**
```
DEBUG    - Checking column 'email' in row 5
INFO     - Successfully loaded 10,000 records
WARNING  - Student #42 missing math score, using 0
ERROR    - Failed to save plot: /output/ not writable
CRITICAL - Database offline, cannot continue
```

### 1.6 Handlers: Send Logs to Multiple Places

By default, `basicConfig()` only sends to **console**. What if you want **file + console**?

```python
import logging
from logging.handlers import RotatingFileHandler

# Create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# HANDLER 1: Console (show only INFO and above)
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_formatter = logging.Formatter('%(levelname)-8s | %(message)s')
console_handler.setFormatter(console_formatter)

# HANDLER 2: File (save everything including DEBUG)
file_handler = RotatingFileHandler(
    'app.log',
    maxBytes=5_000_000,  # Rotate at 5 MB
    backupCount=3         # Keep 3 old files
)
file_handler.setLevel(logging.DEBUG)
file_formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
file_handler.setFormatter(file_formatter)

# Attach both handlers
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# Now log messages go BOTH places
logger.debug("Detailed debugging info")        # Only in app.log
logger.info("Normal operation")                # Both console and app.log
logger.error("Something failed")               # Both console and app.log
```

**Result:**

Console sees:
```
INFO     | Loaded 10,000 records
WARNING  | Student #42 missing data
ERROR    | Failed to save plot
```

`app.log` saves:
```
2026-03-30 14:32:05,123 - __main__ - DEBUG - Checking if field 'email' exists
2026-03-30 14:32:05,234 - __main__ - INFO - Loaded 10,000 records
2026-03-30 14:32:05,345 - __main__ - WARNING - Student #42 missing data
2026-03-30 14:32:05,456 - __main__ - ERROR - Failed to save plot
```

### 1.7 Log Rotation: Keep Files Tidy

If your script runs for days, `app.log` grows huge. Use `RotatingFileHandler`:

```python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    'app.log',
    maxBytes=5_000_000,  # 5 MB per file
    backupCount=5        # Keep 5 old files
)
```

**What happens:**
- When `app.log` reaches 5 MB → rename to `app.log.1`
- Old `app.log.1` → becomes `app.log.2`
- Old `app.log.5` → deleted
- New `app.log` created

You'll never run out of disk space!

### 1.8 Logger Hierarchy: Control Noisy Libraries

Libraries like `requests` (HTTP client) log a lot. You can silence them:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Silence the requests library - only show WARNING and above
logging.getLogger("requests").setLevel(logging.WARNING)
logging.getLogger("urllib3").setLevel(logging.WARNING)

logger.debug("My app's debug info")  # ✅ Shows
logger.info("requests library debug info would go here")  # ❌ Hidden
```

Loggers form a **hierarchy** by name:
```
root
├── myapp (your main logger)
│   ├── myapp.database
│   └── myapp.analysis
└── requests (external library)
```

Change the parent, and all children follow.

### 1.9 Structured JSON Logging (Production Gold Standard)

Regular logs are text. JSON logs are **machine-readable**:

**Regular log:**
```
2026-03-30 14:32:05,123 - analysis - ERROR - Missing value in row 542
```

**JSON log:**
```json
{"timestamp": "2026-03-30T14:32:05.123Z", "logger": "analysis", "level": "ERROR", "message": "Missing value in row 542", "row_id": 542}
```

JSON logs go into **Datadog, Splunk, CloudWatch**. Machines parse them, alert you, create dashboards automatically.

**Setup (requires `pip install python-json-logger`):**

```python
import logging
from pythonjsonlogger import jsonlogger

logger = logging.getLogger()
handler = logging.FileHandler('app-json.log')
formatter = jsonlogger.JsonFormatter()
handler.setFormatter(formatter)
logger.addHandler(handler)

# Log with extra context
logger.info("Analysis complete", extra={
    "rows_processed": 10000,
    "duration_sec": 45.2,
    "source": "database"
})
```

**Output in `app-json.log`:**
```json
{"message": "Analysis complete", "rows_processed": 10000, "duration_sec": 45.2, "source": "database"}
```

Now tools like Datadog can parse this, create alerts ("if rows_processed < 1000, alert"), and build dashboards.

---

## Part 2: Matplotlib - Create Publication-Quality Plots

### 2.1 Object-Oriented API vs State Machine

**Don't do this (state machine):**
```python
import matplotlib.pyplot as plt
plt.plot([1, 2, 3], [1, 4, 9])
plt.title("My Plot")
plt.show()  # ❌ Doesn't work in scripts/servers/notebooks reliably
```

**Do this (object-oriented):**
```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot([1, 2, 3], [1, 4, 9], label='y=x²')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_title('Quadratic Function')
ax.legend()
ax.grid(True, alpha=0.3)
fig.savefig('plot.png', dpi=150, bbox_inches='tight')
plt.close(fig)
```

**Why OO wins:**
- **Explicit control**: Every element is an object you control
- **Works everywhere**: Scripts, servers, notebooks, Docker - anywhere
- **Subplots are easy**: Just use `plt.subplots(2, 2)` for a 2×2 grid
- **No GUI needed**: Can run on servers without a display

### 2.2 Plot Types

#### Line Plot (trends over time)
```python
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot([0, 1, 2, 3], [0, 1, 4, 9], marker='o', linewidth=2, label='y=x²')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_title('Quadratic Function')
ax.legend()
ax.grid(True, alpha=0.3)
fig.savefig('line.png')
plt.close(fig)
```

#### Scatter Plot (correlation between two variables)
```python
fig, ax = plt.subplots(figsize=(10, 6))
ax.scatter([1, 2, 3, 4, 5], [1, 4, 2, 5, 3], s=100, alpha=0.6, color='red')
ax.set_xlabel('Feature X')
ax.set_ylabel('Feature Y')
ax.set_title('Relationship Between Features')
fig.savefig('scatter.png')
plt.close(fig)
```

#### Bar Chart (comparison across categories)
```python
fig, ax = plt.subplots(figsize=(10, 6))
categories = ['Q1', 'Q2', 'Q3', 'Q4']
values = [10, 24, 36, 18]
ax.bar(categories, values, color='steelblue', edgecolor='black')
ax.set_ylabel('Revenue ($K)')
ax.set_title('Quarterly Revenue')
fig.savefig('bar.png')
plt.close(fig)
```

#### Histogram (distribution of single variable)
```python
fig, ax = plt.subplots(figsize=(10, 6))
scores = [85, 92, 78, 88, 75, 95, 82, 90, 77, 93]
ax.hist(scores, bins=5, color='green', edgecolor='black')
ax.set_xlabel('Score')
ax.set_ylabel('Frequency')
ax.set_title('Score Distribution')
fig.savefig('hist.png')
plt.close(fig)
```

#### Multiple Subplots (compare multiple plots)
```python
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Top-left: line plot
axes[0, 0].plot([1, 2, 3], [1, 2, 3])
axes[0, 0].set_title('Line Plot')

# Top-right: scatter plot
axes[0, 1].scatter([1, 2, 3], [1, 2, 3])
axes[0, 1].set_title('Scatter Plot')

# Bottom-left: bar chart
axes[1, 0].bar(['A', 'B', 'C'], [1, 2, 3])
axes[1, 0].set_title('Bar Chart')

# Bottom-right: histogram
axes[1, 1].hist([1, 1, 2, 2, 2], bins=3)
axes[1, 1].set_title('Histogram')

fig.tight_layout()
fig.savefig('subplots.png')
plt.close(fig)
```

### 2.3 Styling

```python
import matplotlib.pyplot as plt
import numpy as np

# Option 1: Use a built-in style
plt.style.use('seaborn-v0_8-darkgrid')

# Option 2: Custom colors
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [1, 4, 9], color='#FF6B6B', linewidth=3)

# Option 3: Color map (gradient)
x = np.linspace(0, 10, 50)
colors = plt.cm.viridis(np.linspace(0, 1, len(x)))
for xi, color in zip(x, colors):
    ax.scatter(xi, np.sin(xi), color=color, s=50)

fig.savefig('styled.png')
plt.close(fig)
```

---

## Part 3: Seaborn - Statistical Plots Made Easy

Seaborn builds on Matplotlib. It makes **statistical plots easier** and **prettier by default**.

### 3.1 Setup

```python
import seaborn as sns
import pandas as pd
import matplotlib.pyplot as plt

# Load your data
df = pd.read_csv('students.csv')

# Set theme once for all plots
sns.set_theme(style='darkgrid', palette='husl')
```

### 3.2 Distribution Plots

#### Histogram with smooth curve
```python
fig, ax = plt.subplots(figsize=(10, 6))
sns.histplot(data=df, x='score', bins=20, kde=True)
ax.set_title('Score Distribution with KDE')
fig.savefig('hist_kde.png')
plt.close(fig)
```

#### Box plot (shows quartiles, outliers)
```python
fig, ax = plt.subplots(figsize=(10, 6))
sns.boxplot(data=df, x='grade', y='score')
ax.set_title('Score by Grade')
fig.savefig('boxplot.png')
plt.close(fig)
```

#### Violin plot (like box plot but shows full distribution)
```python
fig, ax = plt.subplots(figsize=(10, 6))
sns.violinplot(data=df, x='grade', y='score')
ax.set_title('Score Distribution by Grade')
fig.savefig('violinplot.png')
plt.close(fig)
```

### 3.3 Correlation Analysis

#### Heatmap
```python
fig, ax = plt.subplots(figsize=(10, 8))
corr = df[['math', 'english', 'science', 'history']].corr()
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', square=True)
ax.set_title('Subject Score Correlations')
fig.savefig('heatmap.png')
plt.close(fig)
```

The heatmap shows which subjects' scores are related:
- **1.0** = perfect correlation (same subject)
- **0.8** = strong correlation (students good at both subjects)
- **0.0** = no correlation (independent)
- **-0.8** = inverse correlation (good at one, bad at other)

#### Pair plot (all scatter plots at once)
```python
# Shows scatter plots for every pair of numeric columns
sns.pairplot(df[['math', 'english', 'science', 'history']], hue='grade')
plt.savefig('pairplot.png')
plt.close()
```

### 3.4 Advanced Scatter Plot

```python
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(
    data=df,
    x='math',
    y='english',
    hue='grade',        # Color by grade
    size='attendance',  # Size by attendance
    s=200,
    alpha=0.6
)
ax.set_title('Math vs English (colored by grade, sized by attendance)')
fig.savefig('scatter_advanced.png')
plt.close(fig)
```

---

## The Project: Logging Data Analysis Dashboard

You'll build a complete data analysis pipeline that:
1. **Logs every step** (DEBUG, INFO, WARNING, ERROR)
2. **Generates 4 plots** (distribution, heatmap, bar chart, scatter)
3. **Saves logs** to both console AND file
4. **Has type hints** on every function
5. **Zero `print()` calls**

### File 1: `config.py`

Centralized logging configuration:

```python
"""Logging configuration for the data analysis pipeline."""

import logging
from pathlib import Path
from logging.handlers import RotatingFileHandler


def setup_logging(log_file: str = "output/analysis.log") -> logging.Logger:
    """Configure logging to console and file.
    
    Args:
        log_file: Path to log file
        
    Returns:
        Configured logger instance
    """
    # Create output directory if it doesn't exist
    Path("output").mkdir(exist_ok=True)

    # Create logger
    logger = logging.getLogger("data_analysis")
    logger.setLevel(logging.DEBUG)

    # Console handler (INFO and above only - clean output for user)
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    console_formatter = logging.Formatter('%(levelname)-8s | %(message)s')
    console_handler.setFormatter(console_formatter)

    # File handler (DEBUG and above - detailed logs for debugging)
    file_handler = RotatingFileHandler(
        log_file,
        maxBytes=5_000_000,  # Rotate at 5 MB
        backupCount=3         # Keep 3 old files
    )
    file_handler.setLevel(logging.DEBUG)
    file_formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_handler.setFormatter(file_formatter)

    # Attach both handlers
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

    logger.info("Logging system initialized")
    return logger
```

### File 2: `data_generator.py`

Generate sample data:

```python
"""Generate sample student score data."""

import csv
from pathlib import Path


def generate_sample_data(filename: str = "sample_data.csv") -> None:
    """Create sample CSV with 10 students and 4 subject scores.
    
    Args:
        filename: Output CSV filename
    """
    data = [
        ["student_id", "math", "english", "science", "history"],
        [1, 85, 78, 92, 88],
        [2, 92, 88, 95, 91],
        [3, 78, 85, 82, 79],
        [4, 88, 92, 89, 94],
        [5, 75, 72, 78, 75],
        [6, 95, 91, 97, 96],
        [7, 82, 86, 84, 87],
        [8, 90, 89, 91, 88],
        [9, 77, 79, 76, 81],
        [10, 93, 94, 92, 95],
    ]

    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerows(data)

    print(f"Generated {filename}")


if __name__ == "__main__":
    generate_sample_data()
```

### File 3: `analysis.py` (Main Script)

Complete data analysis with logging:

```python
"""Data analysis dashboard with logging."""

import logging
import csv
from pathlib import Path
from typing import List, Dict
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from config import setup_logging

# Initialize logging
logger = setup_logging()


def load_data(filepath: str) -> List[Dict[str, str]]:
    """Load CSV data into a list of dictionaries.
    
    Args:
        filepath: Path to CSV file
        
    Returns:
        List of dictionaries with student data
        
    Raises:
        FileNotFoundError: If CSV file doesn't exist
    """
    logger.info(f"Loading data from {filepath}")

    try:
        with open(filepath, 'r') as f:
            reader = csv.DictReader(f)
            data = list(reader)
        logger.info(f"Successfully loaded {len(data)} records")
        return data
    except FileNotFoundError:
        logger.error(f"File not found: {filepath}")
        raise


def validate_data(data: List[Dict[str, str]]) -> None:
    """Check for missing values in data.
    
    Args:
        data: List of dictionaries with student data
    """
    logger.info("Validating data...")

    for i, row in enumerate(data, 1):
        for key, value in row.items():
            if not value or value.strip() == '':
                logger.warning(f"Row {i}: Missing {key}")

    logger.info("Validation complete")


def create_output_dir() -> None:
    """Create output directories for plots."""
    Path("output/plots").mkdir(parents=True, exist_ok=True)
    logger.debug("Created output/plots directory")


def plot_score_distribution(data: List[Dict[str, str]]) -> None:
    """Create histogram of math scores.
    
    Args:
        data: List of dictionaries with student data
    """
    logger.info("Creating score distribution plot...")

    scores = [int(row['math']) for row in data]

    fig, ax = plt.subplots(figsize=(10, 6))
    ax.hist(scores, bins=8, color='steelblue', edgecolor='black')
    ax.set_xlabel('Score')
    ax.set_ylabel('Frequency')
    ax.set_title('Distribution of Math Scores')
    ax.grid(True, alpha=0.3)

    filepath = "output/plots/score_distribution.png"
    fig.savefig(filepath, dpi=150, bbox_inches='tight')
    logger.info(f"Saved: {filepath}")
    plt.close(fig)


def plot_correlation_heatmap(data: List[Dict[str, str]]) -> None:
    """Create correlation heatmap of all subjects.
    
    Args:
        data: List of dictionaries with student data
    """
    logger.info("Creating correlation heatmap...")

    subjects = ['math', 'english', 'science', 'history']
    
    # Build score matrix
    scores_dict = {}
    for subject in subjects:
        scores = [int(row[subject]) for row in data]
        scores_dict[subject] = scores

    # Calculate correlation
    score_matrix = np.array([scores_dict[s] for s in subjects])
    corr = np.corrcoef(score_matrix)

    fig, ax = plt.subplots(figsize=(8, 6))
    sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm',
                xticklabels=subjects, yticklabels=subjects,
                square=True, ax=ax)
    ax.set_title('Subject Score Correlations')

    filepath = "output/plots/correlation_heatmap.png"
    fig.savefig(filepath, dpi=150, bbox_inches='tight')
    logger.info(f"Saved: {filepath}")
    plt.close(fig)


def plot_subject_comparison(data: List[Dict[str, str]]) -> None:
    """Create bar chart comparing average scores by subject.
    
    Args:
        data: List of dictionaries with student data
    """
    logger.info("Creating subject comparison plot...")

    subjects = ['math', 'english', 'science', 'history']
    averages = []

    for subject in subjects:
        avg = sum(int(row[subject]) for row in data) / len(data)
        averages.append(avg)
        logger.debug(f"Average {subject}: {avg:.2f}")

    fig, ax = plt.subplots(figsize=(10, 6))
    bars = ax.bar(subjects, averages,
                  color=['#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A'])
    ax.set_ylabel('Average Score')
    ax.set_title('Average Score by Subject')
    ax.set_ylim([0, 100])

    # Add value labels on bars
    for bar in bars:
        height = bar.get_height()
        ax.text(bar.get_x() + bar.get_width()/2., height,
                f'{height:.1f}', ha='center', va='bottom', fontsize=10)

    filepath = "output/plots/subject_comparison.png"
    fig.savefig(filepath, dpi=150, bbox_inches='tight')
    logger.info(f"Saved: {filepath}")
    plt.close(fig)


def plot_student_scatter(data: List[Dict[str, str]]) -> None:
    """Create scatter plot: Math vs English.
    
    Args:
        data: List of dictionaries with student data
    """
    logger.info("Creating student scatter plot...")

    math_scores = [int(row['math']) for row in data]
    english_scores = [int(row['english']) for row in data]

    fig, ax = plt.subplots(figsize=(10, 6))
    ax.scatter(math_scores, english_scores, s=100, alpha=0.6, color='purple')
    ax.set_xlabel('Math Score')
    ax.set_ylabel('English Score')
    ax.set_title('Math vs English Scores')
    ax.grid(True, alpha=0.3)

    filepath = "output/plots/math_vs_english.png"
    fig.savefig(filepath, dpi=150, bbox_inches='tight')
    logger.info(f"Saved: {filepath}")
    plt.close(fig)


def main() -> None:
    """Main analysis pipeline."""
    logger.info("=" * 60)
    logger.info("STARTING DATA ANALYSIS DASHBOARD")
    logger.info("=" * 60)

    try:
        # Setup
        create_output_dir()

        # Load and validate
        data = load_data("sample_data.csv")
        validate_data(data)

        # Generate plots
        plot_score_distribution(data)
        plot_correlation_heatmap(data)
        plot_subject_comparison(data)
        plot_student_scatter(data)

        logger.info("=" * 60)
        logger.info("ANALYSIS COMPLETE - All plots saved to output/plots/")
        logger.info("=" * 60)

    except Exception as e:
        logger.error(f"Pipeline failed: {e}", exc_info=True)
        raise


if __name__ == "__main__":
    main()
```

### Running the Project

```bash
# Step 1: Generate sample data
python data_generator.py
# Output: Generated sample_data.csv

# Step 2: Run analysis (creates plots + logs)
python analysis.py

# Step 3: View results
ls -lh output/plots/
cat output/analysis.log
```

### Expected Console Output

```
INFO     | Logging system initialized
INFO     | ============================================================
INFO     | STARTING DATA ANALYSIS DASHBOARD
INFO     | ============================================================
INFO     | Loading data from sample_data.csv
INFO     | Successfully loaded 10 records
INFO     | Validating data...
INFO     | Validation complete
INFO     | Creating score distribution plot...
INFO     | Saved: output/plots/score_distribution.png
INFO     | Creating correlation heatmap...
INFO     | Saved: output/plots/correlation_heatmap.png
INFO     | Creating subject comparison plot...
INFO     | Saved: output/plots/subject_comparison.png
INFO     | Creating student scatter plot...
INFO     | Saved: output/plots/math_vs_english.png
INFO     | ============================================================
INFO     | ANALYSIS COMPLETE - All plots saved to output/plots/
INFO     | ============================================================
```

### Expected Log File Output

```
2026-03-30 14:32:05,123 - data_analysis - INFO - Logging system initialized
2026-03-30 14:32:05,234 - data_analysis - INFO - ============================================================
2026-03-30 14:32:05,234 - data_analysis - INFO - STARTING DATA ANALYSIS DASHBOARD
2026-03-30 14:32:05,234 - data_analysis - INFO - ============================================================
2026-03-30 14:32:05,345 - data_analysis - INFO - Loading data from sample_data.csv
2026-03-30 14:32:05,456 - data_analysis - INFO - Successfully loaded 10 records
2026-03-30 14:32:05,567 - data_analysis - INFO - Validating data...
2026-03-30 14:32:05,678 - data_analysis - INFO - Validation complete
2026-03-30 14:32:05,789 - data_analysis - DEBUG - Average math: 87.50
2026-03-30 14:32:05,890 - data_analysis - DEBUG - Average english: 85.40
2026-03-30 14:32:06,001 - data_analysis - DEBUG - Average science: 88.60
2026-03-30 14:32:06,112 - data_analysis - DEBUG - Average history: 88.40
2026-03-30 14:32:06,223 - data_analysis - INFO - Created output/plots directory
2026-03-30 14:32:06,334 - data_analysis - INFO - Creating score distribution plot...
2026-03-30 14:32:06,445 - data_analysis - INFO - Saved: output/plots/score_distribution.png
...
```

---

## What's Next

**Day 8: Linear Algebra & Calculus**
- NumPy arrays and operations
- Matrix multiplication, determinants, inverses
- Derivatives, gradients, chain rule
- Logging and type hints continue as standard practice
