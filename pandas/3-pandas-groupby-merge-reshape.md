# pandas Reference — Data Cleaning

Docs: https://pandas.pydata.org/docs/user_guide/missing_data.html

Prerequisite: `pandas-basics-series-dataframes.md`, `pandas-selection-filtering.md`

## Detecting Missing Data

```python
df.isnull()                # DataFrame of True/False, element-wise
df.isnull().sum()             # count of missing values PER COLUMN — usually the first check to run
df.isnull().sum().sum()          # total missing values across the whole DataFrame
df.notnull()                        # inverse of isnull()

df["col"].isnull().any()               # True if this column has ANY missing values
```
`isna()`/`notna()` are exact aliases for `isnull()`/`notnull()` — both are
current, interchangeable, just pick one convention and stay consistent.

## Handling Missing Data

```python
df.dropna()                       # drop any row with AT LEAST ONE missing value
df.dropna(subset=["col1"])           # drop rows only where 'col1' specifically is missing
df.dropna(axis=1)                       # drop COLUMNS with any missing values, instead of rows
df.dropna(thresh=3)                        # keep rows with at least 3 non-null values

df.fillna(0)                                  # replace all missing values with 0
df.fillna({"col1": 0, "col2": "unknown"})        # different fill values per column
df["col"].fillna(df["col"].mean())                  # fill with the column's mean — common numeric imputation
df["col"].fillna(method="ffill")                       # forward-fill: propagate last valid value forward
df["col"].fillna(method="bfill")                          # backward-fill: propagate next valid value backward
```
Choosing `dropna()` vs `fillna()` is a domain decision, not a purely
technical one — dropping loses data (and can bias results if missingness
isn't random), filling introduces assumptions about what the missing value
"should" be. Worth deciding deliberately per column rather than applying
one blanket rule to an entire DataFrame.

## Duplicates

```python
df.duplicated()                  # boolean Series, True for rows that are duplicates of an earlier row
df.duplicated().sum()               # count of duplicate rows
df[df.duplicated()]                    # view the actual duplicate rows

df.drop_duplicates()                      # remove duplicate rows, keeps the FIRST occurrence by default
df.drop_duplicates(subset=["col1"])          # consider only 'col1' when identifying duplicates
df.drop_duplicates(keep="last")                 # keep the LAST occurrence instead of the first
```

## Type Conversion

```python
df["col"].astype(int)             # convert to integer
df["col"].astype(float)              # convert to float
df["col"].astype(str)                   # convert to string
df["col"].astype("category")               # convert to pandas' categorical type — saves memory for low-cardinality columns

pd.to_numeric(df["col"], errors="coerce")     # convert to numeric, invalid values become NaN instead of raising
pd.to_datetime(df["col"], errors="coerce")       # convert to datetime, invalid values become NaT
```
`errors="coerce"` is the standard pattern for real-world messy data — rather
than the conversion failing entirely on the first bad value, invalid
entries become `NaN`/`NaT` (pandas' missing-value markers), which you can
then inspect/handle explicitly with the missing-data tools above.

## String Cleaning (`.str` Accessor)

```python
df["name"].str.strip()             # remove leading/trailing whitespace
df["name"].str.lower()                # lowercase
df["name"].str.upper()                   # uppercase
df["name"].str.title()                      # Title Case
df["name"].str.replace("old", "new")           # substring replacement
df["name"].str.split(" ")                         # split into lists
df["name"].str.split(" ", expand=True)               # split into SEPARATE columns
df["name"].str.len()                                    # length of each string
df["email"].str.extract(r"@(\w+)\.com")                     # regex extraction into a new column
```
Column headers are a common target for cleaning too:
```python
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```

## Applying Custom Functions

```python
df["col"].apply(lambda x: x * 2)                  # apply a function to every value in a Series
df.apply(lambda row: row["a"] + row["b"], axis=1)     # apply across columns per row (axis=1 = row-wise)

df["col"].map({"yes": 1, "no": 0})                    # map values via a dict — Series-only, for simple substitutions
df.applymap(lambda x: x * 2)                              # apply element-wise across an ENTIRE DataFrame (deprecated
                                                             # name — see note below)
```
`applymap()` was renamed to `DataFrame.map()` in pandas 2.1+ — if you're on
a current pandas version, prefer `df.map(...)` over `df.applymap(...)`,
which is now deprecated (still works, but will eventually be removed).

`.apply()` runs a Python function per row/element and is meaningfully
slower than a vectorized operation — prefer built-in vectorized methods
(`.str.*`, arithmetic, `np.where`) whenever the same logic can be expressed
that way; reach for `.apply()` when the transformation genuinely needs
custom per-row logic that doesn't have a vectorized equivalent.

## Handling Outliers

```python
q1, q3 = df["col"].quantile([0.25, 0.75])
iqr = q3 - q1
lower, upper = q1 - 1.5 * iqr, q3 + 1.5 * iqr

df[(df["col"] >= lower) & (df["col"] <= upper)]      # filter to values within the IQR-based bounds
df["col"] = df["col"].clip(lower, upper)                # alternative: cap outliers rather than dropping rows
```
The 1.5×IQR rule is a common convention for flagging outliers, not a
universal statistical law — appropriate bounds vary by domain and dataset,
worth sanity-checking against domain knowledge rather than applying blindly.

## Replacing Values

```python
df["col"].replace("N/A", np.nan)                    # replace a specific value
df["col"].replace({"yes": True, "no": False})           # replace multiple specific values via a dict
df.replace(-999, np.nan)                                    # replace across the entire DataFrame
```