---
layout: single
title: "Day 5 of 180 - Advanced Python - Comprehensions, Generators, Decorators & Modern Features"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---

## Why These Topics Matter for AI Engineering
 
When working with large datasets, every byte of memory and every millisecond of computation matters. Today's topics are the tools that separate production-grade Python from tutorial code.
 
**Generators** let you process gigabytes of data without loading it all into RAM. **List comprehensions** make data transformation readable and fast. **Decorators** let you add behavior (like caching or timing) without rewriting code. **Type hints** catch bugs before your code runs-critical when working in teams. **Dataclasses** eliminate boilerplate for data structures. **pathlib** handles file paths correctly on Windows, Mac, and Linux without painful string concatenation.
 
Together, these are the everyday tools of AI engineers processing datasets, building ML pipelines, and shipping production code.
 
---
 
## Setup
 
You need only Python's standard library for this lesson. Open a terminal and create a virtual environment:
 
```bash
python3 -m venv day5_env
source day5_env/bin/activate  # On Windows: day5_env\Scripts\activate
python --version  # Should be 3.9 or higher (3.10+ recommended for newer type hint syntax)
```
 
Create a working directory:
 
```bash
mkdir -p day5_project/{data_samples,analysis_output}
cd day5_project
```
 
---
 
## Part 1: List Comprehensions - Readable Data Transformation
 
### The Analogy
Imagine a factory assembly line where items pass through a filter gate. Some are rejected, the rest are transformed. List comprehensions do exactly that in one readable line.
 
### Basic Syntax
```python
[expression for item in iterable if condition]
```
 
The `if` is optional. Read it left to right: "Make a list of (expression) for each (item) in (iterable) if (condition)."
 
### Example 1: Simple Transformation
```python
# Without comprehension (verbose)
numbers = [1, 2, 3, 4, 5]
squared = []
for n in numbers:
    squared.append(n ** 2)
print(squared)  # [1, 4, 9, 16, 25]
 
# With comprehension (readable)
squared = [n ** 2 for n in numbers]
print(squared)  # [1, 4, 9, 16, 25]
```
 
### Example 2: Filtering
```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
 
# Filter: keep only even numbers
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4, 6, 8, 10]
 
# Filter and transform: keep even numbers, then square them
evens_squared = [n ** 2 for n in numbers if n % 2 == 0]
print(evens_squared)  # [4, 16, 36, 64, 100]
```
 
### Example 3: Dictionary Comprehensions
```python
# Create a dict mapping word to word length
words = ["python", "engineering", "data"]
word_lengths = {word: len(word) for word in words}
print(word_lengths)  # {'python': 6, 'engineering': 11, 'data': 4}
 
# Swap keys and values
original = {"a": 1, "b": 2, "c": 3}
swapped = {v: k for k, v in original.items()}
print(swapped)  # {1: 'a', 2: 'b', 3: 'c'}
```
 
### Example 4: Set Comprehensions
```python
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
 
# Remove duplicates and filter
unique_evens = {n for n in numbers if n % 2 == 0}
print(unique_evens)  # {2, 4}
```
 
### Example 5: Nested Comprehensions - When They Help
```python
# Create a 3x3 matrix (list of lists)
matrix = [[i * 3 + j for j in range(3)] for i in range(3)]
print(matrix)
# [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
 
# Flatten a nested list
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [item for sublist in nested for item in sublist]
print(flattened)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
 
### Example 6: Nested Comprehensions - When to Avoid
```python
# Too complex - hard to read. Use a regular loop or generator instead.
result = [
    x * y
    for x in range(10)
    for y in range(10)
    if (x + y) % 2 == 0 and x * y > 20
]
 
# Better: be explicit
result = []
for x in range(10):
    for y in range(10):
        if (x + y) % 2 == 0 and x * y > 20:
            result.append(x * y)
```
 
**Rule of thumb:** If it takes more than one second to read, use a regular loop. Comprehensions should be *readable*.
 
### Example 7: Generator Expressions (Lazy Evaluation)
```python
# List comprehension - evaluates immediately, stores all in memory
lazy_list = [n ** 2 for n in range(1_000_000)]  # Uses lots of memory
 
# Generator expression - evaluates on demand, one at a time
lazy_gen = (n ** 2 for n in range(1_000_000))  # Uses almost no memory
 
# They look identical except () vs []
# But generators are lazy - they don't compute until you ask
 
