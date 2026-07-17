# R Reference — Data Structures

Docs: https://cran.r-project.org/doc/manuals/r-release/R-intro.html

Prerequisite: `r-basics-syntax.md`

## Vectors — 1D, Single Type (R's Fundamental Data Structure)

```r
v <- c(1, 2, 3, 4, 5)              # c() = "combine" — creates a vector, used CONSTANTLY in R
names_v <- c("a", "b", "c")

length(v)               # 5
v[1]                        # 1 — R IS 1-INDEXED, not 0-indexed (see r-basics-syntax.md)
v[c(1, 3)]                     # c(1, 3) selects elements 1 and 3 — array([1, 3])... actually returns c(1, 3) values
v[-1]                              # ALL elements EXCEPT the first — negative indexing means "exclude", not
                                      # "count from the end" like Python
v[2:4]                                 # elements 2 through 4, inclusive
```
Vectors are the foundation almost everything else in R builds on — even a
single value like `x <- 5` is technically a length-1 vector. All elements
in a vector must share the same type (numeric, character, logical, etc.) —
mixing types causes R to silently "upcast" everything to the most general
common type (e.g. `c(1, "a")` becomes all character/string).

## Vectorized Operations

```r
v <- c(1, 2, 3, 4, 5)
v * 2            # c(2, 4, 6, 8, 10) — applies to every element, no loop needed
v + c(10, 20, 30, 40, 50)     # element-wise addition between two same-length vectors

v[v > 2]            # c(3, 4, 5) — boolean/logical indexing, same concept as numpy/pandas filtering
```
This vectorization is conceptually identical to NumPy's broadcasting (see
`numpy-operations-broadcasting.md`) — operations apply across the whole
vector at once rather than requiring an explicit loop, and is the idiomatic
way to write R code.

## Lists — 1D, Mixed Types Allowed

```r
my_list <- list(name = "Adam", age = 30, active = TRUE)

my_list$name             # "Adam" — access by name with $
my_list[["age"]]            # 30 — access by name with [[ ]], also works with position: my_list[[2]]
my_list["age"]                 # returns a LIST containing just that element (note: single brackets)
```
`[[ ]]` extracts the actual value; `[ ]` returns a sub-list containing that
element — a distinction that trips up a lot of newcomers. Rule of thumb:
use `[[ ]]` or `$` when you want the value itself, `[ ]` when you want to
keep the list structure (e.g. selecting multiple named elements at once).

## Matrices — 2D, Single Type

```r
m <- matrix(1:6, nrow = 2, ncol = 3)
#      [,1] [,2] [,3]
# [1,]    1    3    5
# [2,]    2    4    6

m[1, 2]           # 3 — row 1, column 2
m[1, ]               # entire row 1
m[, 2]                  # entire column 2

t(m)                       # transpose
m %*% t(m)                     # matrix multiplication (%*%, distinct from element-wise * which is just * )
```
Matrices fill column-by-column by default (`byrow = FALSE`) — a common
surprise coming from other languages; pass `byrow = TRUE` to
`matrix()` if you intended row-by-row filling instead.

## Data Frames — 2D, Mixed Types Per Column (Like a pandas DataFrame)

```r
df <- data.frame(
  name = c("Adam", "Beth", "Carl"),
  age = c(30, 25, 40),
  stringsAsFactors = FALSE          # important on older R versions — see note below
)

df$name              # column access with $, returns a vector — like df["name"] in pandas
df[["age"]]              # equivalent to df$age
df[1, ]                     # first row, all columns
df[, "age"]                    # 'age' column, all rows
df[1:2, c("name", "age")]         # subset of rows AND columns

nrow(df)          # number of rows
ncol(df)             # number of columns
str(df)                 # structure — column names, types, sample values (R's equivalent of pandas' df.info())
summary(df)                 # summary statistics per column (R's equivalent of pandas' df.describe())
head(df)                        # first 6 rows by default
```
`stringsAsFactors = FALSE` was required in R versions before 4.0 to prevent
strings from being automatically converted to factors (R's categorical
type, see below) — as of R 4.0+, `FALSE` is now the default behavior, so
this argument is largely a legacy habit at this point, though still
harmless to include explicitly for clarity or when supporting older R
versions.

See `r-tidyverse-dplyr.md` for the far more common, idiomatic way to work
with data frames in modern R (via `dplyr`) rather than this base-R syntax.

## Factors — R's Categorical Data Type

```r
f <- factor(c("low", "medium", "high", "low"))
levels(f)                # "high" "low" "medium" — alphabetical by default, often NOT the order you want

f <- factor(c("low", "medium", "high"), levels = c("low", "medium", "high"), ordered = TRUE)
f < "high"                  # TRUE/FALSE comparisons now respect the specified order
```
Factors are R's mechanism for categorical variables, conceptually similar
to pandas' `category` dtype (`pandas-data-cleaning.md`) or scikit-learn's
`OrdinalEncoder` (`sklearn-preprocessing.md`) when `ordered = TRUE`.
Explicitly specifying `levels` in the intended order is important whenever
the category has a meaningful order (like the "low"/"medium"/"high"
example) — without it, R defaults to alphabetical ordering, which is often
wrong for ordinal data.

## NULL, NA, and NaN — Three Distinct "Missing" Concepts

```r
is.null(x)         # TRUE if x is NULL (no value/object at all)
is.na(x)              # TRUE if x is NA (a missing value within a data structure)
is.nan(x)                # TRUE specifically for NaN (result of undefined math, e.g. 0/0)

sum(c(1, 2, NA, 4))           # NA — any NA in the calculation propagates by default
sum(c(1, 2, NA, 4), na.rm = TRUE)     # 7 — na.rm=TRUE tells the function to ignore NAs
```
`na.rm = TRUE` is a required argument across most R summary functions
(`sum()`, `mean()`, `sd()`, etc.) whenever missing values are present —
forgetting it is one of the most common sources of unexpected `NA` results
in R, since R doesn't silently skip missing values the way some other
tools might.

## Type Conversion

```r
as.numeric("5")         # 5
as.character(5)            # "5"
as.integer(5.7)               # 5 — truncates, does NOT round
as.logical("TRUE")               # TRUE
as.factor(c("a", "b"))              # convert to factor
```