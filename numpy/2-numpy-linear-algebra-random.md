# NumPy Reference — Linear Algebra & Random

Docs: https://numpy.org/doc/stable/reference/routines.linalg.html | https://numpy.org/doc/stable/reference/random/index.html

Prerequisite: `numpy-basics-arrays.md`, `numpy-operations-broadcasting.md`

## Linear Algebra (`np.linalg`)

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])

np.linalg.det(A)            # determinant
np.linalg.inv(A)               # matrix inverse
np.linalg.transpose(A)            # same as A.T
np.linalg.matrix_rank(A)             # rank of the matrix

# Eigenvalues/eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)

# Solve a system of linear equations: Ax = b
b = np.array([5, 6])
x = np.linalg.solve(A, b)         # preferred over computing inv(A) @ b — more numerically stable
```
`np.linalg.solve()` should generally be used instead of manually computing
`np.linalg.inv(A) @ b` — solving directly avoids some of the numerical
precision loss that explicit matrix inversion introduces, and is also
faster. This is the current best-practice approach whenever you're solving
`Ax = b` rather than needing the inverse matrix itself for another purpose.

### Norms (Vector/Matrix Magnitude)
```python
v = np.array([3, 4])
np.linalg.norm(v)             # 5.0 — Euclidean (L2) norm by default
np.linalg.norm(v, ord=1)         # L1 norm (sum of absolute values)
```

### Decompositions
```python
U, S, Vt = np.linalg.svd(A)     # Singular Value Decomposition
Q, R = np.linalg.qr(A)             # QR decomposition
```
SVD is commonly used in dimensionality reduction (it underlies PCA) and
data compression contexts.

## Random Number Generation

### Modern API (Generator-based, current recommended approach)
```python
rng = np.random.default_rng(seed=42)      # create a reproducible random Generator instance

rng.random(5)                     # 5 random floats in [0, 1)
rng.integers(0, 10, size=5)          # 5 random integers in [0, 10)
rng.normal(0, 1, size=5)                # 5 samples from a normal distribution (mean=0, std=1)
rng.choice([1, 2, 3, 4], size=2)           # randomly sample 2 values from a list, with replacement by default
rng.choice([1, 2, 3, 4], size=2, replace=False)   # sample without replacement
rng.shuffle(arr)                              # shuffle an array IN PLACE
```
`np.random.default_rng()` (introduced in NumPy 1.17) is the current
best-practice API for random number generation — prefer it over the legacy
global-state functions below, since each `Generator` instance is
independent and reproducible without side effects on other code that also
uses `np.random`.

### Legacy API (Still Common in Older Code/Tutorials)
```python
np.random.seed(42)              # sets a GLOBAL random seed — affects every subsequent np.random.* call anywhere
np.random.rand(5)                  # 5 random floats in [0, 1)
np.random.randint(0, 10, size=5)      # 5 random integers in [0, 10)
np.random.normal(0, 1, size=5)           # normal distribution samples
np.random.choice([1, 2, 3, 4], size=2)      # random sample from a list
```
The legacy API's global seed state is the main issue — calling
`np.random.seed()` anywhere affects randomness everywhere else in the
program that also uses `np.random.*`, which becomes a source of subtle,
hard-to-track bugs in larger codebases (e.g. two unrelated modules stepping
on each other's random state). The `Generator`-based API avoids this by
keeping random state scoped to each `rng` instance you create.

### Reproducibility
```python
rng = np.random.default_rng(seed=42)
rng.random(3)      # always produces the SAME sequence when seeded with 42
```
Seeding matters for reproducible experiments/tests — always set a seed when
results need to be exactly repeatable (e.g. splitting a dataset the same
way across multiple runs), and deliberately omit it when you want genuine
randomness each run.

## Common Statistics Distributions

```python
rng = np.random.default_rng(seed=42)

rng.normal(loc=0, scale=1, size=1000)          # Gaussian/normal distribution
rng.uniform(low=0, high=10, size=1000)            # uniform distribution
rng.binomial(n=10, p=0.5, size=1000)                 # binomial distribution
rng.poisson(lam=3, size=1000)                           # Poisson distribution
rng.exponential(scale=1.0, size=1000)                      # exponential distribution
```
Full distribution list: https://numpy.org/doc/stable/reference/random/generator.html#distributions

## Practical Example: Train/Test Split (Manual, No scikit-learn)

```python
rng = np.random.default_rng(seed=42)

data = np.arange(100)
indices = rng.permutation(len(data))       # shuffled indices, reproducible via the seed

split_point = int(len(data) * 0.8)
train_idx, test_idx = indices[:split_point], indices[split_point:]

train_data = data[train_idx]
test_data = data[test_idx]
```
For real ML work, `sklearn.model_selection.train_test_split` handles this
(and edge cases like stratification) more robustly — this is mainly useful
for understanding what's happening underneath, or for quick scripts where
pulling in scikit-learn isn't warranted.