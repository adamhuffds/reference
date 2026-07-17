# R Reference — Tidyverse & dplyr

Docs: https://dplyr.tidyverse.org/ | https://www.tidyverse.org/

Prerequisite: `r-data-structures.md`

## What the Tidyverse Is

The tidyverse is a collection of R packages designed to work together
around a shared philosophy ("tidy data") and consistent syntax —
`dplyr` (data manipulation), `ggplot2` (visualization, see
`r-ggplot2-visualization.md`), `tidyr` (reshaping), `readr` (I/O), and
others. This is the current standard, idiomatic way to work with data in R
— base R syntax (as shown in `r-data-structures.md`) still works, but
most modern R code and tutorials default to tidyverse style.

```r
install.packages("tidyverse")     # installs the whole collection at once
library(tidyverse)                   # loads dplyr, ggplot2, tidyr, readr, etc. all together
```

## The Pipe Operator

```r
result <- df %>%
  filter(age > 25) %>%
  select(name, age) %>%
  arrange(desc(age))
```
`%>%` (the "pipe," from the `magrittr` package, loaded automatically with
tidyverse) passes the result of the expression on its left as the first
argument to the function on its right — `df %>% filter(age > 25)` is
equivalent to `filter(df, age > 25)`. This chaining style is the dominant
idiom in modern R, directly analogous to pandas method chaining
(`.query().assign()...`) in `pandas-basics-series-dataframes.md`.

R 4.1+ also has a **built-in** native pipe, `|>`, which doesn't require
loading `magrittr`/tidyverse:
```r
result <- df |> filter(age > 25) |> select(name, age)
```
`|>` is functionally similar to `%>%` for most common cases and is the
current base-R recommended approach going forward, though `%>%` remains
extremely common in existing code/tutorials and has some additional
flexibility (e.g. placeholder `.` for non-first-argument piping) that
`|>` doesn't fully replicate. Either is fine to use; know both since
you'll encounter both in the wild.

## Core dplyr Verbs

### `filter()` — Select Rows
```r
df %>% filter(age > 25)
df %>% filter(age > 25 & city == "NYC")      # & / | work the same as base R, see r-basics-syntax.md
df %>% filter(city %in% c("NYC", "LA"))         # membership check, like pandas' .isin()
```

### `select()` — Select Columns
```r
df %>% select(name, age)              # keep only these columns
df %>% select(-age)                      # drop this column (negative selection)
df %>% select(starts_with("col_"))          # select by name pattern
df %>% select(where(is.numeric))               # select by column type
```

### `mutate()` — Add/Modify Columns
```r
df %>% mutate(age_in_months = age * 12)
df %>% mutate(
  age_group = ifelse(age > 30, "older", "younger"),
  is_adult = age >= 18
)
```
`mutate()` is the tidyverse equivalent of pandas' `.assign()`
(`pandas-basics-series-dataframes.md`) — adds new columns without
modifying the original data frame in place, chainable via the pipe.

### `arrange()` — Sort Rows
```r
df %>% arrange(age)              # ascending
df %>% arrange(desc(age))           # descending
df %>% arrange(city, desc(age))        # multiple columns, mixed order
```

### `summarise()` / `summarize()` — Aggregate (Both Spellings Work)
```r
df %>% summarise(avg_age = mean(age), total = n())
```
`n()` — a special dplyr function returning the count of rows in the
current group — only valid inside `summarise()`/`mutate()` calls.

### `group_by()` + `summarise()` — The R Equivalent of SQL GROUP BY / pandas groupby
```r
df %>%
  group_by(city) %>%
  summarise(
    avg_age = mean(age),
    count = n(),
    max_age = max(age)
  )
```
Same split-apply-combine concept as `pandas-groupby-merge-reshape.md`'s
`groupby()` and `sql-queries-basics.md`'s `GROUP BY` — worth leaning on
that existing mental model rather than relearning it from scratch.

## Joining Data Frames

```r
inner_join(orders, customers, by = "customer_id")
left_join(orders, customers, by = "customer_id")
right_join(orders, customers, by = "customer_id")
full_join(orders, customers, by = "customer_id")

left_join(df1, df2, by = c("id" = "customer_id"))    # different column names on each side
```
Directly parallel to `pandas.merge()`'s `how=` parameter
(`pandas-groupby-merge-reshape.md`) and SQL JOINs
(`sql-queries-joins-advanced.md`) — same join semantics, different syntax.

## Handling Missing Data

```r
df %>% filter(!is.na(col))          # drop rows where col is missing
df %>% drop_na()                       # drop rows with ANY missing value (tidyr function)
df %>% drop_na(col1, col2)                # only consider these specific columns

df %>% mutate(col = replace_na(col, 0))      # fill missing values with 0 (tidyr function)
```

## Reshaping with tidyr

```r
# Wide -> Long (like pandas' melt())
df %>% pivot_longer(cols = c(jan, feb, mar), names_to = "month", values_to = "sales")

# Long -> Wide (like pandas' pivot())
df %>% pivot_wider(names_from = month, values_from = sales)
```
Directly parallel to `pandas-groupby-merge-reshape.md`'s `.melt()`/
`.pivot()` — `pivot_longer`/`pivot_wider` replaced the older `gather()`/
`spread()` functions (still seen in older tidyverse code, but considered
legacy — current documentation recommends `pivot_longer`/`pivot_wider`).

## String Manipulation with `stringr`

```r
str_detect(df$name, "Ad")          # TRUE/FALSE per element — pattern found?
str_replace(df$name, "old", "new")    # replace first match
str_to_upper(df$name)                    # uppercase
str_trim(df$name)                           # remove leading/trailing whitespace
str_split(df$name, " ")                        # split into a list of character vectors
```
`stringr` functions are generally preferred over base R's string functions
(`grepl`, `gsub`, `toupper`, etc.) in tidyverse-style code for consistent
argument ordering (data first, matching the pipe convention) and more
predictable behavior.

## Reading Data with `readr`

```r
df <- read_csv("data.csv")            # tidyverse's CSV reader — faster and more predictable
                                          # type-guessing than base R's read.csv()
write_csv(df, "output.csv")
```
`readr`'s `read_csv()` (tidyverse) vs. base R's `read.csv()` — note the
underscore vs. no separator naming difference, and that `read_csv()`
returns a `tibble` (tidyverse's enhanced data frame) rather than a plain
base R `data.frame`. Tibbles behave almost identically to data frames but
print more concisely and have somewhat stricter, more predictable
subsetting behavior — the current recommended default within a tidyverse
workflow.

## Full Example: A Typical Chained Workflow

```r
result <- df %>%
  filter(!is.na(age)) %>%
  mutate(age_group = ifelse(age >= 30, "30+", "under 30")) %>%
  group_by(age_group, city) %>%
  summarise(avg_income = mean(income), count = n(), .groups = "drop") %>%
  arrange(desc(avg_income))
```
`.groups = "drop"` in `summarise()` explicitly removes the grouping
structure from the result afterward — worth including deliberately, since
leaving groups active can cause confusing behavior in later pipe steps
that expect an ungrouped data frame.