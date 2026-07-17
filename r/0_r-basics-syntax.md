# R Reference — Basics & Syntax

Docs: https://cran.r-project.org/manuals.html | https://www.tidyverse.org/

## Running R

```bash
sudo apt install r-base          # install R on Ubuntu
R                                    # launch the interactive R console (REPL)
Rscript my_script.R                    # run a script non-interactively from the terminal
```
RStudio (a dedicated IDE) is the most common way people actually work with
R day-to-day — worth installing separately if doing serious R work:
https://posit.co/download/rstudio-desktop/

## Variables & Assignment

```r
x <- 5              # the conventional assignment operator in R — read as "x gets 5"
x = 5                  # also works, but <- is the idiomatic R convention (unlike Python's plain =)
y <<- 10                  # global assignment — sets a variable in the global environment even from inside a function
                             # (rarely needed/recommended in normal code — a common source of hard-to-trace bugs)

5 -> x                        # right-assignment also works (rare, but valid) — same effect as x <- 5
```
`<-` vs `=` is one of R's most distinctive syntax quirks for anyone coming
from Python — both work for basic assignment, but `<-` is the convention
you'll see in virtually all R code/documentation, and is required in a few
specific contexts (e.g. inside function argument defaults, `=` means
something different there — named argument matching, not assignment).

## Basic Data Types

```r
x <- 5              # numeric (double, by default — R doesn't distinguish int/float by default)
x <- 5L                # integer (the L suffix forces integer type)
x <- "hello"              # character (string)
x <- TRUE                   # logical (boolean) — T/F also work as shorthand, but TRUE/FALSE is the safer convention
x <- 2+3i                      # complex
x <- NULL                         # absence of a value
x <- NA                              # missing value (distinct from NULL — see below)

class(x)          # check the type/class of a value
typeof(x)            # check the underlying storage type (more granular than class())
```
`NA` vs `NULL` is an important R-specific distinction: `NA` represents a
missing value *within* a data structure (e.g. one missing entry in a
vector of otherwise valid numbers), while `NULL` represents the complete
absence of a value/object. A vector can contain `NA`s but can't meaningfully
contain a `NULL` as one of its elements.

## Operators

```r
5 + 3     # 8
5 - 3       # 2
5 * 3         # 15
5 / 3           # 1.666...
5 %/% 3            # 1 — integer division
5 %% 3               # 2 — modulo
5 ^ 2                  # 25 — exponent (^ or ** both work)

# Comparison: ==, !=, <, >, <=, >=
# Logical: & (element-wise AND), | (element-wise OR), ! (NOT)
# && and || — SINGLE-value logical AND/OR, used in if() conditions specifically,
#             NOT for element-wise vector operations (a common source of confusion —
#             use & / | on vectors, && / || only for single TRUE/FALSE conditions)
```

## Control Flow

```r
x <- 10

if (x > 5) {
  print("big")
} else if (x == 5) {
  print("medium")
} else {
  print("small")
}

# Vectorized ternary-like alternative — very idiomatic R
label <- ifelse(x > 5, "big", "small")
```
`ifelse()` is the vectorized version — it applies element-wise across an
entire vector at once, similar in spirit to `np.where()` in
`numpy-operations-broadcasting.md`, and is generally preferred over a
manual loop with `if`/`else` when working with vectors.

```r
for (i in 1:5) {
  print(i)
}

i <- 0
while (i < 5) {
  print(i)
  i <- i + 1
}
```
`1:5` creates a sequence (integer vector `1, 2, 3, 4, 5`) — R's equivalent
of Python's `range()`, but inclusive of both endpoints.

## Functions

```r
greet <- function(name, greeting = "Hello") {
  paste(greeting, ", ", name, "!", sep = "")
}

greet("Adam")                    # "Hello, Adam!"
greet("Adam", greeting = "Hi")      # "Hi, Adam!"
```
- Functions are assigned to a name with `<-`, just like any other value —
  R treats functions as first-class objects
- The last evaluated expression in a function is automatically returned —
  an explicit `return()` is optional (though often used for clarity,
  especially for early returns)
- Default arguments work the same conceptual way as Python (see
  `python-control-flow-functions.md`)

```r
add <- function(a, b) {
  return(a + b)      # explicit return — needed for an EARLY return, optional otherwise
}
```

## String Basics

```r
paste("Hello", "World")             # "Hello World" — joins with a space by default
paste0("Hello", "World")               # "HelloWorld" — no separator, shorthand for paste(..., sep="")
paste("a", "b", "c", sep = "-")           # "a-b-c" — custom separator

sprintf("Name: %s, Age: %d", "Adam", 30)     # C-style string formatting
nchar("hello")                                  # 5 — string length
toupper("hello")                                   # "HELLO"
tolower("HELLO")                                      # "hello"
substr("hello world", 1, 5)                              # "hello" — substring (1-indexed, inclusive both ends)
```
Note R's **1-based indexing** throughout (`substr` above, vector indexing
in `r-data-structures.md`) — a fundamental difference from Python's
0-based indexing, and a very common source of off-by-one bugs when
switching between the two languages.

## Comments

```r
# single-line comment only — R has no built-in multi-line comment syntax
```

## Getting Help

```r
?mean            # open documentation for a function
help(mean)          # equivalent, longer form
??regression           # fuzzy/keyword search across all installed package documentation
```

## Installing & Loading Packages

```r
install.packages("dplyr")      # install a package from CRAN (one-time, or after an update)
library(dplyr)                    # load an installed package into the current session (needed every session)
```
See `r-tidyverse-dplyr.md` and `r-ggplot2-visualization.md` for the most
commonly used packages in a typical data science workflow.