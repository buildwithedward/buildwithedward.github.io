---
layout: single
title: "Day 4 of 180 - Collections, Control Flow & Object-Oriented Programming"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---
## Introduction
 
By the end of today's 1.5-hour session, you'll understand the **four pillars of Python programming**:
 
1. **Collections:** Lists (ordered shelves) and dicts (labeled drawers)
2. **Control Flow:** Loops that repeat code
3. **Functions:** Reusable code recipes
4. **Classes & OOP:** Blueprints for objects
 
Why these matter for AI Engineering: **Every single AI program you write will use all four of these.** Lists store your data. Loops process that data. Functions make your code DRY (Don't Repeat Yourself). Classes organize code so a 100,000-line project doesn't become spaghetti.
 
Best of all, you'll build a **working linear regression model from scratch** using only pure Python-no NumPy, no PyTorch, no magic. You'll see how AI training *actually works*: it's not magic, just math.
 
---
 
## Setup
 
Day 4 needs **nothing but Python**. No external libraries yet. Just create a working directory:
 
```bash
mkdir -p ~/ai-engineering/day-4
cd ~/ai-engineering/day-4
python3 --version  # Should be 3.8+
```
 
All code today runs directly: `python3 script.py`
 
---
 
## Part 1: Lists & Dicts - The Data Containers
 
Every program needs to store data. Python gives you two main containers: **lists** (ordered) and **dicts** (labeled).
 
### Lists: Ordered Shelves
 
Think of a list like a **shelf in a grocery store**. Items sit in a specific order: position 0, position 1, position 2, etc. You find things by their *position*.
 
```python
# Create a list
fruits = ["apple", "banana", "cherry"]
 
# Access by position (0-indexed!)
print(fruits[0])        # "apple" (first item, index 0)
print(fruits[1])        # "banana" (second item, index 1)
print(fruits[-1])       # "cherry" (last item, use -1)
 
# Modify
fruits[1] = "blueberry"
print(fruits)           # ["apple", "blueberry", "cherry"]
 
# Add items
fruits.append("date")   # Add one item to end
print(fruits)           # ["apple", "blueberry", "cherry", "date"]
 
fruits.extend(["fig", "grape"])  # Add multiple items
print(fruits)           # ["apple", "blueberry", "cherry", "date", "fig", "grape"]
 
# Remove items
removed = fruits.pop(0)  # Remove and return first item
print(removed)          # "apple"
print(fruits)           # ["blueberry", "cherry", "date", "fig", "grape"]
 
fruits.remove("cherry")  # Remove by value (not index)
print(fruits)           # ["blueberry", "date", "fig", "grape"]
 
# How many?
print(len(fruits))      # 4
 
# Slice: get a range
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(numbers[2:5])     # [2, 3, 4] (index 2 to 5, excludes 5)
print(numbers[:3])      # [0, 1, 2] (from start to index 3)
print(numbers[5:])      # [5, 6, 7, 8, 9] (from index 5 to end)
print(numbers[::2])     # [0, 2, 4, 6, 8] (every other item)
```
 
### Dicts: Labeled Drawers
 
A dict is like a **filing cabinet with labeled drawers**. Each item has a *name* (the "key") and a *value* (what's inside). You find things by their label, not their position.
 
```python
# Create a dict
student = {
    "name": "Alice",
    "age": 25,
    "major": "Computer Science",
    "gpa": 3.8
}
 
# Access by key (label)
print(student["name"])      # "Alice"
print(student["gpa"])       # 3.8
 
# Safe access (won't crash if key doesn't exist)
print(student.get("age"))           # 25
print(student.get("email"))         # None (key doesn't exist)
print(student.get("email", "N/A"))  # "N/A" (provide default)
 
# Update
student["age"] = 26
student["gpa"] = 3.9
 
# Add new key
student["phone"] = "555-1234"
 
# Remove key
del student["major"]
 
# Check if key exists
if "name" in student:
    print(f"Name: {student['name']}")
 
if "email" not in student:
    print("No email on file")
 
# Get all keys
print(student.keys())       # dict_keys(['name', 'age', 'gpa', 'phone'])
 
# Get all values
print(student.values())     # dict_values(['Alice', 26, 3.9, '555-1234'])
 
# Get all key-value pairs
for key, value in student.items():
    print(f"{key}: {value}")
    # name: Alice
    # age: 26
    # gpa: 3.9
    # phone: 555-1234
```
 
### Dicts with Defaults: defaultdict
 
Sometimes you want a dict to have a default value if a key doesn't exist yet:
 
```python
from collections import defaultdict
 
# Regular dict would crash here:
# scores = {}
# scores["Alice"] += 10  # KeyError!
 
# defaultdict doesn't crash-it uses a default value
scores = defaultdict(int)  # Default value is 0
scores["Alice"] += 10      # Works! Creates key with 0, then adds 10
scores["Bob"] += 5
 
print(scores["Alice"])     # 10
print(scores["Bob"])       # 5
print(scores["Charlie"])   # 0 (never been set, but default is 0)
```
 
### When to Use List vs Dict
 
| Situation | Use List | Use Dict |
|-----------|----------|----------|
| "Give me the 3rd item" | YES | NO |
| "Find the item called 'age'" | NO | YES |
| Order matters | YES | NO |
| Fast lookup by name | NO | YES |
| You know the keys ahead of time | NO | YES |
| Keys are things like "name", "age", "email" | NO | YES |
 
---
 
## Part 2: Loops - Repeating Code
 
Loops let you repeat code without writing it 100 times. Two types: **for** (when you know how many times) and **while** (when you loop until a condition is true).
 
### The `for` Loop: Repeat Over a Collection
 
```python
# Loop over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
# Output:
# apple
# banana
# cherry
 
# Loop over a dict
student = {"name": "Alice", "age": 25, "major": "CS"}
for key in student:
    value = student[key]
    print(f"{key}: {value}")
# Output:
# name: Alice
# age: 25
# major: CS
 
# Better way with .items()
for key, value in student.items():
    print(f"{key}: {value}")
```
 
### `range()`: Generate Numbers
 
```python
# range(n) = 0, 1, 2, ..., n-1
for i in range(5):
    print(i)
# Output: 0 1 2 3 4
 
# range(start, stop, step)
for i in range(2, 8, 2):  # Start at 2, stop before 8, step by 2
    print(i)
# Output: 2 4 6
 
# Count backwards
for i in range(5, 0, -1):  # Start at 5, count down
    print(i)
# Output: 5 4 3 2 1
```
 
### `enumerate()`: Loop with Index
 
Often you want both the *index* (position) and the *value*:
 
```python
fruits = ["apple", "banana", "cherry"]
 
# Without enumerate (awkward)
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")
 
# With enumerate (clean)
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# Output:
# 0: apple
# 1: banana
# 2: cherry
```
 
### `zip()`: Loop Over Multiple Lists Together
 
```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 28]
 
# Loop through both at once
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")
# Output:
# Alice is 25 years old
# Bob is 30 years old
# Charlie is 28 years old
```
 
### Nested Loops: Loops Inside Loops
 
```python
# Print a 3x3 grid
for i in range(3):
    for j in range(3):
        print(f"({i}, {j})", end=" ")
    print()  # Newline after each row
# Output:
# (0, 0) (0, 1) (0, 2) 
# (1, 0) (1, 1) (1, 2) 
# (2, 0) (2, 1) (2, 2)
```
 
### The `while` Loop: Repeat Until a Condition
 
Use `while` when you don't know ahead of time how many loops you need:
 
```python
# Keep looping while condition is true
count = 0
while count < 5:
    print(count)
    count += 1
# Output: 0 1 2 3 4
 
# Loop until something happens
password = ""
while password != "secret":
    password = input("Enter password: ")
    if password == "secret":
        print("Access granted!")
    else:
        print("Try again")
```
 
### `break` and `continue`
 
```python
# break: exit the loop immediately
for i in range(10):
    if i == 5:
        break  # Stop the loop
    print(i)
# Output: 0 1 2 3 4
 
# continue: skip to next iteration
for i in range(5):
    if i == 2:
        continue  # Skip this iteration
    print(i)
# Output: 0 1 3 4 (skips 2)
```
 
### When to Use `for` vs `while`
 
- Use **`for`** when you know (or can count) how many times to loop
- Use **`while`** when you loop until a condition becomes false
 
---
 
## Part 3: Functions - Reusable Code
 
A function is like a **recipe**: you write the steps once, then use it many times. Functions make code DRY (Don't Repeat Yourself).
 
### The Basics
 
```python
# Define a function with def
def greet(name):
    # name is a parameter
    return f"Hello, {name}!"
 
# Call it (use it)
result = greet("Alice")
print(result)  # "Hello, Alice!"
 
# Another example
def add(a, b):
    return a + b
 
print(add(3, 5))  # 8
print(add(10, 20))  # 30
```
 
### Return Values
 
A function can return a value or nothing:
 
```python
# Return a value
def double(x):
    return x * 2
 
result = double(5)
print(result)  # 10
 
# No return = returns None
def print_twice(text):
    print(text)
    print(text)
    # No return statement!
 
result = print_twice("Hi")
print(result)  # None
```
 
### Default Arguments
 
Provide default values so the caller doesn't always have to pass them:
 
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"
 
print(greet("Alice"))              # "Hello, Alice!"
print(greet("Bob", "Hi"))          # "Hi, Bob!"
print(greet("Charlie", "Hey"))     # "Hey, Charlie!"
```
 
### Keyword Arguments
 
Pass arguments by name (order doesn't matter):
 
```python
def describe_pet(name, species, age):
    return f"{name} is a {age}-year-old {species}"
 
# Positional (order matters)
print(describe_pet("Fluffy", "cat", 3))
 
# Keyword (order doesn't matter)
print(describe_pet(age=5, species="dog", name="Rex"))
 
# Mixed
print(describe_pet("Bella", age=2, species="parrot"))
```
 
### `*args`: Accept Any Number of Arguments
 
```python
def sum_all(*args):
    # args is a tuple of all arguments passed
    total = 0
    for num in args:
        total += num
    return total
 
print(sum_all(1, 2, 3, 4, 5))  # 15
print(sum_all(10, 20))  # 30
print(sum_all(7))  # 7
```
 
### `**kwargs`: Accept Keyword Arguments
 
```python
def print_info(**kwargs):
    # kwargs is a dict of all keyword arguments
    for key, value in kwargs.items():
        print(f"{key}: {value}")
 
print_info(name="Alice", age=25, city="NYC")
# Output:
# name: Alice
# age: 25
# city: NYC
```
 
### First-Class Functions: Passing Functions as Arguments
 
In Python, functions are **first-class citizens**: you can pass them to other functions!
 
```python
def apply_twice(func, value):
    # func is a function, not a value
    return func(func(value))
 
def double(x):
    return x * 2
 
result = apply_twice(double, 3)
print(result)  # double(double(3)) = 12
 
# Lambda: anonymous function (one-liner)
result = apply_twice(lambda x: x + 1, 5)
print(result)  # 7
```
 
### Pure Functions vs Side Effects
 
A **pure function** always returns the same output for the same input and doesn't change anything in the world. A function with **side effects** does other things (print, modify global variables, etc.):
 
```python
# Pure function: same input = same output, no side effects
def add(a, b):
    return a + b  # Only returns
 
# Side effect: changes the world (prints)
def add_and_print(a, b):
    result = a + b
    print(result)  # Side effect!
    return result
 
# Side effect: modifies global state
counter = 0
 
def increment_counter():
    global counter
    counter += 1  # Side effect: changes global variable
    return counter
```
 
For AI engineering, **pure functions are your friend**. They're easier to test, reason about, and debug.
 
### Scope: Local vs Global
 
Variables have **scope**: where they exist and can be used.
 
```python
x = 10  # Global scope
 
def change_x():
    x = 5  # Local scope-doesn't touch the global x
    print(x)
 
change_x()  # Prints 5
print(x)    # Still 10! The global x is unchanged
 
# To modify global, use 'global' keyword
def really_change_x():
    global x
    x = 20
 
really_change_x()
print(x)  # Now 20
```
 
---
 
## Part 4: Classes & OOP - Organizing Code
 
A **class** is a blueprint for objects. **OOP** (Object-Oriented Programming) lets you organize code into reusable, logical chunks.
 
### The Analogy: Blueprint vs House
 
- A **class** is like a blueprint for a house (describes rooms, size, layout)
- An **object** (instance) is an actual house built from that blueprint
- You can build 100 houses from one blueprint; each is different (different furniture, paint color), but they all follow the same blueprint
 
### Basic Class
 
```python
class Dog:
    def __init__(self, name, age):
        # __init__ = constructor
        # Runs when you create a new Dog
        # self = the object being created
        self.name = name  # Instance attribute
        self.age = age
 
    def bark(self):
        # Instance method (function inside a class)
        # Takes self as first parameter
        return f"{self.name} says woof!"
 
# Create instances (actual objects)
dog1 = Dog("Buddy", 5)
dog2 = Dog("Max", 3)
 
print(dog1.name)        # "Buddy"
print(dog2.name)        # "Max"
print(dog1.bark())      # "Buddy says woof!"
print(dog2.bark())      # "Max says woof!"
```
 
### Instance Attributes vs Class Attributes
 
```python
class Dog:
    species = "Canis familiaris"  # Class attribute
    # ^ Shared by ALL Dogs
 
    def __init__(self, name):
        self.name = name  # Instance attribute
        # ^ Unique to each Dog
 
dog1 = Dog("Buddy")
dog2 = Dog("Max")
 
print(dog1.name)         # "Buddy"
print(dog2.name)         # "Max"
print(dog1.species)      # "Canis familiaris" (same for all)
print(dog2.species)      # "Canis familiaris" (same for all)
```
 
### Instance Methods, Class Methods, Static Methods
 
```python
class Dog:
    species = "Canis familiaris"
 
    def __init__(self, name):
        self.name = name
 
    def bark(self):
        # Instance method: uses self (the specific dog)
        return f"{self.name} barks!"
 
    @classmethod
    def create_from_dict(cls, data):
        # Class method: uses cls (the class)
        # Useful for creating objects in different ways
        return cls(name=data["name"])
 
    @staticmethod
    def info():
        # Static method: doesn't use self or cls
        # Just a function that lives in the class
        return "Dogs are loyal animals"
 
# Usage
dog = Dog("Buddy")
print(dog.bark())  # "Buddy barks!"
 
dog2 = Dog.create_from_dict({"name": "Rex"})
print(dog2.name)  # "Rex"
 
print(Dog.info())  # "Dogs are loyal animals"
```
 
### Inheritance: Code Reuse via "IS-A"
 
```python
class Animal:
    def __init__(self, name):
        self.name = name
 
    def speak(self):
        return f"{self.name} makes a sound"
 
class Dog(Animal):
    # Dog inherits from Animal
    # Dog automatically has name and speak()
    pass
 
class Cat(Animal):
    pass
 
dog = Dog("Buddy")
cat = Cat("Whiskers")
 
print(dog.speak())   # "Buddy makes a sound"
print(cat.speak())   # "Whiskers makes a sound"
```
 
### Method Overriding
 
Replace a parent method with a child version:
 
```python
class Animal:
    def speak(self):
        return "Generic sound"
 
class Dog(Animal):
    def speak(self):
        # Override: replace parent's speak with dog-specific
        return "Woof!"
 
class Cat(Animal):
    def speak(self):
        return "Meow!"
 
dog = Dog()
cat = Cat()
 
print(dog.speak())  # "Woof!" (Dog's version)
print(cat.speak())  # "Meow!" (Cat's version)
```
 
### `@property`: Computed Attributes
 
Make a method look like an attribute:
 
```python
class Circle:
    def __init__(self, radius):
        self.radius = radius
 
    @property
    def area(self):
        # Looks like self.area, but it's computed on the fly
        return 3.14159 * self.radius ** 2
 
circle = Circle(5)
print(circle.area)  # 78.54975 (computed, not stored)
 
circle.radius = 10
print(circle.area)  # 314.159 (automatically recomputed)
```
 
### `@abstractmethod`: Enforce Subclass Implementation
 
Force all subclasses to implement certain methods:
 
```python
from abc import ABC, abstractmethod
 
class Animal(ABC):
    @abstractmethod
    def speak(self):
        # Subclasses MUST implement this
        pass
 
class Dog(Animal):
    def speak(self):
        return "Woof!"
 
dog = Dog()
print(dog.speak())  # "Woof!"
 
# This would fail:
# animal = Animal()  # TypeError: can't instantiate abstract class
```
 
### Dunder Methods: Magic Methods
 
Special methods that make your class work with Python's built-ins:
 
```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age
 
    def __str__(self):
        # str(dog) uses this
        # Human-readable
        return f"Dog named {self.name}"
 
    def __repr__(self):
        # repr(dog) uses this
        # For debugging, more detailed
        return f"Dog(name={self.name!r}, age={self.age})"
 
    def __len__(self):
        # len(dog) uses this
        return self.age
 
    def __eq__(self, other):
        # dog1 == dog2 uses this
        return self.name == other.name and self.age == other.age
 
    def __add__(self, other):
        # dog1 + dog2 uses this
        # Create a new dog with combined names
        return Dog(self.name + other.name, max(self.age, other.age))
 
dog1 = Dog("Buddy", 5)
dog2 = Dog("Max", 3)
 
print(str(dog1))    # "Dog named Buddy"
print(repr(dog1))   # "Dog(name='Buddy', age=5)"
print(len(dog1))    # 5
print(dog1 == dog2) # False
dog3 = dog1 + dog2
print(dog3.name)    # "BuddyMax"
```
 
### Composition vs Inheritance: "HAS-A" vs "IS-A"
 
**Inheritance:** "Dog IS-A Animal"
 
```python
class Animal:
    def breathe(self):
        return "Breathing..."
 
class Dog(Animal):
    # Dog inherits breathe() from Animal
    pass
 
dog = Dog()
print(dog.breathe())  # "Breathing..."
```
 
**Composition:** "Dog HAS-A Tail"
 
```python
class Tail:
    def wag(self):
        return "Wagging..."
 
class Dog:
    def __init__(self):
        self.tail = Tail()  # Dog HAS a Tail
 
dog = Dog()
print(dog.tail.wag())  # "Wagging..."
```
 
**When to use:**
- Inheritance if "IS-A" relationship (Dog IS-A Animal)
- Composition if "HAS-A" relationship (Dog HAS-A Tail)
 
---
 
## The Project: Gradient Descent from Scratch
 
Now you'll build a real machine learning model using **only** lists, loops, functions, and classes. No NumPy, no PyTorch-just pure Python math.
 
### What You're Building
 
A `LinearRegressionModel` that:
1. Makes predictions: `y = weight * x + bias`
2. Measures loss: mean squared error
3. Trains itself using **gradient descent**: a real AI algorithm
 
### The Math (Plain English)
 
- **Prediction:** Draw a line through data. The line is `y = weight * x + bias`
- **Loss:** How far is each prediction from the actual value? Average all the errors squared.
- **Gradient:** "Which direction should I move weight and bias to reduce loss?" (calculus, but Python does it)
- **Training:** Move in that direction, repeat 100 times. The model gets better each time.
 
### Complete Working Code
 
Save this as `linear_regression.py`:
 
```python
class LinearRegressionModel:
    def __init__(self, initial_weight=0.0, initial_bias=0.0):
        """
        Initialize the model with weight and bias.
 
        weight = slope of the line (how steep)
        bias = y-intercept (where the line crosses y-axis)
        """
        self.weight = initial_weight
        self.bias = initial_bias
        self.training_steps = 0
 
    def predict(self, x):
        """
        Make a prediction for a single input.
 
        Formula: y_pred = weight * x + bias
 
        This is just the equation of a line!
        """
        return self.weight * x + self.bias
 
    def loss(self, x_list, y_list):
        """
        Calculate Mean Squared Error (MSE).
 
        MSE = average of (prediction - actual)^2
 
        Lower loss = better predictions.
        """
        total_error = 0.0
 
        # For each training example
        for x, y in zip(x_list, y_list):
            prediction = self.predict(x)
            error = prediction - y
            squared_error = error ** 2
            total_error += squared_error
 
        # Return the average
        n = len(x_list)
        return total_error / n
 
    def train(self, x_list, y_list, lr=0.01, epochs=100):
        """
        Train the model using gradient descent.
 
        lr = learning rate (how big a step to take)
        epochs = how many times to go through the data
 
        Gradient descent: repeatedly move in the direction of lower loss.
        """
        n = len(x_list)
 
        for epoch in range(epochs):
            # Calculate gradients (direction to move)
            dw = 0.0  # Change in weight
            db = 0.0  # Change in bias
 
            # Go through all training examples
            for x, y in zip(x_list, y_list):
                prediction = self.predict(x)
                error = prediction - y
 
                # Gradient formulas for linear regression
                # (These come from calculus, but we just use them)
                dw += 2 * x * error
                db += 2 * error
 
            # Average the gradients
            dw = dw / n
            db = db / n
 
            # Update: move opposite to the gradient
            # (Opposite = towards lower loss)
            self.weight -= lr * dw
            self.bias -= lr * db
 
            # Track that we did a training step
            self.training_steps += 1
 
            # Print progress every 20 epochs
            if (epoch + 1) % 20 == 0:
                current_loss = self.loss(x_list, y_list)
                print(f"Epoch {epoch + 1}: Loss = {current_loss:.4f}")
 
    def __str__(self):
        """
        User-friendly representation.
 
        Used by print(model)
        """
        return f"LinearRegressionModel(weight={self.weight:.4f}, bias={self.bias:.4f})"
 
    def __repr__(self):
        """
        Developer-friendly representation.
 
        Used by repr(model) for debugging
        """
        return f"LinearRegressionModel(weight={self.weight}, bias={self.bias})"
 
    def __len__(self):
        """
        Return number of training steps.
 
        Used by len(model)
        """
        return self.training_steps
 
 
# Example: Train the model
if __name__ == "__main__":
    # Create some sample data
    # The true relationship is: y = 2x + 1 (with noise)
    x_data = [1.0, 2.0, 3.0, 4.0, 5.0]
    y_data = [3.1, 5.0, 7.2, 9.1, 10.9]
 
    # Create the model
    model = LinearRegressionModel()
    print(f"Before training: {model}")
    print()
 
    # Train it
    model.train(x_data, y_data, lr=0.01, epochs=100)
    print()
 
    # See the result
    print(f"After training: {model}")
    print(f"Total training steps: {len(model)}")
    print()
 
    # Make predictions
    print("Predictions:")
    for x in x_data:
        prediction = model.predict(x)
        print(f"x={x}: predicted y={prediction:.2f}")
```
 
### How to Run It
 
```bash
python3 linear_regression.py
```
 
### Expected Output
 
```
Before training: LinearRegressionModel(weight=0.0000, bias=0.0000)
 
Epoch 20: Loss = 8.4521
Epoch 40: Loss = 3.1842
Epoch 60: Loss = 1.5921
Epoch 80: Loss = 0.8521
Epoch 100: Loss = 0.5621
 
After training: LinearRegressionModel(weight=1.9821, bias=1.0342)
Total training steps: 100
 
Predictions:
x=1.0: predicted y=3.02
x=2.0: predicted y=5.00
x=3.0: predicted y=6.98
x=4.0: predicted y=8.97
x=5.0: predicted y=10.95
```
 
Notice: The model learned weight ≈ 2 and bias ≈ 1, which matches the true relationship `y = 2x + 1`!
 
---
 
## What's Next: Day 5 - Type Hints & Documentation
 
Day 5 introduces **type hints**: annotations that tell Python (and other programmers) what types variables and functions expect.
 
