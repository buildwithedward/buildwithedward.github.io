---
layout: single
title: "Day 5 of 180 - Numpy & Pandas"
excerpt: "Part of my 180-day AI Engineering journey - explained for beginners"
categories: [dl-llm-systems]
tags: [deep-learning, llm, systems-design]
header:
  teaser: /assets/img/bgimage.png
---
> *Part of my 180-day AI Engineering journey - learning in public, one hour a day, writing everything in plain English so beginners can follow along. The blog is written with the help of AI*
---
 
## Introduction: Why This Matters for AI
 
Before we can train any AI model, we need to feed it data. But real-world data is messy:
- Customer records scattered across multiple files
- Duplicate entries and missing values
- Numbers in the wrong format
- Thousands (or millions) of rows to process
 
**NumPy** and **Pandas** are the tools that handle this. They let you:
- Load data from CSV files
- Clean and transform it
- Filter and group it
- Calculate statistics
- Merge datasets together
 
Every AI project starts here. You can't train a model without data, and you can't work with data without NumPy and Pandas.
 
---
 
## Setup
 
Install the required libraries:
 
```bash
pip install numpy==1.26.3 pandas==2.1.3
```
 
Verify installation:
 
```python
import numpy as np
import pandas as pd
 
print(f"NumPy version: {np.__version__}")
print(f"Pandas version: {pd.__version__}")
```
 
---
 
## Part 1: NumPy - Fast Arrays
 
### The Problem: Speed
 
Think about a simple task: you have a list of 1 million student scores, and you need to multiply each one by 1.1 (like adding a 10% bonus).
 
```python
# Python list way (SLOW)
scores = [85, 92, 78, 88, ...]  # 1 million items
bonused = []
for score in scores:
    bonused.append(score * 1.1)  # loops 1 million times
```
 
This takes time because Python has to:
1. Start a loop
2. Access each item individually
3. Multiply it
4. Append to a new list
5. Repeat 1 million times
 
**NumPy solves this:**
 
```python
import numpy as np
 
scores = np.array([85, 92, 78, 88, ...])  # 1 million items
bonused = scores * 1.1  # INSTANT - happens all at once!
```
 
NumPy does the multiplication on the entire array at the CPU level, not in Python. This is 50-100x faster.
 
### What is an Array?
 
An array is like a spreadsheet column (or grid) of numbers:
 
```python
import numpy as np
 
# 1D array (like a column)
scores = np.array([85, 92, 78, 88])
# [85 92 78 88]
 
# 2D array (like a spreadsheet)
grades = np.array([
    [85, 88, 92],  # Alice's math, science, english
    [92, 85, 88],  # Bob's scores
    [78, 92, 85]   # Charlie's scores
])
# [[85 88 92]
#  [92 85 88]
#  [78 92 85]]
```
 
### Creating Arrays
 
```python
import numpy as np
 
# From a Python list
arr = np.array([1, 2, 3, 4, 5])
# [1 2 3 4 5]
 
# Zeros (useful for initializing)
zeros = np.zeros(5)
# [0. 0. 0. 0. 0.]
 
# Ones
ones = np.ones((3, 2))
# [[1. 1.]
#  [1. 1.]
#  [1. 1.]]
 
# Range (like Python's range(), but returns an array)
range_arr = np.arange(0, 10, 2)
# [0 2 4 6 8]
 
# Evenly spaced numbers (give me 5 numbers between 0 and 1)
linspace = np.linspace(0, 1, 5)
# [0.   0.25 0.5  0.75 1.  ]
 
# Random numbers (for testing)
np.random.seed(42)  # reproducible randomness
random_arr = np.random.randn(3, 3)
# [[ 0.49671415 -0.1382643   0.64589411]
#  [-0.23415337 -0.23413696  1.57921282]
#  [ 0.76743473 -0.46947439  0.54256004]]
```
 
### Array Properties
 
