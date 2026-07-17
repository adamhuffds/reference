# NumPy Reference — Operations & Broadcasting

Docs: https://numpy.org/doc/stable/user/basics.broadcasting.html

Prerequisite: `numpy-basics-arrays.md` for array creation/indexing.

## Element-Wise Arithmetic

```python
import numpy as np
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

a + b        # array([11, 22, 33])
a - b          # array([-9, -18, -27])
a * b            # array([10, 40, 90]) — element-wise, NOT matrix multiplication
a / b              # array([0.1, 0.1, 0.1])
a ** 2               # array([1, 4, 9])
```
All these operate element-wise by default — this is a key NumPy behavior
difference from plain Python lists, where `list1 * list2` would raise an
error and `list1 + list2` concatenates rather than adding.

## Scalar Broadcasting

```python
a = np.array([1, 2, 3])
a + 10          # array([11, 12, 13]) — scalar applied to every element
a * 2             # array([2, 4, 6])
```

## Broadcasting Between Different Shapes

Broadcasting is NumPy's rule set for performing operations on arrays of
different (but compatible) shapes, without explicitly duplicating data.

```python
grid = np.array([[1, 2, 3], [4, 5, 6]])     # shape (2, 3)
row = np.array([10, 20, 30])                   # shape (3,)

grid + row      # row is broadcast across both rows of grid
# array([[11, 22, 33],
#        [14, 25, 36]])
```

### Broadcasting Rules
Two dimensions are compatible when they're equal, or when one of them is 1.
NumPy compares shapes from the **rightmost** dimension leftward:
```
grid: (2, 3)
row:     (3,)   -> treated as (1, 3), then broadcast to (2, 3)
```
```python
col = np.array([[100], [200]])          # shape (2, 1)
grid + col
# array([[101, 102, 103],
#        [204, 205, 206]])
```
Here `(2, 1)` broadcasts against `(2, 3)` by stretching the single column
across all 3 columns. Broadcasting fails (raises `ValueError`) when
dimensions don't match and neither is 1 — e.g. `(2, 3)` and `(2, 4)` cannot
broadcast together.
Docs: https://numpy.org/doc/stable/user/basics.broadcasting.html#general-broadcasting-rules

## Aggregation Functions

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.sum()            # 21 — sum of ALL elements
arr.sum(axis=0)         # array([5, 7, 9]) — sum DOWN each column
arr.sum(axis=1)            # array([6, 15]) — sum ACROSS each row

arr.mean()                    # 3.5
arr.std()                       # standard deviation
arr.var()                         # variance
arr.min() / arr.max()               # min/max across all elements
arr.min(axis=0)                       # min per column
arr.argmin() / arr.argmax()             # INDEX of the min/max value (flattened index)
```
`axis` is the parameter most people trip on initially: `axis=0` operates
"down" columns (collapsing rows), `axis=1` operates "across" rows
(collapsing columns) — think of `axis` as specifying which dimension gets
collapsed/removed by the operation.

## Comparison & Logical Operations

```python
a = np.array([1, 2, 3, 4, 5])

a > 3                  # array([False, False, False, True, True])
a[a > 3]                  # array([4, 5]) — boolean indexing, see numpy-basics-arrays.md

np.where(a > 3, "big", "small")     # array(['small', 'small', 'small', 'big', 'big'])
np.any(a > 3)                          # True — at least one element matches
np.all(a > 3)                             # False — not every element matches

np.logical_and(a > 1, a < 4)                 # element-wise AND
np.logical_or(a < 2, a > 4)                     # element-wise OR
```
`np.where(condition, if_true, if_false)` is the vectorized equivalent of a
ternary expression applied across an entire array — much faster than a
Python-level loop with an `if`/`else` per element.

## Universal Functions (ufuncs)

```python
np.sqrt(arr)        # element-wise square root
np.exp(arr)            # element-wise e^x
np.log(arr)               # natural log
np.abs(arr)                  # absolute value
np.round(arr, 2)                # round to 2 decimal places
np.clip(arr, 0, 10)                # constrain all values into the range [0, 10]
```
"ufunc" (universal function) is NumPy's term for functions that operate
element-wise on arrays with broadcasting support — most of the `np.*` math
functions fall into this category.

## Sorting & Unique Values

```python
arr = np.array([3, 1, 4, 1, 5])

np.sort(arr)               # array([1, 1, 3, 4, 5]) — returns a sorted COPY
arr.sort()                    # sorts IN PLACE, modifies arr directly

np.argsort(arr)                  # indices that WOULD sort the array — useful for sorting one array by another's order
np.unique(arr)                      # array([1, 3, 4, 5]) — sorted unique values
np.unique(arr, return_counts=True)     # also returns how many times each unique value appears
```

## Matrix Multiplication (Distinct from Element-Wise `*`)

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

a @ b                # matrix multiplication (Python 3.5+ operator, current preferred syntax)
np.matmul(a, b)         # equivalent function form
np.dot(a, b)               # also equivalent for 2D arrays (dot() has broader/older semantics for higher dimensions)
```
`@` is the current recommended operator for matrix multiplication — clearer
at a glance than `np.dot()`, and avoids the element-wise-vs-matrix
confusion that plain `*` creates. See `numpy-linear-algebra-random.md` for
more linear algebra operations.