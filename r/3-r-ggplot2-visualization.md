# R Reference — ggplot2 Visualization

Docs: https://ggplot2.tidyverse.org/

Prerequisite: `r-tidyverse-dplyr.md`

```r
library(ggplot2)      # or library(tidyverse), which includes it
```

## The Grammar of Graphics (Core Concept)

ggplot2 builds plots by layering components rather than calling a single
"make this chart type" function — this is a fundamentally different mental
model from most other plotting libraries. Every ggplot2 plot has:
- **Data** — a data frame/tibble
- **Aesthetics (`aes`)** — mapping data columns to visual properties (x, y,
  color, size, etc.)
- **Geometry (`geom_*`)** — the visual representation (points, lines, bars)
- Optionally: statistical transformations, scales, facets, themes

```r
ggplot(data = df, aes(x = age, y = income)) +
  geom_point()
```
Layers are added with `+` (not the pipe `%>%`/`|>`) — this is a common
point of confusion for people used to dplyr's pipe-chaining style, since
ggplot2 predates the pipe operator's adoption and uses its own `+`-based
layering syntax instead.

## Common Geoms (Chart Types)

```r
ggplot(df, aes(x = age, y = income)) + geom_point()          # scatter plot
ggplot(df, aes(x = date, y = sales)) + geom_line()              # line chart
ggplot(df, aes(x = category)) + geom_bar()                         # bar chart (counts)
ggplot(df, aes(x = category, y = sales)) + geom_col()                 # bar chart (explicit values, not counts)
ggplot(df, aes(x = age)) + geom_histogram(binwidth = 5)                  # histogram
ggplot(df, aes(x = category, y = income)) + geom_boxplot()                  # box plot
ggplot(df, aes(x = income)) + geom_density()                                   # density plot
ggplot(df, aes(x = age, y = income)) + geom_smooth(method = "lm")                 # trend line/regression fit
```
`geom_bar()` vs `geom_col()` is a common early confusion: `geom_bar()`
counts occurrences of each x value automatically, while `geom_col()`
expects you to already have the y-values you want plotted (like a
pre-aggregated `summarise()` result from `r-tidyverse-dplyr.md`).

## Combining Multiple Geoms (Layering)

```r
ggplot(df, aes(x = age, y = income)) +
  geom_point() +
  geom_smooth(method = "lm", se = TRUE)
```
`se = TRUE` (default) shades a confidence interval band around the fitted
line — set `se = FALSE` to show just the line.

## Mapping Additional Variables (Color, Size, Shape)

```r
ggplot(df, aes(x = age, y = income, color = city)) +
  geom_point()

ggplot(df, aes(x = age, y = income, color = city, size = years_employed)) +
  geom_point(alpha = 0.6)      # alpha = transparency, useful for overlapping points
```
Putting `color = city` inside `aes()` means color is *data-driven*
(mapped from the `city` column); putting it outside `aes()` (e.g.
`geom_point(color = "blue")`) sets a single fixed color for every point —
mixing these two up is one of the most common ggplot2 mistakes.

## Faceting (Small Multiples)

```r
ggplot(df, aes(x = age, y = income)) +
  geom_point() +
  facet_wrap(~ city)                # separate panel per city, auto-arranged grid

ggplot(df, aes(x = age, y = income)) +
  geom_point() +
  facet_grid(gender ~ city)            # grid arrangement: rows by gender, columns by city
```
Faceting is ggplot2's built-in mechanism for "one small chart per
category" — a very idiomatic R/ggplot2 pattern for comparing subgroups
without manually looping and creating separate plots.

## Labels & Titles

```r
ggplot(df, aes(x = age, y = income)) +
  geom_point() +
  labs(
    title = "Income by Age",
    subtitle = "Sample dataset",
    x = "Age (years)",
    y = "Income ($)",
    caption = "Source: internal data"
  )
```

## Themes

```r
ggplot(df, aes(x = age, y = income)) +
  geom_point() +
  theme_minimal()          # clean, minimal gridlines — a common default choice
```
Other built-in themes: `theme_bw()`, `theme_classic()`, `theme_light()`,
`theme_dark()`. Custom fine-grained control via `theme()`:
```r
+ theme(
    axis.text.x = element_text(angle = 45, hjust = 1),      # rotate x-axis labels
    legend.position = "bottom",
    plot.title = element_text(face = "bold", size = 16)
  )
```

## Scales (Controlling Axes, Colors)

```r
+ scale_x_continuous(limits = c(0, 100))
+ scale_y_log10()                             # log-scale the y-axis
+ scale_color_manual(values = c("NYC" = "blue", "LA" = "red"))    # explicit color mapping
+ scale_color_brewer(palette = "Set2")            # use a predefined ColorBrewer palette
```

## Saving a Plot

```r
p <- ggplot(df, aes(x = age, y = income)) + geom_point()

ggsave("plot.png", plot = p, width = 8, height = 6, dpi = 300)
```
`ggsave()` automatically infers the file format from the extension (`.png`,
`.pdf`, `.svg`, `.jpg`) — the standard way to export a ggplot2 chart rather
than relying on RStudio's manual "Export" button, since it's scriptable
and reproducible.

## A Full Example (Piping Data Prep Into a Plot)

```r
df %>%
  filter(!is.na(income)) %>%
  group_by(city) %>%
  summarise(avg_income = mean(income), .groups = "drop") %>%
  ggplot(aes(x = reorder(city, avg_income), y = avg_income)) +
  geom_col(fill = "steelblue") +
  coord_flip() +                     # horizontal bars — often more readable for long category labels
  labs(title = "Average Income by City", x = NULL, y = "Average Income ($)") +
  theme_minimal()
```
Note the pipe (`%>%`) is used for the dplyr data-prep steps, then the final
result is piped directly into `ggplot()` — at that point, subsequent
layers switch to `+`. This mixed `%>%` then `+` pattern is standard and
worth getting comfortable with, since it reflects ggplot2's older,
independent syntax coexisting with the newer tidyverse pipe convention.
`reorder(city, avg_income)` sorts the bars by value rather than
alphabetically — a common, worthwhile touch for bar charts.