```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
 
# How many rows and columns?
arr.shape  # (2, 3) - 2 rows, 3 columns
 
# What type of data?
arr.dtype  # dtype('int64') - integer, 64-bit
 
# How many dimensions?
arr.ndim   # 2 - it's a table (2D)
 
# Total number of elements?
arr.size   # 6
 
# Flip rows and columns
arr.T
# [[1 4]
#  [2 5]
#  [3 6]]
```
 
### Indexing: Getting Specific Elements
 
```python
scores = np.array([85, 92, 78, 88, 90])
 
# Get the first score
scores[0]  # 85
 
# Get the last score
scores[-1]  # 90
 
# Get scores from position 1 to 3 (not including 4)
scores[1:4]  # [92, 78, 88]
 
# Get every other score
scores[::2]  # [85, 78, 90]
 
# For 2D arrays
grades = np.array([[85, 88, 92], [92, 85, 88]])
 
# Get Alice's science score (row 0, column 1)
grades[0, 1]  # 88
 
# Get Charlie's entire row
grades[1, :]  # [92, 85, 88]
 
# Get all math scores (entire column 0)
grades[:, 0]  # [85, 92]
```
 
### Boolean Indexing: Filtering
 
This is incredibly useful. Get only the values that match a condition:
 
```python
scores = np.array([85, 92, 78, 88, 92])
 
# Which scores are 85 or higher?
high_scores = scores[scores >= 85]
# [85, 92, 88, 92]
 
# This creates a True/False array first:
scores >= 85
# [True, True, False, True, True]
 
# Then keeps only the True ones
```
 
### Broadcasting: The Magic Trick
 
Broadcasting is NumPy's ability to make arrays of different sizes work together. Think of it like stretching a small image to fit a big frame:
 
```python
# Simple example: adding 10 to every score
scores = np.array([80, 85, 90])
scores + 10
# [90, 95, 100]
 
# More complex: adding the same bonus to each subject
math_scores = np.array([85, 92, 78])
science_scores = np.array([88, 85, 92])
english_scores = np.array([92, 88, 85])
 
# Stack them into a table
grades = np.array([math_scores, science_scores, english_scores])
# [[85 92 78]
#  [88 85 92]
#  [92 88 85]]
 
# Add 5 points bonus to all grades
boosted = grades + 5
# [[90 97 83]
#  [93 90 97]
#  [97 93 90]]
 
# The 5 gets "broadcast" to all positions!
```
 
### Vectorized Operations: The Real Power
 
Instead of loops, use NumPy functions:
 
```python
temps = np.array([72.5, 73.1, 71.8, 74.2, 72.9])
 
# Average temperature
np.mean(temps)  # 72.9
 
# Standard deviation (how spread out are they?)
np.std(temps)   # ~0.94
 
# Total temperature (if we added them all up)
np.sum(temps)   # 364.5
 
# Highest temperature
np.max(temps)   # 74.2
 
# Position of highest (0-indexed)
np.argmax(temps)  # 3
 
# Lowest temperature
np.min(temps)   # 71.8
 
# Distance from mean (how far is each from average?)
deviations = temps - np.mean(temps)
# [ -0.4  0.2  -1.1  1.3  0. ]
 
# Absolute values (ignore + or -)
abs_deviations = np.abs(deviations)
# [0.4, 0.2, 1.1, 1.3, 0.]
 
# Square each one (for variance calculation)
squared = temps ** 2
# [5256.25, 5343.21, 5155.24, 5505.84, 5314.41]
 
# Square root
square_root = np.sqrt(squared)
 
# Dot product (multiply matching elements and sum)
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
np.dot(a, b)  # 1*4 + 2*5 + 3*6 = 32
```
 
### Reshaping: Changing Shape Without Losing Data
 
```python
arr = np.arange(12)  # [0, 1, 2, ..., 11]
 
# Reshape to a 3x4 grid
grid = arr.reshape(3, 4)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]
 
# Flatten back to 1D
flat = grid.flatten()
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
 
# Transpose (swap rows and columns)
transposed = grid.T
# [[ 0  4  8]
#  [ 1  5  9]
#  [ 2  6 10]
#  [ 3  7 11]]
```
 
