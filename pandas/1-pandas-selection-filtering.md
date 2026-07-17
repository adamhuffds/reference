# pandas Reference — Series & DataFrames Basics

Docs: https://pandas.pydata.org/docs/

```python
import pandas as pd      # universal convention — always import with this alias
import numpy as np          # pandas is built on numpy; commonly imported alongside it
```

## Series (1D) vs. DataFrame (2D)

A `Series` is a single labeled column of data; a `DataFrame` is a table —
essentially a dict of Series sharing a common index.

```python
s = pd.Series([10, 20, 30], index=["a", "b", "c"])
s["b"]           # 20 — label-based access

df = pd.DataFrame({
    "name": ["Adam", "Beth", "Carl"],
    "age": [30, 25, 40],
})
```

## Creating DataFrames

```python
pd.DataFrame({"col1": [1, 2], "col2": [3, 4]})            # from a dict of lists
pd.DataFrame([[1, 3], [2, 4]], columns=["col1", "col2"])     # from a list of lists
pd.DataFrame.from_records([{"a": 1, "b": 2}, {"a": 3, "b": 4}])   # from a list of dicts
```

## Inspecting a DataFrame

```python
df.head()             # first 5 rows
df.head(10)              # first 10 rows
df.tail()                   # last 5 rows
df.shape                       # (rows, columns) tuple
df.info()                         # column names, dtypes, non-null counts, memory usage
df.describe()                        # summary statistics for numeric columns (mean, std, min, max, quartiles)
df.dtypes                               # dtype of each column
df.columns                                 # column names as an Index
df.index                                      # row index
df.nunique()                                     # number of unique values per column
df["col"].value_counts()                            # frequency count of each unique value in a column
```
`df.info()` and `df.describe()` are usually the first two calls worth making
on any new dataset — `.info()` tells you what you're working with
structurally (nulls, types), `.describe()` gives a numeric sanity check
(are values in the range you'd expect, any obvious outliers).

## Index

```python
df.set_index("name")             # use a column as the index (returns a new DataFrame)
df.reset_index()                    # move the index back into a regular column, restore default 0,1,2... index
df.reset_index(drop=True)              # reset the index WITHOUT keeping the old one as a column
```
A meaningful index (e.g. a date, or a unique ID) makes `.loc[]` lookups more
readable than the default integer index — see `pandas-selection-filtering.md`.

## Renaming Columns

```python
df.rename(columns={"old_name": "new_name"})     # returns a new DataFrame
df.columns = ["a", "b", "c"]                       # overwrite all column names at once, must match column count
df.columns = df.columns.str.strip().str.lower()      # normalize messy headers (whitespace, casing)
```

## Adding / Dropping Columns

```python
df["new_col"] = df["age"] * 2              # add a computed column
df["constant_col"] = "value"                  # add a column with the same value in every row

df.drop(columns=["col_to_remove"])               # drop a column, returns a new DataFrame
df.drop(columns=["col_to_remove"], inplace=True)    # drop in place, modifies df directly

df.drop(index=[0, 1])                                 # drop specific rows by index label
```
`inplace=True` modifies the DataFrame directly rather than returning a new
one — convenient, but current pandas guidance leans toward **avoiding
`inplace=True`** in most cases, since it can silently fail on a copy
(triggering a `SettingWithCopyWarning`) and makes method chaining harder.
Prefer `df = df.drop(...)` (reassignment) as the more current, predictable
pattern.

## Combining Operations (Method Chaining)

```python
result = (
    df
    .rename(columns={"old": "new"})
    .assign(doubled=lambda d: d["new"] * 2)
    .query("doubled > 10")
    .reset_index(drop=True)
)
```
- `.assign()` — adds/modifies columns without `inplace`, returns a new
  DataFrame, chainable
- `.query()` — filter rows using a string expression (see
  `pandas-selection-filtering.md` for more filtering patterns)

Method chaining like this is the current idiomatic pandas style for
multi-step transformations — easier to read top-to-bottom than repeated
`df = df.something()` reassignment lines, and avoids intermediate variable
clutter.

## Copies vs. Views (Same Caution as NumPy)

```python
sub_df = df[df["age"] > 25]        # this MAY be a view, MAY be a copy — ambiguous, pandas will warn you
sub_df["new_col"] = 1                  # can trigger SettingWithCopyWarning

sub_df = df[df["age"] > 25].copy()        # explicit .copy() — unambiguous, safe to modify
sub_df["new_col"] = 1                        # no warning, guaranteed independent
```
Same underlying issue as `numpy-basics-arrays.md`'s views/copies section —
when you intend to modify a filtered/sliced subset independently of the
original DataFrame, call `.copy()` explicitly to avoid ambiguity and the
`SettingWithCopyWarning`.