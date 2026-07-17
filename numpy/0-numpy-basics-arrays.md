# NumPy Reference — Arrays & Basics

Docs: https://numpy.org/doc/stable/

```python
import numpy as np      # universal convention — always import with this alias
```

## Why NumPy Over Plain Python Lists

NumPy arrays are stored as contiguous blocks of memory with a single fixed
dtype, unlike Python lists (which store pointers to individually-boxed
objects). This makes NumPy arrays dramatically faster for numeric operations
and lets operations run as vectorized C loops instead of Python-level `for`
loops — the entire reason NumPy exists as the foundation of the Python data
science stack (pandas, scikit-learn, etc. are all built on it).

## Creating Arrays

```python
np.array([1, 2, 3])                  # from a Python list
np.array([[1, 2], [3, 4]])              # 2D array, from a list of lists

np.zeros(5)                # array([0., 0., 0., 0., 0.])
np.zeros((3, 4))             # 3x4 array of zeros
np.ones((2, 3))                # 2x3 array of ones
np.full((2, 2), 7)                # 2x2 array filled with the value 7
np.eye(3)                           # 3x3 identity matrix

np.arange(0, 10, 2)          # array([0, 2, 4, 6, 8]) — like range(), but returns an array
np.linspace(0, 1, 5)           # array([0., 0.25, 0.5, 0.75, 1.]) — 5 evenly spaced points from 0 to 1

np.random.rand(3, 3)              # 3x3 array, uniform random [0, 1) — see numpy-linear-algebra-random.md
```

## Array Properties

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.shape        # (2, 3) — dimensions
arr.ndim           # 2 — number of dimensions
arr.size              # 6 — total number of elements
arr.dtype               # dtype('int64') — the data type of elements
```

## Data Types (dtype)

```python
np.array([1, 2, 3], dtype=np.float64)     # force a specific type on creation
arr.astype(np.int32)                          # convert an existing array's type (returns a new array)
```
Common dtypes: `np.int32`, `np.int64`, `np.float32`, `np.float64`,
`np.bool_`. Choosing a smaller dtype (e.g. `float32` over the default
`float64`) matters for memory usage on large arrays, at the cost of
precision — worth considering for large datasets where you know the
precision requirements are modest.
Docs: https://numpy.org/doc/stable/reference/arrays.dtypes.html

## Indexing & Slicing

```python
arr = np.array([10, 20, 30, 40, 50])

arr[0]           # 10
arr[-1]            # 50
arr[1:4]              # array([20, 30, 40]) — slicing works like Python lists
arr[::2]                # array([10, 30, 50]) — every other element
```

### 2D Indexing
```python
grid = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

grid[0, 0]          # 1 — row 0, column 0
grid[1, :]            # array([4, 5, 6]) — entire row 1
grid[:, 1]              # array([2, 5, 8]) — entire column 1
grid[0:2, 1:3]             # sub-grid: rows 0-1, columns 1-2
```

### Boolean (Mask) Indexing
```python
arr = np.array([1, -2, 3, -4, 5])

arr[arr > 0]              # array([1, 3, 5]) — select elements matching a condition
arr[arr > 0] = 0             # set all positive elements to 0, in place
```
This pattern — a boolean condition used directly as an index — is
idiomatic NumPy and generally preferred over a Python `for` loop with an
`if` check, since it runs as a vectorized operation instead of iterating in
pure Python.

### Fancy Indexing (Index Arrays)
```python
arr = np.array([10, 20, 30, 40, 50])
indices = [0, 2, 4]
arr[indices]         # array([10, 30, 50]) — select multiple specific positions at once
```

## Reshaping

```python
arr = np.arange(12)          # array([0, 1, ..., 11])

arr.reshape(3, 4)               # reshape into a 3x4 2D array (must match total element count: 3*4=12)
arr.reshape(3, -1)                 # -1 lets NumPy infer that dimension automatically
arr.flatten()                        # collapse back to 1D — returns a COPY
arr.ravel()                            # collapse to 1D — returns a VIEW when possible (faster, shares memory)

arr.T                                     # transpose (swap rows/columns) for 2D+ arrays
```
`flatten()` vs `ravel()`: prefer `ravel()` when you don't need an
independent copy, since it avoids an unnecessary memory allocation — use
`flatten()` specifically when you need to modify the result without
affecting the original array.

## Views vs. Copies (Important Gotcha)

```python
arr = np.array([1, 2, 3, 4, 5])
view = arr[1:3]           # slicing returns a VIEW, not a copy — shares the same underlying memory
view[0] = 99                 # this ALSO modifies the original arr!

copy = arr[1:3].copy()          # explicit .copy() breaks the memory link
copy[0] = 0                        # this does NOT affect the original arr
```
This is one of the most common sources of subtle bugs for people coming
from plain Python lists (where slicing always copies) — when in doubt about
whether you need an independent array, call `.copy()` explicitly.

## Combining & Splitting Arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])          # array([1, 2, 3, 4, 5, 6])
np.vstack([a, b])                  # stack vertically -> 2D array, one row per input
np.hstack([a, b])                     # stack horizontally -> array([1, 2, 3, 4, 5, 6])

np.split(np.arange(9), 3)                # split into 3 equal parts
```