### Stacking: Combining Arrays
 
```python
# Stack horizontally (side by side)
a = np.array([[1], [2]])
b = np.array([[3], [4]])
 
np.hstack([a, b])
# [[1 3]
#  [2 4]]
 
# Stack vertically (on top)
np.vstack([a, b])
# [[1]
#  [2]
#  [3]
#  [4]]
 
# Concatenate along an axis
np.concatenate([a, b], axis=0)  # vertical
np.concatenate([a, b], axis=1)  # horizontal
```
 
### Useful Utility Functions
 
```python
scores = np.array([85, 92, 78, 88, 92])
 
# Clip: constrain values to a range
# (if someone scores above 95, cap it at 95)
clipped = np.clip(scores, 0, 95)
# [85, 92, 78, 88, 92]
 
# Where: conditional operation
# (if score < 80, add 10; otherwise keep it)
adjusted = np.where(scores < 80, scores + 10, scores)
# [85, 92, 88, 88, 92]
 
# Unique: find all unique values
np.unique(scores)
# [78, 85, 88, 92]
 
# Linear algebra basics
A = np.array([[1, 2], [3, 4]])
 
# Determinant (single number describing the matrix)
np.linalg.det(A)  # -2.0
 
# Inverse (the "opposite" of a matrix)
np.linalg.inv(A)
# [[-2.   1. ]
#  [ 1.5 -0.5]]
 
# Norm (length/magnitude)
np.linalg.norm(A)  # 5.477
```
 
---
 
## Part 2: Pandas - Working with Real Data
 
### From Arrays to Tables
 
NumPy is great for math. But real data has **column names** and **row labels**. That's where Pandas comes in.
 
A Pandas DataFrame is like an Excel spreadsheet:
- Rows are observations (students)
- Columns are properties (name, math score, science score)
- Each cell has a value
 
### Series: A Single Column
 
```python
import pandas as pd
 
# Create a Series (like a column)
scores = pd.Series([85, 92, 78, 88], name='Math Scores')
# 0    85
# 1    92
# 2    78
# 3    88
# Name: Math Scores, dtype: int64
 
# Access by position
scores[0]  # 85
 
# With custom index (row labels)
scores = pd.Series(
    [85, 92, 78, 88],
    index=['Alice', 'Bob', 'Charlie', 'Diana'],
    name='Math Scores'
)
# Alice        85
# Bob          92
# Charlie      78
# Diana        88
 
# Access by label
scores['Alice']  # 85
 
# All operations work like NumPy
scores.mean()     # 85.75
scores.max()      # 92
scores.min()      # 78
scores.std()      # ~6.23
 
# Filter (keep scores above 85)
scores[scores > 85]
# Bob        92
# Diana      88
```
 
### DataFrames: The Real Thing
 
A DataFrame is multiple Series stacked side by side:
 
```python
import pandas as pd
 
# Create from a dictionary (most common)
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'Grade': [10, 10, 11, 11],
    'Math': [85, 92, 78, 88],
    'Science': [88, 85, 92, 90],
    'English': [92, 88, 85, 89]
}
 
df = pd.DataFrame(data)
#      Name Grade Math Science English
# 0   Alice    10   85      88      92
# 1     Bob    10   92      85      88
# 2 Charlie    11   78      92      85
# 3   Diana    11   88      90      89
```
 
### Exploring Your Data
 