print(next(lazy_gen))  # 0 (first square)
print(next(lazy_gen))  # 1 (second square)
```
 
---
 
## Part 2: Generators - Processing Without Loading Everything
 
### The Analogy
A vending machine gives you one item at a time when you press the button, instead of handing you the entire inventory.
 
### Why Generators Matter
Processing a 100 GB dataset? Load it line by line with a generator instead of loading all 100 GB into RAM.
 
```python
import sys
 
# List: stores all in memory at once
list_squares = [n ** 2 for n in range(1000)]
print(f"List memory: {sys.getsizeof(list_squares)} bytes")  # ~9000+ bytes
 
# Generator: computes one at a time
def gen_squares(n):
    for i in range(n):
        yield i ** 2
 
gen_squares_obj = gen_squares(1000)
print(f"Generator memory: {sys.getsizeof(gen_squares_obj)} bytes")  # ~128 bytes
```
 
### Example 1: Simple Generator with `yield`
```python
def count_up(max_num: int):
    """Generator that yields numbers from 0 to max_num."""
    current = 0
    while current < max_num:
        yield current
        current += 1
 
# Use it
for num in count_up(5):
    print(num)
# Output: 0 1 2 3 4
```
 
### Example 2: Reading Large Files Line by Line
```python
def read_large_file(file_path: str):
    """Generator that reads file line by line without loading all into memory."""
    with open(file_path, "r") as f:
        for line in f:
            yield line.strip()
 
# Use it (simulated with a string)
# In real use: for line in read_large_file("huge_file.txt"):
#     process(line)
```
 
### Example 3: `yield from` - Delegating to Another Generator
```python
def gen_a():
    """First generator."""
    yield 1
    yield 2
 
def gen_b():
    """Second generator."""
    yield 3
    yield 4
 
def gen_combined():
    """Combine both generators."""
    yield from gen_a()
    yield from gen_b()
 
for value in gen_combined():
    print(value)
# Output: 1 2 3 4
```
 
### Example 4: Manual Iteration with `next()`
```python
def countdown(n: int):
    """Countdown generator."""
    while n > 0:
        yield n
        n -= 1
 
counter = countdown(3)
print(next(counter))  # 3
print(next(counter))  # 2
print(next(counter))  # 1
# print(next(counter))  # Would raise StopIteration
```
 
### Example 5: Infinite Generator
```python
def infinite_counter(start: int = 0):
    """Infinite generator - keeps going forever."""
    current = start
    while True:
        yield current
        current += 1
 
counter = infinite_counter(10)
print(next(counter))  # 10
print(next(counter))  # 11
print(next(counter))  # 12
# Keep calling next() - it never stops
```
 
### Example 6: Generator with Conditional Yield
```python
def fibonacci(limit: int):
    """Generate Fibonacci numbers up to limit."""
    a, b = 0, 1
    while a < limit:
        yield a
        a, b = b, a + b
 
for fib in fibonacci(100):
    print(fib, end=" ")
# Output: 0 1 1 2 3 5 8 13 21 34 55 89
```
 
---
 
## Part 3: Decorators - Adding Behavior Without Rewriting Code
 
### The Analogy
A gift wrapper takes your present and wraps it in paper and ribbon, then hands back the wrapped version. The present is unchanged, but it's now decorated. Decorators do the same for functions.
 
### Example 1: Write a Decorator from Scratch
```python
import time
from typing import Callable, Any
 
def timer(func: Callable) -> Callable:
    """Decorator that measures how long a function takes."""
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper
 
# Use it
@timer
def slow_function():
    time.sleep(1)
    return "Done!"
 
result = slow_function()
# Output: slow_function took 1.0001 seconds
# Done!
```
 
### Problem: Decorators Lose Function Metadata
```python
def simple_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
 
