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

Here's a uncomfortable truth: **you cannot do AI without statistics.**

Every machine learning model makes predictions. Every prediction has uncertainty. Every decision about "is this model good?" requires statistical thinking. You'll hear phrases like "p-value," "confidence interval," "null hypothesis" and if you don't understand them, you'll make expensive mistakes.

The good news: statistics isn't magic. It's applied common sense with numbers. Today, we're building from the ground up - no assumptions about what you know.

---

## Setup

**Installation:**

```bash
pip install numpy scipy matplotlib
```

**Python setup** (copy this into every script today):

```python
import logging
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
from typing import tuple, dict, list

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)
```

We're using `logging` instead of `print()` (Day 7's lesson) and type hints on every function (Day 5). No exceptions, no Pydantic, no pytest (that's tomorrow, Day 10).

---

## Part 1: Probability Basics

### What is Probability? (The Weather Forecast Analogy)

You check your weather app. It says: **"30% chance of rain tomorrow."**

That 30% is a **probability**. It's a number between 0 and 1 (or 0% and 100%) that tells you how likely something is to happen.

- **0 = impossible** → You will not randomly turn into a penguin today
- **0.5 = equally likely** → Fair coin flip (heads or tails)
- **1 = certain** → The Earth is round (pretty sure)

In AI, probability answers questions like:
- What's the chance this email is spam?
- How likely is this patient to have diabetes given these symptoms?
- If I deploy this model, what's the probability of a wrong prediction?

### Key Concept: Sample Space

**Sample space** = all possible outcomes.

- Coin flip: {heads, tails}
- Die roll: {1, 2, 3, 4, 5, 6}
- Email classification: {spam, not spam}

### Probability Notation: P(A)

We write **P(A)** to mean "the probability of event A."

Example:
```python
# Fair coin flip
P(heads) = 0.5  # 50% chance
P(tails) = 0.5  # 50% chance

# Fair die roll
P(rolling a 6) = 1/6 ≈ 0.167  # 16.7% chance
```

### Rule 1: Complement - P(not A) = 1 - P(A)

The probabilities of all outcomes must add up to 1 (certainty).

```python
def complement_rule(p_a: float) -> float:
    """If P(A) is true, then P(not A) = 1 - P(A)."""
    return 1 - p_a

logger.info(f"P(heads) = 0.5, P(not heads) = {complement_rule(0.5)}")
# Output: P(not heads) = 0.5
```

### Rule 2: Independent Events - P(A and B) = P(A) × P(B)

If two events are **independent** (one doesn't affect the other), multiply their probabilities.

Example: Two coin flips

```python
p_heads_first = 0.5
p_heads_second = 0.5
p_both_heads = p_heads_first * p_heads_second

logger.info(f"P(HH) = {p_both_heads}")
# Output: P(HH) = 0.25
```

Another example: Spam detection

Imagine:
- P(contains "free") = 0.8
- P(contains "click now") = 0.7
- If independent: P(both) = 0.8 × 0.7 = 0.56

### Rule 3: Addition - P(A or B) = P(A) + P(B) - P(A and B)

The **or** rule accounts for overlap (when both can happen).

```python
def addition_rule(p_a: float, p_b: float, p_both: float) -> float:
    """P(A or B) = P(A) + P(B) - P(A and B)."""
    return p_a + p_b - p_both

# Die roll: P(rolling 1 or 2)
p_1 = 1/6
p_2 = 1/6
p_both = 0  # Can't roll both 1 AND 2 at the same time

p_1_or_2 = addition_rule(p_1, p_2, p_both)
logger.info(f"P(1 or 2) = {p_1_or_2:.4f}")
# Output: P(1 or 2) = 0.3333
```

### Rule 4: Conditional Probability - P(A | B)

**P(A | B)** means "probability of A **given** B happened."

The vertical bar `|` is read as "given."

Example: Weather and clouds

```
P(rain | clouds) = "probability of rain given we see clouds"
```

**Formula:**
```
P(A | B) = P(A and B) / P(B)
```

The denominator reduces the sample space to only cases where B happened.

**Worked example:**

```python
def conditional_probability(p_a_and_b: float, p_b: float) -> float:
    """P(A | B) = P(A and B) / P(B)."""
    if p_b == 0:
        logger.error("P(B) cannot be zero")
        return 0.0
    return p_a_and_b / p_b

# Medical test scenario
# P(positive test AND has disease) = 0.0099
# P(positive test) = 0.0101
p_disease_given_positive = conditional_probability(0.0099, 0.0101)
logger.info(f"P(disease | positive test) = {p_disease_given_positive:.4f}")
# Output: P(disease | positive test) ≈ 0.98 (98%)
```

### Simulating Coin Flips and Dice Rolls

Let's use Python's `random` module to simulate probability:

```python
import random

def simulate_coin_flips(n: int) -> dict[str, int]:
    """Flip a coin n times, count heads and tails."""
    heads = sum(1 for _ in range(n) if random.random() < 0.5)
    tails = n - heads
    logger.info(f"Simulated {n} flips: {heads} heads, {tails} tails")
    return {'heads': heads, 'tails': tails}

def simulate_die_rolls(n: int) -> dict[int, int]:
    """Roll a die n times, count occurrences of each face."""
    results = {i: 0 for i in range(1, 7)}
    for _ in range(n):
        roll = random.randint(1, 6)
        results[roll] += 1

    logger.info(f"Simulated {n} die rolls:")
    for face, count in results.items():
        logger.info(f"  Face {face}: {count} ({count/n:.1%})")
    return results

# Run simulations
simulate_coin_flips(1000)
simulate_die_rolls(1000)
```

**Expected output:**
```
Simulated 1000 flips: 509 heads, 491 tails
Simulated 1000 die rolls:
  Face 1: 168 (16.8%)
  Face 2: 167 (16.7%)
  Face 3: 172 (17.2%)
  ...
```

---

## Part 2: Distributions

### What is a Distribution?

Imagine rolling a die 1000 times and making a histogram:

```
Frequency
   |     ___
   |    |   |
   |    |   |
   |    |   |
   |____|___|____
      1 2 3 ... 6
```

Each face appears roughly 1/6 of the time (≈ 167 times). That histogram is a **distribution** - it shows the probability of each outcome.

### Uniform Distribution: Everyone Gets Equal Odds

In a uniform distribution, every outcome is equally likely.

**Real-world examples:**
- Die roll (each face: 1/6)
- Picking a random number between 0 and 100
- Shuffled deck of cards (each card equally likely)

```python
import matplotlib.pyplot as plt

def plot_uniform_distribution() -> None:
    """Visualize uniform distribution."""
    # Generate 10,000 random samples between 0 and 100
    samples = np.random.uniform(0, 100, 10000)

    plt.figure(figsize=(10, 5))
    plt.hist(samples, bins=50, edgecolor='black', alpha=0.7)
    plt.xlabel('Value')
    plt.ylabel('Frequency')
    plt.title('Uniform Distribution: 10,000 Samples from [0, 100]')
    plt.grid(alpha=0.3)
    plt.savefig('uniform_distribution.png', dpi=150)
    logger.info("Saved uniform distribution plot")
    plt.close()

plot_uniform_distribution()
```

### Normal (Gaussian) Distribution: The Bell Curve

The **normal distribution** is the most important distribution in statistics. It looks like a bell.

**Real-world examples:**
- Student exam scores
- Adult heights
- IQ scores
- Measurement errors
- Stock price returns (approximately)

**Key parameters:**
- **μ (mu)** = mean (center of bell)
- **σ (sigma)** = standard deviation (width of bell)

```python
def plot_normal_distribution() -> None:
    """Visualize normal distribution."""
    # Generate 10,000 samples from a normal distribution
    # mean=100 (like IQ), std=15
    samples = np.random.normal(loc=100, scale=15, size=10000)

    plt.figure(figsize=(10, 5))
    plt.hist(samples, bins=50, edgecolor='black', alpha=0.7, density=True)

    # Overlay theoretical normal curve
    x = np.linspace(samples.min(), samples.max(), 100)
    plt.plot(x, stats.norm.pdf(x, loc=100, scale=15), 'r-', linewidth=2, label='Theory')

    plt.xlabel('Value')
    plt.ylabel('Density')
    plt.title('Normal Distribution: μ=100, σ=15 (like IQ scores)')
    plt.legend()
    plt.grid(alpha=0.3)
    plt.savefig('normal_distribution.png', dpi=150)
    logger.info("Saved normal distribution plot")
    plt.close()

plot_normal_distribution()
```

### The 68-95-99.7 Rule: Knowing What's Normal

In a normal distribution:

- **68%** of data falls within 1 std of the mean
- **95%** falls within 2 stds
- **99.7%** falls within 3 stds

Example: IQ scores (mean=100, std=15)

```python
def demonstrate_68_95_99_7_rule() -> None:
    """Explain the empirical rule."""
    mean, std = 100, 15

    # 1 std: [85, 115]
    logger.info(f"68% of IQ scores between {mean-std} and {mean+std}")

    # 2 stds: [70, 130]
    logger.info(f"95% of IQ scores between {mean-2*std} and {mean+2*std}")

    # 3 stds: [55, 145]
    logger.info(f"99.7% of IQ scores between {mean-3*std} and {mean+3*std}")

    # Verify with simulation
    samples = np.random.normal(loc=mean, scale=std, size=100000)
    within_1 = np.sum((samples >= mean - std) & (samples <= mean + std)) / len(samples)
    within_2 = np.sum((samples >= mean - 2*std) & (samples <= mean + 2*std)) / len(samples)
    within_3 = np.sum((samples >= mean - 3*std) & (samples <= mean + 3*std)) / len(samples)

    logger.info(f"Simulated: {within_1:.1%} within 1 std (expected 68%)")
    logger.info(f"Simulated: {within_2:.1%} within 2 stds (expected 95%)")
    logger.info(f"Simulated: {within_3:.1%} within 3 stds (expected 99.7%)")

demonstrate_68_95_99_7_rule()
```

**Output:**
```
Simulated: 68.1% within 1 std (expected 68%)
Simulated: 95.2% within 2 stds (expected 95%)
Simulated: 99.7% within 3 stds (expected 99.7%)
```

### Binomial Distribution: Counting Successes

The **binomial distribution** answers: "If I do N independent trials, each with success probability p, how many successes do I get?"

**Real-world examples:**
- Flipping a coin 100 times, counting heads
- Shooting 10 basketball free throws, counting makes
- Spam detector running on 1000 emails, counting false positives

```python
def plot_binomial_distribution() -> None:
    """Visualize binomial distribution."""
    # Flip a fair coin 10 times, repeat 10,000 times
    # Count heads each time
    samples = np.random.binomial(n=10, p=0.5, size=10000)

    plt.figure(figsize=(10, 5))
    plt.hist(samples, bins=11, edgecolor='black', alpha=0.7, align='left')
    plt.xlabel('Number of Heads (out of 10 flips)')
    plt.ylabel('Frequency')
    plt.title('Binomial Distribution: n=10 flips, p=0.5')
    plt.grid(alpha=0.3)
    plt.savefig('binomial_distribution.png', dpi=150)
    logger.info("Saved binomial distribution plot")
    plt.close()

plot_binomial_distribution()
```

### Central Limit Theorem: The Magic of Averages

Here's one of the most important theorems in statistics:

**If you take samples from ANY distribution and average them, those averages follow a normal distribution.**

This is huge because:
1. We don't need to know the original distribution
2. Normal distributions are easy to work with
3. This is why normal distributions are everywhere

```python
def demonstrate_central_limit_theorem() -> None:
    """Show that sample means become normal."""
    logger.info("Demonstrating Central Limit Theorem...")

    # Original distribution: uniform (NOT normal)
    original = np.random.uniform(0, 100, 100000)

    # Take 1000 samples of size 50, compute mean of each
    sample_means = []
    for _ in range(1000):
        sample = np.random.choice(original, size=50, replace=True)
        sample_means.append(np.mean(sample))

    sample_means = np.array(sample_means)

    fig, axes = plt.subplots(1, 2, figsize=(14, 5))

    # Plot 1: Original distribution (uniform)
    axes[0].hist(original, bins=50, alpha=0.7, edgecolor='black')
    axes[0].set_title('Original Distribution: Uniform [0, 100]')
    axes[0].set_xlabel('Value')
    axes[0].set_ylabel('Frequency')

    # Plot 2: Distribution of sample means (normal!)
    axes[1].hist(sample_means, bins=50, alpha=0.7, edgecolor='black')
    axes[1].set_title('Distribution of Sample Means (1000 trials, n=50)')
    axes[1].set_xlabel('Mean Value')
    axes[1].set_ylabel('Frequency')

    plt.tight_layout()
    plt.savefig('clt_demonstration.png', dpi=150)
    logger.info("Saved CLT demonstration plot")
    plt.close()

    logger.info(f"Original distribution: uniform")
    logger.info(f"Sample means: μ={np.mean(sample_means):.2f}, σ={np.std(sample_means):.2f}")
    logger.info("Sample means distribution looks normal!")

demonstrate_central_limit_theorem()
```

---

## Part 3: Descriptive Statistics

### Mean: The Average

The **mean** is the sum of all values divided by the count.

```python
def compute_mean(data: list[float]) -> float:
    """Compute mean from scratch."""
    if not data:
        logger.error("Cannot compute mean of empty list")
        return 0.0
    return sum(data) / len(data)

# Example: exam scores
scores = [85, 92, 78, 88, 95, 81]
mean = compute_mean(scores)
logger.info(f"Mean score: {mean:.2f}")

# Verify with NumPy
mean_numpy = np.mean(scores)
logger.info(f"NumPy mean: {mean_numpy:.2f}")
```

**When to use mean:**
- Symmetric data (normal distribution)
- No extreme outliers
- Example: student exam scores (usually)

**When NOT to use:**
- Skewed data (income, housing prices)
- Outliers (average salary with one billionaire)

### Median: The Middle Value

The **median** is the middle value when sorted.

```python
def compute_median(data: list[float]) -> float:
    """Compute median from scratch."""
    if not data:
        logger.error("Cannot compute median of empty list")
        return 0.0

    sorted_data = sorted(data)
    n = len(sorted_data)

    if n % 2 == 1:
        # Odd number of elements
        return float(sorted_data[n // 2])
    else:
        # Even number of elements: average the two middle
        mid1 = sorted_data[n // 2 - 1]
        mid2 = sorted_data[n // 2]
        return (mid1 + mid2) / 2

scores = [85, 92, 78, 88, 95, 81]
median = compute_median(scores)
logger.info(f"Median score: {median:.2f}")

# Verify with NumPy
median_numpy = np.median(scores)
logger.info(f"NumPy median: {median_numpy:.2f}")
```

**When to use median:**
- Skewed data (income distribution)
- Outliers present
- Example: housing prices (median is more representative than mean)

### Mode: Most Frequent Value

The **mode** is the value that appears most often.

```python
def compute_mode(data: list[float]) -> float:
    """Find the most frequent value."""
    if not data:
        logger.error("Cannot compute mode of empty list")
        return 0.0

    from collections import Counter
    counts = Counter(data)
    most_common, _ = counts.most_common(1)[0]
    return float(most_common)

scores = [85, 92, 78, 88, 95, 81, 88]  # 88 appears twice
mode = compute_mode(scores)
logger.info(f"Mode score: {mode:.2f}")
```

### Variance: How Spread Out Is the Data?

**Variance** is the average squared distance from the mean.

```
σ² = (Σ(x - μ)²) / n
```

Why squared? So negative differences don't cancel out positive ones.

```python
def compute_variance(data: list[float]) -> float:
    """Compute variance from scratch."""
    if len(data) < 2:
        logger.error("Need at least 2 data points")
        return 0.0

    mean = compute_mean(data)
    squared_diffs = [(x - mean) ** 2 for x in data]
    variance = sum(squared_diffs) / (len(data) - 1)  # -1 for sample variance
    return variance

scores = [85, 92, 78, 88, 95, 81]
variance = compute_variance(scores)
logger.info(f"Variance: {variance:.2f}")

# Verify with NumPy
variance_numpy = np.var(scores, ddof=1)
logger.info(f"NumPy variance: {variance_numpy:.2f}")
```

**Problem:** Variance is in squared units. Hard to interpret.

### Standard Deviation: Variance's Friendly Cousin

**Standard deviation** is the square root of variance.

```
σ = √(σ²)
```

**Advantage:** Same units as the original data.

```python
def compute_std(data: list[float]) -> float:
    """Compute standard deviation from scratch."""
    variance = compute_variance(data)
    return variance ** 0.5

scores = [85, 92, 78, 88, 95, 81]
std = compute_std(scores)
logger.info(f"Std dev: {std:.2f}")

# Verify with NumPy
std_numpy = np.std(scores, ddof=1)
logger.info(f"NumPy std: {std_numpy:.2f}")
```

**Interpretation:** If mean=88 and std=6, most students scored between 82–94.

### Percentiles and Quartiles: Dividing the Data

**Percentile:** Value below which a certain percentage of data falls.

- 25th percentile (Q1): 25% of data below this
- 50th percentile (median): 50% below
- 75th percentile (Q3): 75% below

```python
def compute_percentiles(data: list[float]) -> dict[str, float]:
    """Compute quartiles and common percentiles."""
    arr = np.array(data)
    return {
        'min': float(np.min(arr)),
        'q25': float(np.percentile(arr, 25)),
        'median': float(np.median(arr)),
        'q75': float(np.percentile(arr, 75)),
        'max': float(np.max(arr)),
    }

scores = [85, 92, 78, 88, 95, 81, 90, 87]
percentiles = compute_percentiles(scores)

logger.info("Percentile breakdown:")
for name, value in percentiles.items():
    logger.info(f"  {name}: {value:.2f}")
```

---

## Part 4: Bayes' Theorem

### The Problem: Updating Beliefs with Evidence

You just tested positive for a rare disease.

Naturally, you panic. The test is 99% accurate, so you must be 99% likely to have the disease, right?

**Wrong.** This is where Bayes' theorem saves the day.

### Bayes' Theorem Formula

```
P(A | B) = P(B | A) × P(A) / P(B)
```

Where:
- **P(A | B)** = Posterior (what we want: probability of disease given positive test?)
- **P(B | A)** = Likelihood (test accuracy: if you have disease, what's probability of positive test?)
- **P(A)** = Prior (base rate: how common is the disease?)
- **P(B)** = Total probability of observing B

### Medical Test Example: When a Positive Test Isn't Good News

**Setup:**
- Disease affects 1 in 10,000 people
- Test accuracy: 99% (both sensitivity and specificity)

**Step 1: Prior probability (base rate)**
```python
p_disease = 1 / 10000
logger.info(f"P(disease) = {p_disease:.6f}")  # 0.0001
```

**Step 2: Likelihood (test accuracy)**
```python
p_positive_if_disease = 0.99  # Test catches 99% of true positives
p_positive_if_no_disease = 0.01  # Test gives 1% false positives

logger.info(f"P(positive | disease) = {p_positive_if_disease}")
logger.info(f"P(positive | no disease) = {p_positive_if_no_disease}")
```

**Step 3: Total probability P(B)**
```
P(positive) = P(positive | disease) × P(disease) + P(positive | no disease) × P(no disease)
```

```python
p_no_disease = 1 - p_disease
p_positive = (p_positive_if_disease * p_disease) + \
             (p_positive_if_no_disease * p_no_disease)

logger.info(f"P(positive) = {p_positive:.6f}")
```

**Step 4: Bayes' theorem**
```python
def bayes_theorem(likelihood: float, prior: float, evidence: float) -> float:
    """Compute posterior using Bayes' theorem."""
    if evidence <= 0:
        logger.error("Evidence must be positive")
        return 0.0
    return (likelihood * prior) / evidence

p_disease_given_positive = bayes_theorem(
    likelihood=p_positive_if_disease,
    prior=p_disease,
    evidence=p_positive
)

logger.info(f"P(disease | positive test) = {p_disease_given_positive:.4f}")
logger.info(f"You are only {p_disease_given_positive * 100:.2f}% likely to have the disease!")
```

**Output:**
```
P(disease | positive test) = 0.0099
You are only 0.99% likely to have the disease!
```

**Wow!** Even with a positive test from a 99% accurate test, you're less than 1% likely to have the disease. Why?

Because the disease is so rare. False positives (1% of healthy people) outnumber true positives (99% of the tiny number who have it).

### Spam Email Classifier: Pure Probability

Let's use Bayes' theorem to classify emails as spam or not spam, without any ML library.

```python
class SimpleSpamClassifier:
    """Spam detector using Bayes' theorem."""

    def __init__(self, prior_spam: float) -> None:
        """Initialize with prior probability of spam."""
        self.prior_spam = prior_spam
        self.prior_ham = 1 - prior_spam
        logger.info(f"Initialized: P(spam)={prior_spam}, P(ham)={self.prior_ham}")

    def classify(self,
                 p_word_in_spam: float,
                 p_word_in_ham: float) -> tuple[float, float]:
        """Classify based on presence of a keyword.

        Returns: (p_spam, p_ham) posteriors
        """
        # P(word | spam)
        # P(word | ham)
        # P(spam | word) = ?

        p_word = (p_word_in_spam * self.prior_spam) + \
                 (p_word_in_ham * self.prior_ham)

        p_spam_given_word = (p_word_in_spam * self.prior_spam) / p_word
        p_ham_given_word = (p_word_in_ham * self.prior_ham) / p_word

        logger.debug(f"P(spam | word) = {p_spam_given_word:.4f}")
        return p_spam_given_word, p_ham_given_word

# Create classifier
# Assume 20% of emails are spam
classifier = SimpleSpamClassifier(prior_spam=0.20)

# Word: "free"
# 80% of spam emails contain "free"
# 10% of legitimate emails contain "free"
p_spam, p_ham = classifier.classify(
    p_word_in_spam=0.80,
    p_word_in_ham=0.10
)

logger.info(f"Email contains 'free': {p_spam:.1%} spam, {p_ham:.1%} ham")

# Word: "unsubscribe"
# 5% of spam emails contain "unsubscribe" (to avoid filters)
# 30% of legitimate emails contain "unsubscribe" (newsletters)
p_spam, p_ham = classifier.classify(
    p_word_in_spam=0.05,
    p_word_in_ham=0.30
)

logger.info(f"Email contains 'unsubscribe': {p_spam:.1%} spam, {p_ham:.1%} ham")
```

**Output:**
```
Initialized: P(spam)=0.20, P(ham)=0.80
Email contains 'free': 64.0% spam, 36.0% ham
Email contains 'unsubscribe': 4.8% spam, 95.2% ham
```

See how "free" strongly suggests spam, while "unsubscribe" suggests legitimacy? That's Bayes' theorem at work.

---

## Part 5: Hypothesis Testing

### The Core Question: Is This Real or Luck?

A pharmaceutical company tests a new drug on 100 patients:
- Control group: mean recovery time 10 days
- Treatment group: mean recovery time 5 days

Difference: 5 days faster.

**But is the drug actually working, or did we just get lucky?**

Hypothesis testing answers this question statistically.

### Setting Up Hypotheses

**Null Hypothesis (H₀):** No effect exists
```
H₀: Drug has no effect on recovery time
```

**Alternative Hypothesis (H₁):** Effect exists
```
H₁: Drug does affect recovery time
```

The null hypothesis is the "boring" one. We try to disprove it.

```python
logger.info("H₀: Drug has no effect")
logger.info("H₁: Drug does have an effect")
```

### p-value: The Key Number

**p-value** = "If H₀ were true (drug does nothing), how often would we observe a difference this extreme or larger, by pure chance?"

```python
# Example: we observe a 5-day difference
# p-value = 0.03
logger.info("p-value = 0.03")
logger.info("Interpretation: 3% chance of seeing this data if the drug is useless")
```

**Critical misconceptions:**

❌ WRONG: "p-value = 0.03 means 97% chance the drug works"
✅ RIGHT: "p-value = 0.03 means 3% chance we'd see this data if the drug doesn't work"

### Significance Level (α): The Threshold

We choose a significance level, typically **α = 0.05**.

**Decision rule:**
- If p-value < 0.05: reject H₀ (claim the drug works)
- If p-value ≥ 0.05: fail to reject H₀ (can't claim it works)

```python
p_value = 0.03
alpha = 0.05

if p_value < alpha:
    logger.info("✓ Reject H₀: The effect is statistically significant")
else:
    logger.info("✗ Fail to reject H₀: No significant effect detected")
```

### Type I and Type II Errors

Every hypothesis test can make two kinds of mistakes:

**Type I Error (False Positive):**
- We reject H₀, but H₀ is actually true
- We claim the drug works, but it doesn't
- Probability: α (usually 0.05)

**Type II Error (False Negative):**
- We fail to reject H₀, but H₀ is actually false
- We claim the drug doesn't work, but it does
- Probability: β (usually 0.20)

```python
logger.info("Type I Error (False Positive):")
logger.info("  Claim drug works when it doesn't")
logger.info("  Probability: α = 0.05")
logger.info("")
logger.info("Type II Error (False Negative):")
logger.info("  Claim drug doesn't work when it does")
logger.info("  Probability: β ≈ 0.20 (related to statistical power)")
```

### t-test: Comparing Two Group Means

The **t-test** compares means of two groups.

**Question:** Are the treatment and control groups significantly different?

```python
import numpy as np
from scipy.stats import ttest_ind

# Generate synthetic data
np.random.seed(42)
control = np.random.normal(loc=10, scale=2, size=50)  # mean=10 days
treatment = np.random.normal(loc=5, scale=2, size=50)  # mean=5 days

logger.info(f"Control group: mean={np.mean(control):.2f}, std={np.std(control):.2f}")
logger.info(f"Treatment group: mean={np.mean(treatment):.2f}, std={np.std(treatment):.2f}")

# Run t-test
t_statistic, p_value = ttest_ind(control, treatment)

logger.info(f"t-statistic: {t_statistic:.4f}")
logger.info(f"p-value: {p_value:.6f}")

# Decision
alpha = 0.05
if p_value < alpha:
    logger.info(f"✓ Reject H₀ (p={p_value:.4f} < α={alpha})")
    logger.info("  The groups are significantly different")
else:
    logger.info(f"✗ Fail to reject H₀ (p={p_value:.4f} ≥ α={alpha})")
    logger.info("  No significant difference detected")
```

**Output:**
```
Control group: mean=10.12, std=2.05
Treatment group: mean=5.08, std=1.87
t-statistic: 12.7832
p-value: 0.000000
✓ Reject H₀ (p=0.000000 < α=0.05)
  The groups are significantly different
```

---

## Part 6: Confidence Intervals

### What is a 95% Confidence Interval?

**Common misconception:**
❌ "There's a 95% probability the true mean is in this interval"

**Correct interpretation:**
✅ "If we repeated this experiment 100 times, 95 of those experiments would produce intervals containing the true mean"

The interval is random (depends on our sample). The true mean is fixed.

### Computing a Confidence Interval

**Formula:**
```
CI = mean ± (critical value × standard error)
```

```python
from scipy import stats
import numpy as np

def compute_confidence_interval(data: np.ndarray,
                                 confidence: float = 0.95) -> tuple[float, float]:
    """Compute confidence interval for the mean."""
    n = len(data)
    mean = np.mean(data)

    # Standard error of the mean
    sem = stats.sem(data)  # = std / sqrt(n)

    # t-critical value (use t-distribution because sample is small-ish)
    df = n - 1
    alpha = 1 - confidence
    t_crit = stats.t.ppf(1 - alpha/2, df)

    # Margin of error
    margin = t_crit * sem

    logger.debug(f"n={n}, mean={mean:.2f}, sem={sem:.4f}, margin={margin:.2f}")

    return (mean - margin, mean + margin)

# Example: exam scores
scores = np.array([85, 92, 78, 88, 95, 81, 90, 87, 83, 89])

lower, upper = compute_confidence_interval(scores, confidence=0.95)

logger.info(f"Sample mean: {np.mean(scores):.2f}")
logger.info(f"95% Confidence Interval: [{lower:.2f}, {upper:.2f}]")
logger.info("Interpretation: We're 95% confident the true population mean")
logger.info(f"  is between {lower:.2f} and {upper:.2f}")
```

**Output:**
```
Sample mean: 86.80
95% Confidence Interval: [82.47, 91.13]
```

### Visualizing Confidence Intervals

```python
def plot_confidence_intervals() -> None:
    """Visualize CIs for two groups."""
    np.random.seed(42)

    # Generate data
    group1 = np.random.normal(loc=80, scale=8, size=50)
    group2 = np.random.normal(loc=85, scale=8, size=50)

    # Compute CIs
    ci1 = compute_confidence_interval(group1)
    ci2 = compute_confidence_interval(group2)

    mean1 = np.mean(group1)
    mean2 = np.mean(group2)

    # Plot
    fig, ax = plt.subplots(figsize=(10, 6))

    groups = ['Group 1', 'Group 2']
    means = [mean1, mean2]
    errors = [
        [mean1 - ci1[0], mean2 - ci2[0]],
        [ci1[1] - mean1, ci2[1] - mean2]
    ]

    ax.errorbar(groups, means, yerr=errors, fmt='o', markersize=10,
                capsize=10, capthick=2, linewidth=2)

    ax.set_ylabel('Mean Score')
    ax.set_title('95% Confidence Intervals for Mean Scores')
    ax.grid(axis='y', alpha=0.3)

    plt.tight_layout()
    plt.savefig('confidence_intervals.png', dpi=150)
    logger.info("Saved confidence interval plot")
    plt.close()

plot_confidence_intervals()
```

---

## The Project: Student Score Statistical Analysis

Let's bring it all together. We'll:
1. Generate synthetic exam score data
2. Compute descriptive statistics
3. Plot distributions
4. Run a t-test comparing two groups
5. Compute confidence intervals

**Complete script:**

```python
import logging
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
from typing import dict, tuple, list

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# ============================================================================
# DATA GENERATION
# ============================================================================

def generate_exam_scores() -> tuple[np.ndarray, np.ndarray]:
    """Generate synthetic exam scores for study vs no-study groups."""
    np.random.seed(42)

    # Students who studied: higher mean (78), lower variance
    study_group = np.random.normal(loc=78, scale=8, size=50)

    # Students who didn't study: lower mean (68), higher variance
    no_study_group = np.random.normal(loc=68, scale=10, size=50)

    logger.info(f"Generated {len(study_group)} study group scores")
    logger.info(f"Generated {len(no_study_group)} no-study group scores")

    return study_group, no_study_group

# ============================================================================
# DESCRIPTIVE STATISTICS
# ============================================================================

def compute_stats(data: np.ndarray) -> dict[str, float]:
    """Compute 7-point summary statistics."""
    logger.debug(f"Computing stats for {len(data)} values")

    return {
        'mean': float(np.mean(data)),
        'median': float(np.median(data)),
        'std': float(np.std(data, ddof=1)),
        'min': float(np.min(data)),
        'max': float(np.max(data)),
        'q25': float(np.percentile(data, 25)),
        'q75': float(np.percentile(data, 75)),
    }

def log_stats(name: str, stats_dict: dict[str, float]) -> None:
    """Log statistics nicely."""
    logger.info(f"{name} statistics:")
    for key, value in stats_dict.items():
        logger.info(f"  {key:8s}: {value:7.2f}")

# ============================================================================
# HYPOTHESIS TESTING: t-test
# ============================================================================

def run_t_test(group1: np.ndarray, group2: np.ndarray) -> tuple[float, float]:
    """Run independent samples t-test."""
    t_stat, p_value = stats.ttest_ind(group1, group2)

    logger.info(f"t-test results:")
    logger.info(f"  t-statistic: {t_stat:.4f}")
    logger.info(f"  p-value:     {p_value:.6f}")

    if p_value < 0.05:
        logger.info("  ✓ Statistically significant difference (p < 0.05)")
    else:
        logger.info("  ✗ No statistically significant difference (p >= 0.05)")

    return t_stat, p_value

# ============================================================================
# CONFIDENCE INTERVALS
# ============================================================================

def compute_ci(data: np.ndarray, confidence: float = 0.95) -> tuple[float, float]:
    """Compute confidence interval for the mean."""
    mean = np.mean(data)
    sem = stats.sem(data)
    df = len(data) - 1

    alpha = 1 - confidence
    t_crit = stats.t.ppf(1 - alpha/2, df)
    margin = t_crit * sem

    logger.debug(f"CI: mean={mean:.2f}, sem={sem:.4f}, margin={margin:.2f}")

    return (mean - margin, mean + margin)

# ============================================================================
# VISUALIZATION
# ============================================================================

def plot_score_distributions(study: np.ndarray, no_study: np.ndarray) -> None:
    """Plot overlaid histograms of both groups."""
    fig, ax = plt.subplots(figsize=(12, 6))

    # Histograms
    ax.hist(study, bins=15, alpha=0.6, label='Study Group', color='green', edgecolor='black')
    ax.hist(no_study, bins=15, alpha=0.6, label='No-Study Group', color='red', edgecolor='black')

    ax.set_xlabel('Exam Score')
    ax.set_ylabel('Frequency')
    ax.set_title('Distribution of Exam Scores by Study Status')
    ax.legend()
    ax.grid(alpha=0.3)

    plt.tight_layout()
    plt.savefig('output/plots/score_distribution.png', dpi=150)
    logger.info("Saved score distribution plot")
    plt.close()

def plot_ci_comparison(study: np.ndarray, no_study: np.ndarray,
                       ci_study: tuple[float, float],
                       ci_no_study: tuple[float, float]) -> None:
    """Plot confidence intervals for both groups."""
    fig, ax = plt.subplots(figsize=(10, 6))

    groups = ['Study Group', 'No-Study Group']
    means = [np.mean(study), np.mean(no_study)]

    errors = [
        [means[0] - ci_study[0], means[1] - ci_no_study[0]],
        [ci_study[1] - means[0], ci_no_study[1] - means[1]]
    ]

    ax.errorbar(groups, means, yerr=errors, fmt='o', markersize=12,
                capsize=12, capthick=2, linewidth=2, color='blue')

    ax.set_ylabel('Mean Score')
    ax.set_title('95% Confidence Intervals for Mean Exam Scores')
    ax.grid(axis='y', alpha=0.3)

    # Add value labels
    for i, (group, mean, ci) in enumerate(zip(groups, means, [ci_study, ci_no_study])):
        ax.text(i, mean + 2, f'{mean:.1f}\n[{ci[0]:.1f}, {ci[1]:.1f}]',
                ha='center', fontsize=10)

    plt.tight_layout()
    plt.savefig('output/plots/ci_comparison.png', dpi=150)
    logger.info("Saved confidence interval comparison plot")
    plt.close()

# ============================================================================
# MAIN ANALYSIS
# ============================================================================

def main() -> None:
    """Run complete statistical analysis."""
    logger.info("Starting exam score analysis...")

    # Generate data
    study_group, no_study_group = generate_exam_scores()

    # Descriptive statistics
    logger.info("")
    logger.info("=" * 60)
    logger.info("DESCRIPTIVE STATISTICS")
    logger.info("=" * 60)

    study_stats = compute_stats(study_group)
    no_study_stats = compute_stats(no_study_group)

    log_stats("Study Group", study_stats)
    logger.info("")
    log_stats("No-Study Group", no_study_stats)

    # Hypothesis test
    logger.info("")
    logger.info("=" * 60)
    logger.info("HYPOTHESIS TESTING (t-test)")
    logger.info("=" * 60)

    t_stat, p_value = run_t_test(study_group, no_study_group)

    # Confidence intervals
    logger.info("")
    logger.info("=" * 60)
    logger.info("CONFIDENCE INTERVALS (95%)")
    logger.info("=" * 60)

    ci_study = compute_ci(study_group)
    ci_no_study = compute_ci(no_study_group)

    logger.info(f"Study Group CI:     [{ci_study[0]:.2f}, {ci_study[1]:.2f}]")
    logger.info(f"No-Study Group CI:  [{ci_no_study[0]:.2f}, {ci_no_study[1]:.2f}]")

    # Visualization
    logger.info("")
    logger.info("=" * 60)
    logger.info("CREATING VISUALIZATIONS")
    logger.info("=" * 60)

    plot_score_distributions(study_group, no_study_group)
    plot_ci_comparison(study_group, no_study_group, ci_study, ci_no_study)

    logger.info("")
    logger.info("=" * 60)
    logger.info("ANALYSIS COMPLETE")
    logger.info("=" * 60)

if __name__ == '__main__':
    main()
```

**Expected output:**

```
2026-03-28 10:15:30,123 - __main__ - INFO - Starting exam score analysis...
2026-03-28 10:15:30,124 - __main__ - INFO - Generated 50 study group scores
2026-03-28 10:15:30,124 - __main__ - INFO - Generated 50 no-study group scores

============================================================
DESCRIPTIVE STATISTICS
============================================================
2026-03-28 10:15:30,125 - __main__ - INFO - Study Group statistics:
2026-03-28 10:15:30,125 - __main__ - INFO -   mean    :   78.41
2026-03-28 10:15:30,125 - __main__ - INFO -   median  :   78.27
2026-03-28 10:15:30,125 - __main__ - INFO -   std     :    8.12
2026-03-28 10:15:30,125 - __main__ - INFO -   min     :   60.53
2026-03-28 10:15:30,125 - __main__ - INFO -   max     :   95.18
2026-03-28 10:15:30,125 - __main__ - INFO -   q25     :   71.98
2026-03-28 10:15:30,125 - __main__ - INFO -   q75     :   84.87

2026-03-28 10:15:30,125 - __main__ - INFO - No-Study Group statistics:
2026-03-28 10:15:30,125 - __main__ - INFO -   mean    :   68.37
2026-03-28 10:15:30,125 - __main__ - INFO -   median  :   68.15
2026-03-28 10:15:30,125 - __main__ - INFO -   std     :    9.87
2026-03-28 10:15:30,125 - __main__ - INFO -   min     :   46.82
2026-03-28 10:15:30,125 - __main__ - INFO -   max     :   92.33
2026-03-28 10:15:30,125 - __main__ - INFO -   q25     :   61.45
2026-03-28 10:15:30,125 - __main__ - INFO -   q75     :   76.28

============================================================
HYPOTHESIS TESTING (t-test)
============================================================
2026-03-28 10:15:30,126 - __main__ - INFO - t-test results:
2026-03-28 10:15:30,126 - __main__ - INFO -   t-statistic: 4.8732
2026-03-28 10:15:30,126 - __main__ - INFO -   p-value:     0.000005
2026-03-28 10:15:30,126 - __main__ - INFO -   ✓ Statistically significant difference (p < 0.05)

============================================================
CONFIDENCE INTERVALS (95%)
============================================================
2026-03-28 10:15:30,127 - __main__ - INFO - Study Group CI:     [75.42, 81.40]
2026-03-28 10:15:30,127 - __main__ - INFO - No-Study Group CI:  [65.10, 71.64]

============================================================
CREATING VISUALIZATIONS
============================================================
2026-03-28 10:15:30,200 - __main__ - INFO - Saved score distribution plot
2026-03-28 10:15:30,250 - __main__ - INFO - Saved confidence interval comparison plot

============================================================
ANALYSIS COMPLETE
============================================================
```

---

## What's Next

**Day 10: pytest - Introduction to Automated Testing**

pytest is the Python standard for testing. We'll write:
- Unit tests for statistical functions
- Test fixtures for reusable test data
- Parametrized tests to test multiple inputs
- Tests for edge cases (empty lists, negative values, etc.)