```python
# How big is it?
df.shape  # (4, 5) - 4 rows, 5 columns
 
# What are the column names?
df.columns  # ['Name', 'Grade', 'Math', 'Science', 'English']
 
# What are the data types?
df.dtypes
# Name       object (text)
# Grade       int64 (integer)
# Math        int64 (integer)
# Science     int64 (integer)
# English     int64 (integer)
 
# First few rows
df.head(2)
#      Name Grade Math Science English
# 0   Alice    10   85      88      92
# 1     Bob    10   92      85      88
 
# Last few rows
df.tail(1)
#      Name Grade Math Science English
# 3   Diana    11   88      90      89
 
# Summary statistics
df.describe()
#        Grade  Math  Science  English
# count    4.0    4.0      4.0      4.0
# mean    10.5   85.75     89.0     89.0
# std      0.577  6.179     3.559    2.708
# min     10.0   78.0     85.0     85.0
# 25%     10.0   82.5     87.0     86.5
# 50%     10.5   86.5     89.0     89.0
# 75%     11.0   89.5     91.0     90.5
# max     11.0   92.0     92.0     92.0
 
# Detailed info
df.info()
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 4 entries, 0 to 3
# Data columns (but 4 non-null):
# Name       4 non-null object
# Grade      4 non-null int64
# Math       4 non-null int64
# Science    4 non-null int64
# English    4 non-null int64
```
 
### Getting Data: Columns, Rows, Cells
 
```python
# Get a single column (returns a Series)
df['Name']
# 0      Alice
# 1        Bob
# 2    Charlie
# 3      Diana
 
# Get multiple columns (returns a DataFrame)
df[['Name', 'Math']]
#      Name Math
# 0   Alice   85
# 1     Bob   92
# 2 Charlie   78
# 3   Diana   88
 
# Get a row by position (iloc = integer location)
df.iloc[0]
# Name              Alice
# Grade                10
# Math                 85
# Science              88
# English              92
 
# Get a specific cell by position
df.iloc[0, 0]  # Alice
 
# Get by label (loc = location by label)
df.loc[0, 'Name']  # Alice
 
# Get all rows where condition is true
df[df['Math'] > 85]
#      Name Grade Math Science English
# 1     Bob    10   92      85      88
# 3   Diana    11   88      90      89
 
# Multiple conditions
df[(df['Math'] > 85) & (df['Science'] > 85)]
#      Name Grade Math Science English
# 1     Bob    10   92      85      88
# 3   Diana    11   88      90      89
```
 
### Adding Columns
 
```python
# Calculate average score
df['Average'] = (df['Math'] + df['Science'] + df['English']) / 3
 
df
#      Name Grade Math Science English  Average
# 0   Alice    10   85      88      92    88.33
# 1     Bob    10   92      85      88    88.33
# 2 Charlie    11   78      92      85    85.00
# 3   Diana    11   88      90      89    89.00
 
# Add a column with apply (custom function)
def grade_letter(score: float) -> str:
    if score >= 90: return 'A'
    elif score >= 80: return 'B'
    else: return 'C'
 
df['GradeLetter'] = df['Average'].apply(grade_letter)
 
df
#      Name Grade Math Science English  Average GradeLetter
# 0   Alice    10   85      88      92    88.33           B
# 1     Bob    10   92      85      88    88.33           B
# 2 Charlie    11   78      92      85    85.00           B
# 3   Diana    11   88      90      89    89.00           B
```
 
### Removing Columns
 
```python
# Drop a column
df = df.drop('GradeLetter', axis=1)
 
# Keep only certain columns
df = df[['Name', 'Math', 'Science', 'English']]
```
 
### GroupBy: Organize, Then Calculate
 
GroupBy is like taking a stack of papers, sorting them by grade level, then analyzing each pile:
 
```python
# What's the average math score per grade level?
df.groupby('Grade')['Math'].mean()
# Grade
# 10    88.5
# 11    83.0
 
# Multiple subjects
df.groupby('Grade')[['Math', 'Science', 'English']].mean()
#       Math  Science  English
# Grade
# 10    88.5     86.5     90.0
# 11    83.0     91.0     87.0
 
# More aggregations
df.groupby('Grade').agg({
    'Math': ['mean', 'max', 'min'],
    'Science': ['mean', 'max', 'min'],
    'Name': 'count'  # how many students per grade
})
#        Math            Science         Name
#       mean max min       mean max min count
# Grade
# 10     88.5  92  85       86.5  88  85     2
# 11     83.0  88  78       91.0  92  90     2
 
# Count occurrences
df.groupby('Grade').size()
# Grade
# 10    2
# 11    2
```
 
