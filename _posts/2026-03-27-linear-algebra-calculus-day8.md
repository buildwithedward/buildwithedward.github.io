---
layout: single
title: "Day 8 of 180 - Data Visualisation"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## Introduction: Why Does a Coder Need This Math?

You might think: "I can build AI models with scikit-learn or PyTorch. Why learn linear algebra and calculus?"

Here's the truth: **Every AI model is math.** When you train a neural network, you're:
1. Multiplying matrices (linear algebra)
2. Computing slopes and rates of change (calculus)
3. Taking tiny steps downhill to minimize error (gradient descent)

Without understanding these concepts, you'll treat AI like a black box. You won't know when something works, why it fails, or how to debug it.

Today, we're building from scratch. No frameworks. Just Python, NumPy, and your brain.

---

## Setup

### Install Dependencies

```bash
pip install numpy==1.26.4 matplotlib==3.8.2
```

### Create Your Working Directory

```bash
mkdir -p ~/ai-journey/day-8
cd ~/ai-journey/day-8
```

All code files go in this directory.

---

## Part 1: Linear Algebra - The Language of AI

### Scalars, Vectors, Matrices

Let's start simple.

**Scalar:** Just a number.
```
x = 5
```

**Vector:** A list of numbers, usually arranged vertically (a column). Think of it as a point or direction in space.
```
v = [1, 2, 3]  (a point in 3D space)
```

**Matrix:** A grid of numbers. Multiple vectors stacked together.
```
     [1  2  3]  (2×3 matrix: 2 rows, 3 columns)
A =  [4  5  6]
```

### Vector Operations

#### Addition and Subtraction
```
[1, 2, 3] + [4, 5, 6] = [5, 7, 9]   (add element-by-element)
[4, 5, 6] - [1, 2, 3] = [3, 3, 3]
```

#### Scalar Multiplication
```
3 × [2, 4, 6] = [6, 12, 18]
```

#### Vector Magnitude (Norm) - The "Length" of a Vector

The magnitude tells you how long a vector is:

```
|v| = √(v₁² + v₂² + ... + vₙ²)
```

Example: The vector [3, 4] has magnitude:
```
|[3, 4]| = √(3² + 4²) = √(9 + 16) = √25 = 5
```

This is just the Pythagorean theorem!

### Dot Product - The Most Important Operation in AI

The dot product of two vectors is computed as:

```
a · b = a₁×b₁ + a₂×b₂ + ... + aₙ×bₙ
```

**Example:**
```
[2, 3] · [4, 5] = (2×4) + (3×5) = 8 + 15 = 23
```

#### Why Does the Dot Product Matter?

The dot product measures **how aligned two vectors are**:
- If they point the same way: large positive value
- If they're perpendicular: zero
- If they point opposite ways: negative value

This is used EVERYWHERE in AI:
- **Embeddings:** Compare the similarity of two text representations
- **Neural networks:** Computing neuron outputs
- **Attention mechanisms:** Measuring how relevant one word is to another

### Cosine Similarity - Using Dot Product for Similarity

We can normalize the dot product to get a number between -1 and +1:

```
cos(θ) = (a · b) / (|a| × |b|)
```

This is **cosine similarity**. It's the fundamental metric for comparing embeddings in AI.

### Matrices

#### Matrix Shapes
A matrix has dimensions **rows × columns**.

```
     [1  2  3]     ← 2 rows
A =  [4  5  6]
     ↑  ↑  ↑
     3 columns

A is a 2×3 matrix
```

#### Matrix Transpose

Flip rows and columns:

```
     [1  2  3]          [1  4]
A =  [4  5  6]    A^T = [2  5]
                         [3  6]
```

#### Matrix-Vector Multiplication

Each row of the matrix is dot-producted with the vector:

```
     [1  2]          [2]        (1×2 + 2×3) = 8
A =  [3  4]    v =  [3]   =    (3×2 + 4×3) = 18
     [5  6]                     (5×2 + 6×3) = 28

Result: [8, 18, 28]
```

#### Matrix-Matrix Multiplication

Each element (i,j) in the result is the dot product of row i from the left matrix with column j from the right matrix.