@simple_decorator
def greet(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"
 
print(greet.__name__)  # 'wrapper' - WRONG! Should be 'greet'
print(greet.__doc__)   # None - WRONG! Should be the docstring
```
 
### Example 2: Fix with `functools.wraps`
```python
import functools
from typing import Callable, Any
 
def timer(func: Callable) -> Callable:
    """Decorator that measures function execution time."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        import time
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper
 
@timer
def greet(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"
 
print(greet.__name__)  # 'greet' - CORRECT
print(greet.__doc__)   # 'Say hello to someone.' - CORRECT
```
 
### Example 3: Decorator with Arguments (Decorator Factory)
```python
import functools
from typing import Callable, Any
 
def repeat(times: int) -> Callable:
    """Decorator factory that repeats a function N times."""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> None:
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator
 
@repeat(times=3)
def say_hello():
    print("Hello!")
 
say_hello()
# Output:
# Hello!
# Hello!
# Hello!
```
 
### Example 4: Stacking Decorators
```python
import functools
import time
from typing import Callable, Any
 
def timer(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"[TIMER] {func.__name__}: {end - start:.4f}s")
        return result
    return wrapper
 
def log_call(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f"[LOG] Calling {func.__name__} with args={args}, kwargs={kwargs}")
        return func(*args, **kwargs)
    return wrapper
 
# Stack decorators - applied bottom to top
@timer
@log_call
def add(a: int, b: int) -> int:
    time.sleep(0.1)
    return a + b
 
result = add(2, 3)
# Output:
# [LOG] Calling add with args=(2, 3), kwargs={}
# [TIMER] add: 0.1005s
# 5
```
 
### Example 5: Caching Decorator (Memoization)
```python
import functools
from typing import Callable, Any
 
def cache(func: Callable) -> Callable:
    """Decorator that caches function results."""
    results = {}
 
    @functools.wraps(func)
    def wrapper(*args: Any) -> Any:
        if args not in results:
            results[args] = func(*args)
        return results[args]
    return wrapper
 
@cache
def expensive_computation(n: int) -> int:
    """Simulates a slow calculation."""
    import time
    time.sleep(1)
    return n ** 2
 
print(expensive_computation(5))  # Takes 1 second
print(expensive_computation(5))  # Instant (from cache)
```
 
---
 
## Part 4: Type Hints - Your First Production Standard
 
### Why Type Hints Matter
- **Catch bugs early:** IDEs warn you when you pass the wrong type
- **Better autocomplete:** Your editor knows what methods are available
- **Self-documenting:** Readers immediately know what types are expected
- **Easier refactoring:** Change a type signature, find all broken calls
 
Type hints are now a Day 5 production standard. From this point forward, **every function must have type hints on parameters and return type**.
 
### Example 1: Basic Types
```python
def greet(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}!"
 
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b
 
def is_positive(num: float) -> bool:
    """Check if a number is positive."""
    return num > 0
```
 
### Example 2: Collections
```python
def first_three(numbers: list[int]) -> list[int]:
    """Return the first three numbers."""
    return numbers[:3]
 
def count_words(text: str) -> dict[str, int]:
    """Count occurrences of each word."""
    words = text.split()
    return {word: words.count(word) for word in set(words)}
 
def get_coords() -> tuple[float, float]:
    """Return latitude and longitude."""
    return (40.7128, -74.0060)
 
def unique_tags(tags: list[str]) -> set[str]:
    """Return unique tags."""
    return set(tags)
```
 
### Example 3: Optional Types
```python
from typing import Optional
 
def find_user(user_id: int) -> Optional[str]:
    """Find a user by ID, or None if not found."""
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)
 
# Python 3.10+ allows this syntax
def find_user_modern(user_id: int) -> str | None:
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)
```
 
### Example 4: Union Types (Multiple Possible Types)
```python
from typing import Union
 
def process_data(data: Union[int, str]) -> str:
    """Accept either int or str."""
    return str(data).upper()
 
# Python 3.10+ allows this syntax
def process_data_modern(data: int | str) -> str:
    return str(data).upper()
```
 
### Example 5: Generic Types with TypeVar
```python
from typing import TypeVar
 
T = TypeVar('T')  # 'T' can be any type
 
def get_first(items: list[T]) -> T:
    """Get the first item from a list."""
    return items[0]
 
# Works with any type
first_num = get_first([1, 2, 3])      # int
first_str = get_first(["a", "b"])     # str
```
 
### Example 6: Callable Types (Function Types)
```python
from typing import Callable
 
def apply_twice(func: Callable[[int], int], value: int) -> int:
    """Apply a function twice to a value."""
    return func(func(value))
 
def square(n: int) -> int:
    return n ** 2
 
result = apply_twice(square, 3)  # 3 -> 9 -> 81
print(result)  # 81
```
 
### Example 7: Protocol for Duck Typing
```python
from typing import Protocol
 
class Drawable(Protocol):
    """Anything with a draw() method."""
    def draw(self) -> None:
        ...
 
class Circle:
    def draw(self) -> None:
        print("Drawing circle")
 
class Square:
    def draw(self) -> None:
        print("Drawing square")
 
def render(shape: Drawable) -> None:
    """Draw any shape."""
    shape.draw()
 
render(Circle())   # Works
render(Square())   # Works
```
 
### Example 8: TypedDict for Typed Dictionaries
```python
from typing import TypedDict
 
class Person(TypedDict):
    name: str
    age: int
    email: str
 
def greet_person(person: Person) -> str:
    return f"Hello {person['name']}, age {person['age']}"
 
# Dict with correct types
person = {"name": "Alice", "age": 30, "email": "alice@example.com"}
print(greet_person(person))
```
 
---
 
## Part 5: Dataclasses - Auto-Generating Data Structure Boilerplate
 
### The Analogy
A dataclass is like a form template. Instead of writing `__init__`, `__repr__`, and `__eq__` by hand, the decorator fills them in automatically.
 
### Example 1: Basic Dataclass
```python
from dataclasses import dataclass
 
@dataclass
class Person:
    name: str
    age: int
    email: str
 
# Automatically gets __init__, __repr__, __eq__
person = Person(name="Alice", age=30, email="alice@example.com")
print(person)  # Person(name='Alice', age=30, email='alice@example.com')
 
# Can compare directly
person2 = Person(name="Alice", age=30, email="alice@example.com")
print(person == person2)  # True
```
 
### Example 2: Default Values
```python
from dataclasses import dataclass
 
@dataclass
class Config:
    host: str
    port: int = 8000
    debug: bool = False
 
# Can omit fields with defaults
config = Config(host="localhost")
print(config)  # Config(host='localhost', port=8000, debug=False)
```
 
### Example 3: `field()` for Custom Defaults
```python
from dataclasses import dataclass, field
 
@dataclass
class Team:
    name: str
    members: list[str] = field(default_factory=list)
    scores: dict[str, int] = field(default_factory=dict)
 
team1 = Team(name="Alpha")
team2 = Team(name="Beta")
 
# Each team has its own list/dict (not shared!)
team1.members.append("Alice")
print(team1.members)  # ['Alice']
print(team2.members)  # []
```
 
### Example 4: Validation with `__post_init__`
```python
from dataclasses import dataclass
 
@dataclass
class Person:
    name: str
    age: int
 
    def __post_init__(self) -> None:
        """Validate after initialization."""
        if self.age < 0:
            raise ValueError(f"Age cannot be negative: {self.age}")
        if not self.name:
            raise ValueError("Name cannot be empty")
 
# This works
person = Person(name="Alice", age=30)
 
# This fails
try:
    bad_person = Person(name="Bob", age=-5)
except ValueError as e:
    print(f"Error: {e}")  # Error: Age cannot be negative: -5
```
 
### Example 5: Immutable Dataclass with `frozen=True`
```python
from dataclasses import dataclass
 
@dataclass(frozen=True)
class Point:
    x: float
    y: float
 
point = Point(x=10.0, y=20.0)
print(point)  # Point(x=10.0, y=20.0)
 
# Try to modify
try:
    point.x = 15.0
except Exception as e:
    print(f"Error: {type(e).__name__}")  # Error: FrozenInstanceError
```
 
---
 
## Part 6: pathlib - Working with File Paths Correctly
 
### Why pathlib Over os.path
- `os.path.join()` requires string concatenation: `os.path.join("folder", "file.txt")`
- `pathlib` uses the `/` operator: `Path("folder") / "file.txt"`
- Handles Windows, Mac, Linux paths automatically
- More readable, more Pythonic
 
### Example 1: Creating and Checking Paths
```python
from pathlib import Path
 
# Create a Path
file_path = Path("data") / "input.txt"  # Works on any OS
print(file_path)  # data/input.txt (or data\input.txt on Windows)
 
# Check if it exists
if file_path.exists():
    print("File exists!")
else:
    print("File does not exist")
 
# Check type
if file_path.is_file():
    print("It's a file")
elif file_path.is_dir():
    print("It's a directory")
```
 
### Example 2: Reading and Writing
```python
from pathlib import Path
 
# Write text
output_file = Path("output") / "result.txt"
output_file.parent.mkdir(parents=True, exist_ok=True)  # Create parent dir if needed
output_file.write_text("Hello, World!")
 
# Read text
content = output_file.read_text()
print(content)  # Hello, World!
 
# Append text (read, modify, write back)
content = output_file.read_text()
output_file.write_text(content + "\nNew line added")
```
 
### Example 3: Listing Files with `.glob()`
```python
from pathlib import Path
 
# Find all .txt files recursively
for txt_file in Path("data").glob("**/*.txt"):
    print(txt_file)
 
# Find all .py files in current directory (non-recursive)
for py_file in Path(".").glob("*.py"):
    print(py_file)
```
 
### Example 4: Iterating Directories with `.iterdir()`
```python
from pathlib import Path
 
# List everything in a directory
for item in Path("data").iterdir():
    if item.is_file():
        print(f"File: {item.name}")
    elif item.is_dir():
        print(f"Directory: {item.name}")
```
 
### Example 5: Working with Path Components
```python
from pathlib import Path
 
file_path = Path("data/analysis/results.txt")
 
print(file_path.name)        # results.txt
print(file_path.stem)        # results (filename without extension)
print(file_path.suffix)      # .txt (extension)
print(file_path.parent)      # data/analysis
print(file_path.parts)       # ('data', 'analysis', 'results.txt')
```
 
### Example 6: Resolving to Absolute Path
```python
from pathlib import Path
 
relative = Path("data/input.txt")
absolute = relative.resolve()
print(absolute)  # /full/path/to/data/input.txt
```
 
---
 
## The Project: File Analysis Tool
 
Let's build a **complete file analysis tool** that uses all six concepts together:
 
### Requirements
- Use **pathlib** to scan a directory
- Use a **generator** to lazily yield file statistics for each file
- Use **list comprehension** to filter by file extension
- Use **@dataclass** for the FileStats data structure
- Use **@timer decorator** to measure analysis time
- Use **type hints** on every function
 
### Step 1: Create the Main Script
 
Create `analyzer.py`:
 
```python
import time
import functools
from dataclasses import dataclass
from pathlib import Path
from typing import Generator, Callable, Any
 
# =============================================================================
# Data Structure: FileStats
# =============================================================================
 
@dataclass
class FileStats:
    """Statistics for a single file."""
    name: str
    path: Path
    size_bytes: int
    extension: str
 
    def __post_init__(self) -> None:
        """Validate after initialization."""
        if self.size_bytes < 0:
            raise ValueError(f"Size cannot be negative: {self.size_bytes}")
 
# =============================================================================
# Decorator: Timer
# =============================================================================
 
def timer(func: Callable) -> Callable:
    """Decorator that measures function execution time."""
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        elapsed = end - start
        print(f"\n[TIMER] {func.__name__} took {elapsed:.4f} seconds\n")
        return result
    return wrapper
 
# =============================================================================
# Generator: Analyze Files
# =============================================================================
 
def analyze_directory(directory: Path) -> Generator[FileStats, None, None]:
    """
    Generator that yields FileStats for each file in a directory.
    Lazy evaluation - doesn't load all files at once.
    """
    if not directory.is_dir():
        raise ValueError(f"Not a directory: {directory}")
 
    for file_path in directory.rglob("*"):
        if file_path.is_file():
            try:
                size = file_path.stat().st_size
                yield FileStats(
                    name=file_path.name,
                    path=file_path,
                    size_bytes=size,
                    extension=file_path.suffix,
                )
            except OSError:
                # Skip files we can't access
                pass
 
# =============================================================================
# Main Analysis Functions with Type Hints
# =============================================================================
 
def filter_by_extension(
    stats: Generator[FileStats, None, None],
    extension: str
) -> list[FileStats]:
    """
    Filter files by extension using list comprehension.
    Example: filter_by_extension(gen, ".py") returns only Python files.
    """
    return [
        stat for stat in stats
        if stat.extension.lower() == extension.lower()
    ]
 
def get_largest_files(
    stats: Generator[FileStats, None, None],
    top_n: int = 5
) -> list[FileStats]:
    """
    Return the N largest files, sorted by size descending.
    """
    all_stats = list(stats)
    return sorted(all_stats, key=lambda s: s.size_bytes, reverse=True)[:top_n]
 
def total_size_by_extension(
    stats: Generator[FileStats, None, None]
) -> dict[str, int]:
    """
    Use dict comprehension to sum sizes by extension.
    """
    all_stats = list(stats)
    return {
        ext: sum(s.size_bytes for s in all_stats if s.extension == ext)
        for ext in {s.extension for s in all_stats}
    }
 
def format_size(size_bytes: int) -> str:
    """Convert bytes to human-readable format."""
    for unit in ["B", "KB", "MB", "GB"]:
        if size_bytes < 1024:
            return f"{size_bytes:.1f} {unit}"
        size_bytes /= 1024
    return f"{size_bytes:.1f} TB"
 
# =============================================================================
# Main Entry Point
# =============================================================================
 
@timer
def main(directory: str) -> None:
    """
    Run full analysis with timing.
    """
    target_dir = Path(directory)
 
    if not target_dir.exists():
        print(f"Error: Directory does not exist: {target_dir}")
        return
 
    print(f"Analyzing directory: {target_dir}\n")
 
    # Example 1: All Python files
    print("=" * 70)
    print("PYTHON FILES")
    print("=" * 70)
    py_files = filter_by_extension(analyze_directory(target_dir), ".py")
    for stat in py_files:
        print(f"  {stat.name:<40} {format_size(stat.size_bytes):>10}")
 
    # Example 2: Largest files
    print("\n" + "=" * 70)
    print("TOP 5 LARGEST FILES")
    print("=" * 70)
    largest = get_largest_files(analyze_directory(target_dir), top_n=5)
    for stat in largest:
        print(f"  {stat.name:<40} {format_size(stat.size_bytes):>10}")
 
    # Example 3: Size by extension
    print("\n" + "=" * 70)
    print("SIZE BY FILE EXTENSION")
    print("=" * 70)
    sizes = total_size_by_extension(analyze_directory(target_dir))
    for ext, size in sorted(sizes.items(), key=lambda x: x[1], reverse=True):
        ext_label = ext if ext else "[no extension]"
        print(f"  {ext_label:<40} {format_size(size):>10}")
 
if __name__ == "__main__":
    # Analyze the current directory
    main(".")
```
 
### Step 2: Create Sample Data
 
Create `create_sample_data.py` to generate test files:
 
```python
from pathlib import Path
 
def create_sample_structure() -> None:
    """Create sample directory structure for testing."""
    base = Path("sample_data")
    base.mkdir(exist_ok=True)
 
    # Create subdirectories
    (base / "python_scripts").mkdir(exist_ok=True)
    (base / "config").mkdir(exist_ok=True)
    (base / "docs").mkdir(exist_ok=True)
 
    # Create sample files
    (base / "python_scripts" / "main.py").write_text(
        "print('Hello World')\n" * 100
    )
    (base / "python_scripts" / "utils.py").write_text(
        "def helper():\n    pass\n" * 50
    )
    (base / "config" / "settings.json").write_text(
        '{"debug": true, "port": 8000}\n' * 30
    )
    (base / "config" / "database.ini").write_text(
        "[database]\nhost=localhost\nport=5432\n" * 40
    )
    (base / "docs" / "README.md").write_text(
        "# Project Documentation\n\nThis is a sample file.\n" * 60
    )
    (base / "notes.txt").write_text(
        "Random notes and ideas\n" * 25
    )
 
    print(f"Sample data created in: {base}")
 
if __name__ == "__main__":
    create_sample_structure()
```
 
### Step 3: Run the Analysis
 
```bash
# Create sample data
python create_sample_data.py
# Output: Sample data created in: sample_data
 
# Analyze the sample data
python analyzer.py sample_data
```
 
### Expected Output
 
```
Analyzing directory: sample_data
 
======================================================================
PYTHON FILES
======================================================================
  main.py                                    1.2 KB
  utils.py                                   0.6 KB
 
======================================================================
TOP 5 LARGEST FILES
======================================================================
  README.md                                  3.5 KB
  settings.json                              1.6 KB
  database.ini                               2.1 KB
  main.py                                    1.2 KB
  utils.txt                                  0.6 KB
 
======================================================================
SIZE BY FILE EXTENSION
======================================================================
  .md                                        3.5 KB
  .ini                                       2.1 KB
  .json                                      1.6 KB
  .py                                        1.8 KB
  [no extension]                             0.6 KB
 
[TIMER] main took 0.0024 seconds
```
 
---
 
## What's Next
 
**Day 6** introduces **NumPy and Pandas**, Python's powerhouses for numerical computing and data analysis. 
 
---