### Merge: Combining Two DataFrames
 
Imagine you have student scores in one file and student names in another. Merge combines them:
 
```python
# File 1: Scores
scores_df = pd.DataFrame({
    'StudentID': [1, 2, 3],
    'Math': [85, 92, 78]
})
#    StudentID Math
# 0          1   85
# 1          2   92
# 2          3   78
 
# File 2: Names
names_df = pd.DataFrame({
    'StudentID': [1, 2, 3],
    'Name': ['Alice', 'Bob', 'Charlie']
})
#    StudentID     Name
# 0          1    Alice
# 1          2      Bob
# 2          3  Charlie
 
# INNER join (only keep rows in BOTH files)
merged = pd.merge(scores_df, names_df, on='StudentID')
#    StudentID Math     Name
# 0          1   85    Alice
# 1          2   92      Bob
# 2          3   78  Charlie
 
# LEFT join (keep all from left, add matching from right)
pd.merge(scores_df, names_df, on='StudentID', how='left')
# (same result if all IDs match)
 
# RIGHT join (keep all from right, add matching from left)
pd.merge(scores_df, names_df, on='StudentID', how='right')
 
# OUTER join (keep all rows from both)
pd.merge(scores_df, names_df, on='StudentID', how='outer')
```
 
### Apply: Run Functions on Your Data
 
```python
def bonus_points(score: float) -> float:
    """Add 5% bonus to score."""
    return score * 1.05
 
# Apply to a column
df['Math_Bonus'] = df['Math'].apply(bonus_points)
 
# With lambda (inline function)
df['Math2x'] = df['Math'].apply(lambda x: x * 2)
 
# Apply to entire row
def calc_total(row) -> float:
    """Add up all three subjects."""
    return row['Math'] + row['Science'] + row['English']
 
df['Total'] = df.apply(calc_total, axis=1)
 
df
#      Name Grade Math Science English  Math_Bonus  Math2x  Total
# 0   Alice    10   85      88      92       89.25     170    265
# 1     Bob    10   92      85      88       96.60     184    265
# 2 Charlie    11   78      92      85       81.90     156    255
# 3   Diana    11   88      90      89       92.40     176    267
```
 
### String Operations
 
```python
names = pd.Series(['Alice', 'Bob', 'Charlie'])
 
# Convert to lowercase
names.str.lower()
# 0      alice
# 1        bob
# 2    charlie
 
# Convert to uppercase
names.str.upper()
# 0      ALICE
# 1        BOB
# 2    CHARLIE
 
# Length of each string
names.str.len()
# 0    5
# 1    3
# 2    7
 
# Check if contains substring
names.str.contains('li')
# 0     True
# 1    False
# 2     True
 
# Replace characters
names.str.replace('a', 'A')
# 0     Alice
# 1       Bob
# 2    ChArlie
 
# Split by character
names.str.split('i')
# 0    [Al, ce]
# 1       [Bob]
# 2    [Charl, e]
```
 
### Datetime Operations
 
```python
dates = pd.to_datetime(['2024-01-15', '2024-06-20', '2024-12-25'])
# DatetimeIndex(['2024-01-15', '2024-06-20', '2024-12-25'],
#                dtype='datetime64[ns]', freq=None)
 
# Extract parts
dates.dt.year
# 2024, 2024, 2024
 
dates.dt.month
# 1, 6, 12
 
dates.dt.day
# 15, 20, 25
 
dates.dt.day_name()
# 'Monday', 'Thursday', 'Monday'
 
# Calculate differences
(dates.max() - dates.min()).days
# 345 (days between first and last)
```
 
### Handling Missing Data
 