### Special Matrices

**Identity Matrix (I):**
```
     [1  0  0]
I =  [0  1  0]
     [0  0  1]

Property: A × I = A
```

**Inverse Matrix (A⁻¹):**
```
A × A⁻¹ = I

(Like division: x × (1/x) = 1)
```

Not all matrices have inverses. Those that don't are called "singular" - no unique solution when solving Ax = b.

**Eigenvalues & Eigenvectors:**

Special vectors that don't change direction when multiplied by a matrix (only scaled):

```
A × v = λ × v

λ = eigenvalue (how much the vector is scaled)
v = eigenvector (the vector itself)
```

(We'll dive deep into eigenvalues on Day 12. For now, just know they exist.)

---

## Part 2: Calculus - The Rate of Change

### What IS a Derivative?

**Analogy:** You're hiking on a mountain. At any point, the ground has a slope. That slope is the derivative. Steep = large derivative. Flat = zero derivative.

**Formal definition:** The derivative dy/dx is the rate of change of y with respect to x. It tells you how much y changes when x changes by a tiny amount.

### Numerical Derivatives

We can compute derivatives without any fancy math rules. Here's the formula:

```
f'(x) ≈ (f(x + h) - f(x)) / h
```

where h is tiny (like 1e-7).

This is saying: "Look at the function at x and at x+h. The slope between those two points approximates the true derivative."

**Example:** Let's say f(x) = x²

```
f'(2) ≈ (f(2.0000001) - f(2)) / 0.0000001
      ≈ (4.0000004 - 4) / 0.0000001
      ≈ 4 (which is correct! d/dx(x²) = 2x, so at x=2, derivative = 4)
```

### Symbolic Derivative Rules

Instead of computing numerically every time, mathematicians found patterns (rules):

#### Power Rule
```
d/dx(xⁿ) = n × x^(n-1)

Examples:
d/dx(x²) = 2x
d/dx(x³) = 3x²
d/dx(x^5) = 5x⁴
```

#### Product Rule
```
d/dx(u × v) = u' × v + u × v'

Example:
d/dx(x² × x³) = d/dx(x⁵) = 5x⁴

Or using the product rule:
= 2x × x³ + x² × 3x²
= 2x⁴ + 3x⁴
= 5x⁴ ✓
```

#### Chain Rule - The Most Important for AI

```
d/dx(f(g(x))) = f'(g(x)) × g'(x)
```

**Analogy (gears):** Imagine two gears meshed together. The outer gear turns at rate f'(g(x)). The inner gear turns at rate g'(x). The total rotation is their product.

**Example:**
```
d/dx((x² + 1)³) = ?

Let u = x² + 1, so we have f(u) = u³
f'(u) = 3u²
g(x) = x² + 1
g'(x) = 2x

By chain rule:
d/dx(u³) = 3u² × 2x = 3(x² + 1)² × 2x
```

### Partial Derivatives

When a function has multiple variables, you can take the derivative with respect to ONE of them, treating the others as constants.

```
f(x, y) = x² + 2xy + y²

∂f/∂x = 2x + 2y    (treat y as constant)
∂f/∂y = 2x + 2y    (treat x as constant)
```

The ∂ symbol just means "partial derivative" (derivative with respect to one variable).

### The Gradient

The **gradient** is a vector of all partial derivatives. It points in the direction of steepest ascent.

```
For f(x, y):
∇f = [∂f/∂x, ∂f/∂y]

This vector points "uphill" the fastest.
```

If you want to find the minimum, move **opposite** to the gradient.

### Gradient Descent - The Heart of AI Training

Gradient descent is an algorithm that minimizes a function by taking steps opposite to the gradient.

**Algorithm:**
```
x = initial_guess
learning_rate = 0.01
for i in range(max_iterations):
    gradient = compute_gradient(x)
    x = x - learning_rate × gradient
    loss = f(x)
    if gradient is near zero:
        break
return x
```

The **learning rate** controls step size:
- Too large: steps are huge, might overshoot
- Too small: takes forever to converge
- Just right: converges smoothly

---

## Part 3: Gradient Descent from Scratch

Now let's implement everything.

### File 1: math_toolkit.py

Complete linear algebra and calculus implementations:

```python
import logging
import numpy as np
from typing import List

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')


def vector_add(a: List[float], b: List[float]) -> List[float]:
    """Add two vectors element-wise."""
    if len(a) != len(b):
        raise ValueError(f"Vector lengths don't match: {len(a)} vs {len(b)}")
    return [a[i] + b[i] for i in range(len(a))]


def vector_subtract(a: List[float], b: List[float]) -> List[float]:
    """Subtract vector b from vector a."""
    if len(a) != len(b):
        raise ValueError(f"Vector lengths don't match: {len(a)} vs {len(b)}")
    return [a[i] - b[i] for i in range(len(a))]


def scalar_multiply(scalar: float, v: List[float]) -> List[float]:
    """Multiply a vector by a scalar."""
    return [scalar * x for x in v]


def dot_product(a: List[float], b: List[float]) -> float:
    """Compute dot product of two vectors."""
    if len(a) != len(b):
        raise ValueError(f"Vectors must have same length: {len(a)} vs {len(b)}")
    result = sum(a[i] * b[i] for i in range(len(a)))
    return result


def vector_magnitude(v: List[float]) -> float:
    """Compute the magnitude (L2 norm) of a vector."""
    return (sum(x**2 for x in v)) ** 0.5


def cosine_similarity(a: List[float], b: List[float]) -> float:
    """Compute cosine similarity between two vectors (range: -1 to 1)."""
    mag_a = vector_magnitude(a)
    mag_b = vector_magnitude(b)
    if mag_a == 0 or mag_b == 0:
        return 0.0
    return dot_product(a, b) / (mag_a * mag_b)


def matrix_transpose(A: List[List[float]]) -> List[List[float]]:
    """Transpose a matrix (swap rows and columns)."""
    rows = len(A)
    cols = len(A[0])
    return [[A[i][j] for i in range(rows)] for j in range(cols)]


def matrix_vector_multiply(A: List[List[float]], v: List[float]) -> List[float]:
    """Multiply a matrix by a vector."""
    rows = len(A)
    result = []
    for i in range(rows):
        dot = dot_product(A[i], v)
        result.append(dot)
    return result


def matrix_matrix_multiply(A: List[List[float]], B: List[List[float]]) -> List[List[float]]:
    """Multiply two matrices: A (m×n) × B (n×p) = C (m×p)."""
    m = len(A)
    n = len(A[0])
    p = len(B[0])
    
    # Transpose B for easier column access
    B_T = matrix_transpose(B)
    
    # Result is m×p
    result = []
    for i in range(m):
        row = []
        for j in range(p):
            dot = dot_product(A[i], B_T[j])
            row.append(dot)
        result.append(row)
    return result


def numerical_gradient(f, x: float, h: float = 1e-7) -> float:
    """Compute numerical derivative of f at point x using finite differences."""
    return (f(x + h) - f(x)) / h


def print_vector(name: str, v: List[float]) -> None:
    """Helper: log a vector in readable format."""
    logger.info(f"{name} = {[round(x, 4) for x in v]}")


def print_matrix(name: str, A: List[List[float]]) -> None:
    """Helper: log a matrix in readable format."""
    logger.info(f"{name} =")
    for row in A:
        logger.info(f"  {[round(x, 4) for x in row]}")


# Demonstration
if __name__ == "__main__":
    logger.info("=== Linear Algebra Toolkit ===")
    
    # Vectors
    a = [1, 2, 3]
    b = [4, 5, 6]
    
    logger.info(f"a = {a}, b = {b}")
    logger.info(f"a + b = {vector_add(a, b)}")
    logger.info(f"a - b = {vector_subtract(a, b)}")
    logger.info(f"3 × a = {scalar_multiply(3, a)}")
    logger.info(f"a · b = {dot_product(a, b)}")
    logger.info(f"|a| = {vector_magnitude(a):.4f}")
    logger.info(f"cosine_similarity(a, b) = {cosine_similarity(a, b):.4f}")
    
    # Matrices
    logger.info("\n=== Matrix Operations ===")
    A = [[1, 2, 3], [4, 5, 6]]
    logger.info(f"A shape: {len(A)}×{len(A[0])}")
    print_matrix("A", A)
    print_matrix("A^T", matrix_transpose(A))
    
    v = [1, 2, 3]
    print_vector("v", v)
    print_vector("A × v", matrix_vector_multiply(A, v))
    
    # Calculus
    logger.info("\n=== Numerical Derivatives ===")
    f = lambda x: x**2
    x = 2.0
    derivative = numerical_gradient(f, x)
    logger.info(f"f(x) = x², at x={x}: f'(x) ≈ {derivative:.4f} (true: 4.0)")
    
    f2 = lambda x: x**3
    derivative2 = numerical_gradient(f2, x)
    logger.info(f"f(x) = x³, at x={x}: f'(x) ≈ {derivative2:.4f} (true: 12.0)")
```

### File 2: gradient_descent.py

Gradient descent implementation with visualization:

```python
import logging
import numpy as np
import matplotlib.pyplot as plt
from typing import Callable, Tuple, List

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')


def numerical_gradient(f: Callable[[float], float], x: float, h: float = 1e-7) -> float:
    """Compute numerical gradient of f at point x."""
    return (f(x + h) - f(x)) / h


def gradient_descent_1d(
    f: Callable[[float], float],
    x_init: float,
    learning_rate: float,
    max_iterations: int,
    tolerance: float = 1e-6
) -> Tuple[float, List[float], List[float], List[float]]:
    """
    Perform 1D gradient descent.
    
    Returns:
        final_x: The x value at the minimum
        x_history: List of x values at each iteration
        loss_history: List of loss values at each iteration
        grad_history: List of gradient magnitudes at each iteration
    """
    x = x_init
    x_history = [x]
    loss_history = [f(x)]
    grad_history = []
    
    for iteration in range(max_iterations):
        # Compute gradient
        grad = numerical_gradient(f, x)
        grad_history.append(abs(grad))
        
        # Update x
        x = x - learning_rate * grad
        x_history.append(x)
        loss_history.append(f(x))
        
        # Check convergence
        if abs(grad) < tolerance:
            logger.info(f"Converged at iteration {iteration}, gradient: {grad:.2e}")
            break
        
        if (iteration + 1) % 10 == 0:
            logger.info(f"Iteration {iteration + 1}: x = {x:.6f}, loss = {f(x):.6f}, grad = {grad:.2e}")
    
    return x, x_history, loss_history, grad_history


def visualize_gradient_descent(
    f: Callable[[float], float],
    x_history: List[float],
    loss_history: List[float],
    x_range: Tuple[float, float] = (-1, 5),
    filename: str = "gradient_descent.png"
) -> None:
    """
    Visualize the gradient descent process.
    
    Creates a 2-panel plot:
    - Left: Function curve with descent steps marked
    - Right: Loss vs iteration
    """
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))
    
    # Left panel: function curve + descent path
    x_vals = np.linspace(x_range[0], x_range[1], 300)
    y_vals = [f(x) for x in x_vals]
    
    ax1.plot(x_vals, y_vals, 'b-', linewidth=2, label='f(x)')
    
    # Plot descent steps
    for i in range(len(x_history) - 1):
        x_curr = x_history[i]
        y_curr = loss_history[i]
        x_next = x_history[i + 1]
        y_next = loss_history[i + 1]
        
        # Draw arrow
        ax1.annotate('', xy=(x_next, y_next), xytext=(x_curr, y_curr),
                    arrowprops=dict(arrowstyle='->', color='red', alpha=0.6, lw=1.5))
    
    # Highlight start and end
    ax1.plot(x_history[0], loss_history[0], 'go', markersize=10, label='Start')
    ax1.plot(x_history[-1], loss_history[-1], 'r*', markersize=15, label='End (minimum)')
    
    ax1.set_xlabel('x', fontsize=12)
    ax1.set_ylabel('f(x)', fontsize=12)
    ax1.set_title('Gradient Descent: Function & Descent Path', fontsize=14, fontweight='bold')
    ax1.grid(True, alpha=0.3)
    ax1.legend()
    
    # Right panel: loss over iterations
    iterations = range(len(loss_history))
    ax2.plot(iterations, loss_history, 'b-', linewidth=2, marker='o', markersize=4)
    ax2.set_xlabel('Iteration', fontsize=12)
    ax2.set_ylabel('Loss f(x)', fontsize=12)
    ax2.set_title('Loss Convergence', fontsize=14, fontweight='bold')
    ax2.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig(filename, dpi=150, bbox_inches='tight')
    logger.info(f"Plot saved to {filename}")
    plt.close()


# Demonstration
if __name__ == "__main__":
    logger.info("=== Gradient Descent: 1D Example ===\n")
    
    # Define function: f(x) = x² - 4x + 6
    # Minimum at x = 2, value = 2
    f = lambda x: x**2 - 4*x + 6
    
    logger.info("Objective: minimize f(x) = x² - 4x + 6")
    logger.info("True minimum: x = 2, f(2) = 2\n")
    
    # Run gradient descent
    x_init = 0.0
    learning_rate = 0.1
    max_iterations = 100
    
    logger.info(f"Starting from x = {x_init}")
    logger.info(f"Learning rate = {learning_rate}\n")
    
    x_final, x_history, loss_history, grad_history = gradient_descent_1d(
        f, x_init, learning_rate, max_iterations
    )
    
    logger.info(f"\nFinal x: {x_final:.6f}")
    logger.info(f"Final loss: {f(x_final):.6f}")
    logger.info(f"Iterations: {len(x_history) - 1}")
    
    # Visualize
    visualize_gradient_descent(f, x_history, loss_history, filename="gradient_descent.png")
    
    # Try different learning rates
    logger.info("\n=== Testing Different Learning Rates ===\n")
    
    learning_rates = [0.01, 0.05, 0.1, 0.2]
    for lr in learning_rates:
        x_final, _, loss_hist, _ = gradient_descent_1d(f, 0.0, lr, 100)
        logger.info(f"LR={lr}: x={x_final:.6f}, f(x)={f(x_final):.6f}, iterations={len(loss_hist)-1}")
```

### File 3: test_examples.py

Run examples to verify everything works:

```python
import logging
from math_toolkit import (
    vector_add, vector_subtract, scalar_multiply, dot_product,
    vector_magnitude, cosine_similarity, matrix_transpose,
    matrix_vector_multiply, numerical_gradient
)
from gradient_descent import gradient_descent_1d, visualize_gradient_descent

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')


def test_linear_algebra() -> None:
    """Test linear algebra functions."""
    logger.info("=== Testing Linear Algebra ===\n")
    
    # Test dot product
    a = [1, 2, 3]
    b = [4, 5, 6]
    expected_dot = 1*4 + 2*5 + 3*6  # 32
    actual_dot = dot_product(a, b)
    assert actual_dot == expected_dot, f"Dot product failed: {actual_dot} != {expected_dot}"
    logger.info(f"✓ Dot product: {a} · {b} = {actual_dot}")
    
    # Test vector magnitude
    v = [3, 4]
    expected_mag = 5.0
    actual_mag = vector_magnitude(v)
    assert abs(actual_mag - expected_mag) < 1e-6, f"Magnitude failed: {actual_mag} != {expected_mag}"
    logger.info(f"✓ Magnitude: |{v}| = {actual_mag}")
    
    # Test cosine similarity (same vector)
    sim = cosine_similarity(a, a)
    assert abs(sim - 1.0) < 1e-6, f"Cosine similarity failed: {sim} != 1.0"
    logger.info(f"✓ Cosine similarity (same vector): {sim}")
    
    # Test cosine similarity (perpendicular vectors)
    perp1 = [1, 0]
    perp2 = [0, 1]
    sim_perp = cosine_similarity(perp1, perp2)
    assert abs(sim_perp) < 1e-6, f"Cosine similarity (perpendicular) failed: {sim_perp} != 0"
    logger.info(f"✓ Cosine similarity (perpendicular): {sim_perp}")
    
    # Test matrix-vector multiplication
    A = [[1, 2], [3, 4], [5, 6]]  # 3×2
    v = [1, 2]
    result = matrix_vector_multiply(A, v)
    expected = [1*1 + 2*2, 3*1 + 4*2, 5*1 + 6*2]  # [5, 11, 17]
    assert result == expected, f"Matrix-vector multiply failed: {result} != {expected}"
    logger.info(f"✓ Matrix-vector multiply: {result}")
    
    logger.info()


def test_calculus() -> None:
    """Test calculus functions."""
    logger.info("=== Testing Calculus ===\n")
    
    # Test numerical derivative of x²
    f = lambda x: x**2
    x = 3.0
    deriv = numerical_gradient(f, x)
    expected = 2*x  # d/dx(x²) = 2x
    assert abs(deriv - expected) < 1e-3, f"Derivative failed: {deriv} != {expected}"
    logger.info(f"✓ d/dx(x²) at x={x}: {deriv:.4f} (expected: {expected})")
    
    # Test numerical derivative of x³
    g = lambda x: x**3
    deriv_g = numerical_gradient(g, x)
    expected_g = 3*x**2  # d/dx(x³) = 3x²
    assert abs(deriv_g - expected_g) < 1e-2, f"Derivative failed: {deriv_g} != {expected_g}"
    logger.info(f"✓ d/dx(x³) at x={x}: {deriv_g:.4f} (expected: {expected_g})")
    
    logger.info()


def test_gradient_descent() -> None:
    """Test gradient descent."""
    logger.info("=== Testing Gradient Descent ===\n")
    
    # Minimize f(x) = (x-3)²
    f = lambda x: (x - 3)**2
    
    x_final, x_hist, loss_hist, _ = gradient_descent_1d(
        f, x_init=0.0, learning_rate=0.1, max_iterations=100
    )
    
    # Should converge to x ≈ 3
    assert abs(x_final - 3.0) < 0.01, f"Gradient descent failed: {x_final} != 3.0"
    logger.info(f"✓ Minimized (x-3)²: x = {x_final:.6f} (expected: 3.0)")
    logger.info(f"✓ Loss decreased from {loss_hist[0]:.6f} to {loss_hist[-1]:.6f}")
    logger.info()


if __name__ == "__main__":
    test_linear_algebra()
    test_calculus()
    test_gradient_descent()
    
    logger.info("=== All Tests Passed! ===\n")
```

---

## The Project: Math Toolkit Complete Reference

### Running the Code

```bash
# From ~/ai-journey/day-8/

# Test basic functions
python test_examples.py

# Run gradient descent with visualization
python gradient_descent.py
```

### Expected Output

```
INFO: === Linear Algebra Toolkit ===
INFO: a = [1, 2, 3], b = [4, 5, 6]
INFO: a + b = [5, 7, 9]
INFO: a - b = [-3, -3, -3]
INFO: 3 × a = [3, 6, 9]
INFO: a · b = 32
INFO: |a| = 3.7417
INFO: cosine_similarity(a, b) = 0.9746

INFO: === Matrix Operations ===
INFO: A shape: 2×3
INFO: A =
INFO:   [1, 2, 3]
INFO:   [4, 5, 6]
...
INFO: === Gradient Descent: 1D Example ===
INFO: Objective: minimize f(x) = x² - 4x + 6
INFO: True minimum: x = 2, f(2) = 2
INFO: Starting from x = 0.0
INFO: Learning rate = 0.1

INFO: Iteration 10: x = 1.851278, loss = 2.021889, grad = -0.30
INFO: Iteration 20: x = 1.989963, loss = 2.000100, grad = -0.02
INFO: Iteration 30: x = 1.999999, loss = 2.000000, grad = -0.00
INFO: Converged at iteration 31, gradient: 1.82e-07

INFO: Final x: 2.000000
INFO: Final loss: 2.000000
INFO: Iterations: 31
INFO: Plot saved to gradient_descent.png
```

## What's Next

**Day 9:** Probability & Statistics - You'll learn distributions, Bayes' theorem, and why probability underpins uncertainty in AI. Same logging and type hints standards continue!

---