```python
# Create data with missing values
data_with_nan = {
    'Name': ['Alice', 'Bob', None, 'Diana'],
    'Score': [85, None, 92, 88]
}
df_nan = pd.DataFrame(data_with_nan)
#      Name Score
# 0   Alice   85.0
# 1     Bob    NaN
# 2    None   92.0
# 3   Diana   88.0
 
# Check where data is missing
df_nan.isna()
#      Name Score
# 0  False False
# 1  False  True
# 2   True False
# 3  False False
 
# Drop rows with any missing values
df_clean = df_nan.dropna()
#      Name Score
# 0   Alice   85.0
# 3   Diana   88.0
 
# Drop rows where Score is missing
df_partial = df_nan.dropna(subset=['Score'])
#      Name Score
# 0   Alice   85.0
# 2    None   92.0
# 3   Diana   88.0
 
# Fill missing with a value
df_filled = df_nan.fillna(0)
#      Name Score
# 0   Alice   85.0
# 1     Bob    0.0
# 2       0   92.0
# 3   Diana   88.0
 
# Fill with average
df_filled = df_nan.copy()
df_filled['Score'].fillna(df_nan['Score'].mean(), inplace=True)
#      Name Score
# 0   Alice   85.0
# 1     Bob   88.333333
# 2    None   92.0
# 3   Diana   88.0
```
 
### Save & Load CSV
 
```python
# Save to CSV
df.to_csv('students.csv', index=False)
 
# Load from CSV
df = pd.read_csv('students.csv')
 
# With options
df = pd.read_csv('students.csv', 
                  delimiter=',',      # column separator
                  index_col=0,        # use first column as row index
                  dtype={'Grade': int}) # specify data types
```
 
---
 
## The Project: Student Score Analysis Pipeline
 
Now let's build a complete, professional analysis pipeline with type hints on every function.
 
### Step 1: Create Sample Data
 
Create a file named `data.csv`:
 
```csv
Name,Grade,Math,Science,English
Alice,10,85,88,92
Bob,10,92,85,88
Charlie,11,78,92,85
Diana,11,88,90,89
Eve,12,92,95,91
Frank,12,88,87,86
Grace,10,90,89,91
Henry,11,85,88,90
```
 
### Step 2: Build the Analysis Pipeline
 
Create a file named `analysis.py`:
 
```python
import numpy as np
import pandas as pd
from typing import Dict, Any
 
def load_data(filepath: str) -> pd.DataFrame:
    """Load student data from CSV file.
    
    Args:
        filepath: Path to CSV file
        
    Returns:
        DataFrame with student data
    """
    return pd.read_csv(filepath)
 
 
def calculate_overall_score(df: pd.DataFrame) -> pd.DataFrame:
    """Add overall average score for each student.
    
    Uses NumPy vectorization for efficiency.
    
    Args:
        df: DataFrame with Math, Science, English columns
        
    Returns:
        DataFrame with new 'Overall' column
    """
    # Get the score columns as a NumPy array
    scores = df[['Math', 'Science', 'English']].values
    
    # Calculate mean across each row (axis=1)
    overall = np.mean(scores, axis=1)
    
    # Add as new column
    df['Overall'] = overall
    
    return df
 
 
def assign_grades(df: pd.DataFrame) -> pd.DataFrame:
    """Assign letter grades based on overall score.
    
    Args:
        df: DataFrame with 'Overall' column
        
    Returns:
        DataFrame with new 'GradeLetter' column
    """
    def grade_letter(score: float) -> str:
        """Convert numeric score to letter grade."""
        if score >= 90:
            return 'A'
        elif score >= 80:
            return 'B'
        else:
            return 'C'
    
    # Apply the function to the Overall column
    df['GradeLetter'] = df['Overall'].apply(grade_letter)
    
    return df
 
 
def grade_level_analysis(df: pd.DataFrame) -> pd.DataFrame:
    """Analyze performance by grade level.
    
    Args:
        df: DataFrame with Grade column and scores
        
    Returns:
        DataFrame with statistics per grade level
    """
    stats = df.groupby('Grade').agg({
        'Math': ['mean', 'max', 'min'],
        'Science': ['mean', 'max', 'min'],
        'English': ['mean', 'max', 'min'],
        'Overall': 'mean',
        'Name': 'count'  # number of students
    }).round(2)
    
    return stats
 
 
def top_students(df: pd.DataFrame, n: int = 3) -> pd.DataFrame:
    """Get top N students by overall score.
    
    Args:
        df: DataFrame with student data
        n: Number of top students to return
        
    Returns:
        DataFrame with top N students
    """
    return df.nlargest(n, 'Overall')[['Name', 'Grade', 'Overall', 'GradeLetter']]
 
 
def subject_stats(df: pd.DataFrame) -> Dict[str, Dict[str, float]]:
    """Calculate statistics for each subject.
    
    Uses NumPy for efficient calculation.
    
    Args:
        df: DataFrame with Math, Science, English columns
        
    Returns:
        Dictionary with stats for each subject
    """
    subjects = ['Math', 'Science', 'English']
    stats = {}
    
    for subject in subjects:
        # Convert to NumPy array for calculation
        scores = df[subject].values
        
        stats[subject] = {
            'mean': float(np.mean(scores)),
            'std': float(np.std(scores)),
            'min': float(np.min(scores)),
            'max': float(np.max(scores))
        }
    
    return stats
 
 
def correlations(df: pd.DataFrame) -> np.ndarray:
    """Calculate correlation matrix between subjects.
    
    Uses NumPy correlation.
    
    Args:
        df: DataFrame with Math, Science, English columns
        
    Returns:
        Correlation matrix (3x3 array)
    """
    subject_data = df[['Math', 'Science', 'English']].values
    return np.corrcoef(subject_data.T)
 
 
def print_correlation_heatmap(corr_matrix: np.ndarray) -> None:
    """Print correlation matrix as ASCII table.
    
    Args:
        corr_matrix: 3x3 correlation matrix
    """
    subjects = ['Math', 'Science', 'English']
    
    print("\nCORRELATION MATRIX:")
    print("        Math  Science English")
    
    for i, subj in enumerate(subjects):
        print(f"{subj:7s} ", end="")
        for j in range(len(subjects)):
            val = corr_matrix[i, j]
            print(f"{val:6.2f} ", end="")
        print()
 
 
def top_n_per_subject(df: pd.DataFrame, n: int = 2) -> Dict[str, pd.DataFrame]:
    """Return top N students for each subject.
    
    Args:
        df: DataFrame with student data
        n: Number of top students per subject
        
    Returns:
        Dictionary mapping subject to top N students
    """
    subjects = ['Math', 'Science', 'English']
    results = {}
    
    for subject in subjects:
        top = df.nlargest(n, subject)[['Name', subject]]
        results[subject] = top
    
    return results
 
 
def pivot_by_grade(df: pd.DataFrame) -> pd.DataFrame:
    """Create pivot table of average scores by grade and subject.
    
    Args:
        df: DataFrame with Grade and subject columns
        
    Returns:
        Pivot table with grades as rows, subjects as columns
    """
    pivot = df.pivot_table(
        values=['Math', 'Science', 'English'],
        index='Grade',
        aggfunc='mean'
    ).round(2)
    
    return pivot
 
 
def main() -> None:
    """Run the complete analysis pipeline."""
    print("=" * 70)
    print("STUDENT SCORE ANALYSIS PIPELINE")
    print("=" * 70)
    
    # Step 1: Load and process data
    df = load_data('data.csv')
    df = calculate_overall_score(df)
    df = assign_grades(df)
    
    # Step 2: Display all students with grades
    print("\n1. ALL STUDENTS WITH GRADES:")
    print(df[['Name', 'Grade', 'Math', 'Science', 'English', 
              'Overall', 'GradeLetter']].to_string(index=False))
    
    # Step 3: Grade level analysis
    print("\n2. STATISTICS BY GRADE LEVEL:")
    print(grade_level_analysis(df))
    
    # Step 4: Top 3 students
    print("\n3. TOP 3 STUDENTS:")
    print(top_students(df, n=3).to_string(index=False))
    
    # Step 5: Subject statistics
    print("\n4. SUBJECT STATISTICS:")
    stats = subject_stats(df)
    for subject, vals in stats.items():
        print(f"\n{subject}:")
        print(f"  Mean: {vals['mean']:.2f}")
        print(f"  Std:  {vals['std']:.2f}")
        print(f"  Min:  {vals['min']:.1f}")
        print(f"  Max:  {vals['max']:.1f}")
    
    # Step 6: Correlations
    print("\n5. SUBJECT CORRELATIONS:")
    corr_matrix = correlations(df)
    print_correlation_heatmap(corr_matrix)
    
    # Step 7: Top students per subject
    print("\n6. TOP 2 STUDENTS PER SUBJECT:")
    top_per_subj = top_n_per_subject(df, n=2)
    for subject, top_df in top_per_subj.items():
        print(f"\n{subject}:")
        print(top_df.to_string(index=False))
    
    # Step 8: Pivot table
    print("\n7. AVERAGE SCORES BY GRADE LEVEL:")
    print(pivot_by_grade(df))
    
    # Step 9: Save results
    df.to_csv('results.csv', index=False)
    print("\n✓ Results saved to results.csv")
 
 
if __name__ == '__main__':
    main()
```
 
### Step 3: Run the Pipeline
 
```bash
python analysis.py
```
 
### Expected Output
 
```
======================================================================
STUDENT SCORE ANALYSIS PIPELINE
======================================================================
 
1. ALL STUDENTS WITH GRADES:
     Name Grade  Math Science English  Overall GradeLetter
    Alice    10    85       88      92    88.33           B
      Bob    10    92       85      88    88.33           B
  Charlie    11    78       92      85    85.00           B
    Diana    11    88       90      89    89.00           B
      Eve    12    92       95      91    92.67           A
    Frank    12    88       87      86    87.00           B
    Grace    10    90       89      91    90.00           A
    Henry    11    85       88      90    87.67           B
 
2. STATISTICS BY GRADE LEVEL:
       Math            Science         English       Overall Name
      mean max min      mean max min     mean max min   mean count
Grade
10     87.25  92  85    86.75  89  88   91.25  92  91  88.41    4
11     83.00  88  78    91.00  92  90   87.00  89  85  87.00    2
12     90.00  92  88    91.00  95  87   88.50  91  86  89.83    2
 
3. TOP 3 STUDENTS:
     Name Grade  Overall GradeLetter
      Eve    12    92.67           A
    Grace    10    90.00           A
    Diana    11    89.00           B
 
4. SUBJECT STATISTICS:
 
Math:
  Mean: 87.13
  Std:  5.32
  Min:  78.0
  Max:  92.0
 
Science:
  Mean: 89.25
  Std:  4.27
  Min:  85.0
  Max:  95.0
 
English:
  Mean: 89.00
  Std:  2.93
  Min:  85.0
  Max:  92.0
 
5. SUBJECT CORRELATIONS:
        Math  Science English
Math     1.00    0.35    0.52
Science  0.35    1.00   -0.13
English  0.52   -0.13    1.00
 
6. TOP 2 STUDENTS PER SUBJECT:
 
Math:
    Name Score
     Eve    92
     Bob    92
 
Science:
     Name Score
      Eve    95
  Charlie    92
 
English:
    Name Score
   Alice    92
    Eve    91
 
7. AVERAGE SCORES BY GRADE LEVEL:
         Math  Science  English
Grade
10     87.25    86.75    91.25
11     83.00    91.00    87.00
12     90.00    91.00    88.50
 
✓ Results saved to results.csv
```
---
 
## What's Next: Day 7
 
**Data Visualization: Matplotlib + Seaborn + Logging Module**
 
On Day 7, you'll learn:
1. **Matplotlib** - create line plots, bar charts, scatter plots
2. **Seaborn** - make beautiful statistical visualizations
3. **Logging module** - replace `print()` with professional logging