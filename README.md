# EDA Milestone Deliverable

**DS 201 — Principles of Data Science**

This assignment covers exercises 10.3.3 and 10.4.1 from Chapter 10 of R for Data Science.

---
"Exploratory Data Analysis with R — Milestone Deliverable"
author: "Jacob Dudas"
date: "November 13, 2026"
output:
  github_document:
    pandoc_args: ["--wrap=none"]
---

**Course:** DS 201 — Principles of Data Science  
**Reading:** [R for Data Science (2nd edition), Chapter 10](https://r4ds.hadley.nz/EDA.html)  
**Assigned sections:** 10.3.3 (Questions 1–4) and 10.4.1 (Questions 1–3)

For this assignment, I used R to answer the seven questions and included code and graphs to support my answers.

```{r setup, include=FALSE}
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE,
  fig.width = 8,
  fig.height = 4.5,
  fig.align = "center"
)
```

```{r packages}
library(tidyverse)
library(nycflights13)

theme_set(theme_minimal(base_size = 12))
```

## 10.3.3 Exercises — Variation (4 Questions)

### 10.3.3 — Question 1: Distributions of `x`, `y`, and `z`

**Question:** Explore the distribution of each of the `x`, `y`, and `z` variables in `diamonds`. What do you learn? Think about a diamond and how you might decide which dimension is the length, width, and depth.

I compared the three measurements and checked for unusual values.

```{r q1033-1-summary}
diamonds |>
  select(x, y, z) |>
  pivot_longer(everything(), names_to = "dimension", values_to = "mm") |>
  group_by(dimension) |>
  summarise(
    min_mm = min(mm),
    median_mm = median(mm),
    max_mm = max(mm),
    number_equal_to_zero = sum(mm == 0),
    number_over_20_mm = sum(mm > 20),
    .groups = "drop"
  ) |>
  knitr::kable(digits = 2, caption = "Diamond dimensions and potential outliers")
```

```{r q1033-1-full-distributions, fig.height=7}
diamonds |>
  pivot_longer(cols = c(x, y, z),
               names_to = "dimension", values_to = "mm") |>
  ggplot(aes(x = mm)) +
  geom_histogram(binwidth = 0.25, boundary = 0) +
  facet_wrap(~ dimension, ncol = 1) +
  labs(title = "Distributions of diamond dimensions (full range)",
       x = "Dimension in millimeters", y = "Number of diamonds")
```

I also zoomed in to get a better look at the main group of measurements.

```{r q1033-1-zoomed-distributions, fig.height=7}
diamonds |>
  pivot_longer(cols = c(x, y, z),
               names_to = "dimension", values_to = "mm") |>
  ggplot(aes(x = mm)) +
  geom_histogram(binwidth = 0.20, boundary = 0) +
  facet_wrap(~ dimension, ncol = 1, scales = "free_y") +
  coord_cartesian(xlim = c(0, 12)) +
  labs(title = "Diamond dimensions (zoomed to 0–12 mm)",
       x = "Dimension in millimeters", y = "Number of diamonds")
```

**Answer:** Based on the histograms, `x` and `y` have very similar measurements, while `z` is generally smaller. This makes sense because `x` and `y` represent the length and width, while `z` measures the depth of a diamond. I also found some measurements of zero and a few unusually large values, which likely indicate errors in the data.

### 10.3.3 — Question 2: Distribution of `price`

**Question:** Explore the distribution of `price`. Do you discover anything unusual or surprising? (Try a wide range of histogram `binwidth` values.)

I tried different bin widths to see how the price distribution changes.

```{r q1033-2-wide}
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 1000, boundary = 0) +
  labs(title = "Diamond prices, $1,000 bins",
       x = "Price (USD)", y = "Number of diamonds")
```

```{r q1033-2-medium}
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 100, boundary = 0) +
  labs(title = "Diamond prices, $100 bins",
       x = "Price (USD)", y = "Number of diamonds")
```

```{r q1033-2-fine}
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 5, boundary = 0) +
  coord_cartesian(xlim = c(1400, 1600)) +
  labs(title = "Diamond prices near $1,500, $5 bins",
       x = "Price (USD)", y = "Number of diamonds")
```

I also checked the prices in the gap.

```{r q1033-2-counts}
diamonds |>
  summarise(
    minimum_price = min(price),
    median_price = median(price),
    maximum_price = max(price),
    priced_1455_through_1545 = sum(between(price, 1455, 1545))
  ) |>
  knitr::kable(caption = "Price distribution and the gap around $1,500")
```

**Answer:** Most diamonds are priced on the lower end, with fewer diamonds at higher prices. When I used smaller bin widths, I noticed there were no diamonds priced between $1,455 and $1,545. This shows how changing the bin width can reveal patterns that are harder to see with larger bins.

### 10.3.3 — Question 3: 0.99 carat versus 1.00 carat

**Question:** How many diamonds are 0.99 carat? How many are 1 carat? What do you think causes the difference?

```{r q1033-3-counts}
diamonds |>
  filter(carat %in% c(0.99, 1.00)) |>
  count(carat, name = "number_of_diamonds") |>
  knitr::kable(caption = "Exact counts at 0.99 and 1.00 carats")
```

```{r q1033-3-plot}
diamonds |>
  filter(carat >= 0.90, carat <= 1.10) |>
  count(carat) |>
  ggplot(aes(x = carat, y = n)) +
  geom_col(width = 0.009) +
  scale_x_continuous(breaks = seq(0.90, 1.10, by = 0.02)) +
  labs(title = "Counts of diamonds near 1 carat",
       x = "Carat", y = "Number of diamonds")
```

**Answer:** There are **23 diamonds at 0.99 carat** compared with **1,558 at 1.00 carat**. I think this is because 1 carat is an important selling point, so diamond cutters may try to keep the weight at or above that amount. However, the data alone cannot confirm this.

### 10.3.3 — Question 4: `coord_cartesian()` versus `xlim()` and `ylim()`

**Question:** Compare and contrast `coord_cartesian()` versus `xlim()` or `ylim()` when zooming in on a histogram. What happens if you leave `binwidth` unset? What happens if you try to zoom so only half a bar shows?

First, I compared the zooming methods using the same bin width.

```{r q1033-4-fixed-coord}
price_hist <- ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 500, boundary = 0) +
  labs(x = "Price (USD)", y = "Number of diamonds")

price_hist +
  coord_cartesian(xlim = c(1150, 1650)) +
  labs(title = "coord_cartesian(): portions of original bars remain visible")
```

```{r q1033-4-fixed-xlim}
price_hist +
  xlim(1150, 1650) +
  labs(title = "xlim(): data outside the specified range are removed")
```

Next, I compared them without setting a bin width.

```{r q1033-4-auto-coord}
auto_hist <- ggplot(diamonds, aes(x = price)) +
  geom_histogram() +
  labs(x = "Price (USD)", y = "Number of diamonds")

auto_hist +
  coord_cartesian(xlim = c(1000, 5000)) +
  labs(title = "Automatic bins with coord_cartesian()")
```

```{r q1033-4-auto-xlim}
auto_hist +
  xlim(1000, 5000) +
  labs(title = "Automatic bins with xlim()")
```

Finally, I compared what happens when zooming in on the y-axis.

```{r q1033-4-y-coord}
carat_hist <- ggplot(diamonds, aes(x = carat)) +
  geom_histogram(binwidth = 0.10, boundary = 0) +
  labs(x = "Carat", y = "Number of diamonds")

carat_hist +
  coord_cartesian(ylim = c(0, 500)) +
  labs(title = "coord_cartesian(): tall bars are visually clipped")
```

```{r q1033-4-y-ylim}
carat_hist +
  ylim(0, 500) +
  labs(title = "ylim(): bars outside the y-scale may disappear")
```

**Answer:** `coord_cartesian()` zooms in without removing any data, while `xlim()` and `ylim()` can remove observations or bars outside their limits. If I do not set `binwidth`, R automatically uses 30 bins, and using `xlim()` can change how those bins are calculated. If the zoom cuts through a bar, `coord_cartesian()` shows only the visible part of it. Therefore, I would use `coord_cartesian()` to zoom in without changing the original histogram.

## 10.4.1 Exercises — Unusual Values (3 Questions)

### 10.4.1 — Question 1: Missing values in histograms and bar charts

**Question:** What happens to missing values in a histogram? What happens to missing values in a bar chart? Why are they treated differently?

I used a small example with missing values to compare the graphs.

```{r q1041-1-data}
example_data <- tibble(
  measurement = c(1, 2, 2, 3, NA_real_),
  category = c("A", "B", "A", NA_character_, "B")
)

example_data |>
  summarise(missing_measurements = sum(is.na(measurement)),
            missing_categories = sum(is.na(category))) |>
  knitr::kable(caption = "Missing-value counts in the example")

example_data |> count(category)
```

```{r q1041-1-histogram}
ggplot(example_data, aes(x = measurement)) +
  geom_histogram(binwidth = 1, boundary = 0, na.rm = TRUE) +
  labs(title = "Histogram: missing numerical values are omitted",
       x = "Measurement", y = "Count")
```

```{r q1041-1-bar-chart}
ggplot(example_data, aes(x = category)) +
  geom_bar() +
  scale_x_discrete(na.translate = TRUE) +
  labs(title = "Bar chart: missing category appears as NA",
       x = "Category", y = "Count")
```

**Answer:** Histograms leave out missing values because an `NA` cannot be placed into a numerical range. Bar charts can show missing values as their own `NA` category. This is because histograms group numerical values into intervals, while bar charts count categories.

### 10.4.1 — Question 2: `na.rm = TRUE` in `mean()` and `sum()`

**Question:** What does `na.rm = TRUE` do in `mean()` and `sum()`?

```{r q1041-2-na-rm}
values <- c(2, 4, NA, 8)

mean(values)                 # Returns NA
mean(values, na.rm = TRUE)   # (2 + 4 + 8) / 3 = 4.666667

sum(values)                  # Returns NA
sum(values, na.rm = TRUE)    # 2 + 4 + 8 = 14
```

**Answer:** Using `na.rm = TRUE` tells R to ignore missing values when calculating a mean or sum. Without it, both functions return `NA` in this example. When I use it, the mean is about **4.67** and the sum is **14**.

### 10.4.1 — Question 3: Scheduled departure times and cancelled flights

**Question:** Recreate the frequency plot of `scheduled_dep_time` colored by whether the flight was cancelled or not. Also facet by `cancelled`. Try different `scales` values to reduce the visual effect of the much larger non-cancelled group.

I converted the scheduled departure times to hours and labeled flights with missing departure times as cancelled.

```{r q1041-3-prepare-flights}
flights_eda <- nycflights13::flights |>
  mutate(
    cancelled = is.na(dep_time),
    sched_hour = sched_dep_time %/% 100,
    sched_min = sched_dep_time %% 100,
    scheduled_dep_time = sched_hour + sched_min / 60
  )

flights_eda |>
  count(cancelled) |>
  mutate(percent = round(100 * n / sum(n), 2)) |>
  knitr::kable(caption = "Counts of cancelled and non-cancelled flights")
```

First, I plotted both groups together.

```{r q1041-3-original-plot}
ggplot(flights_eda, aes(x = scheduled_dep_time, color = cancelled)) +
  geom_freqpoly(binwidth = 1/4, linewidth = 0.8) +
  scale_x_continuous(breaks = seq(0, 24, by = 3)) +
  labs(title = "Scheduled departure times by cancellation status",
       x = "Scheduled departure hour", y = "Number of flights",
       color = "Cancelled")
```

Next, I used separate graphs with the same scales.

```{r q1041-3-facet-fixed, fig.height=5.7}
ggplot(flights_eda, aes(x = scheduled_dep_time, color = cancelled)) +
  geom_freqpoly(binwidth = 1/4, linewidth = 0.8) +
  facet_wrap(~ cancelled, ncol = 1, scales = "fixed") +
  scale_x_continuous(breaks = seq(0, 24, by = 3)) +
  labs(title = "Faceted flight departure times: fixed scales",
       x = "Scheduled departure hour", y = "Number of flights",
       color = "Cancelled")
```

Then, I let each graph use its own y-axis.

```{r q1041-3-facet-free-y, fig.height=5.7}
ggplot(flights_eda, aes(x = scheduled_dep_time, color = cancelled)) +
  geom_freqpoly(binwidth = 1/4, linewidth = 0.8) +
  facet_wrap(~ cancelled, ncol = 1, scales = "free_y") +
  scale_x_continuous(breaks = seq(0, 24, by = 3)) +
  labs(title = "Faceted flight departure times: free y-scales",
       x = "Scheduled departure hour", y = "Number of flights",
       color = "Cancelled")
```

Finally, I let both axes vary.

```{r q1041-3-facet-free, fig.height=5.7}
ggplot(flights_eda, aes(x = scheduled_dep_time, color = cancelled)) +
  geom_freqpoly(binwidth = 1/4, linewidth = 0.8) +
  facet_wrap(~ cancelled, ncol = 1, scales = "free") +
  labs(title = "Faceted flight departure times: both scales free",
       x = "Scheduled departure hour", y = "Number of flights",
       color = "Cancelled")
```

**Answer:** There are many more non-cancelled flights than cancelled flights, which makes the cancelled-flight pattern difficult to see. With `scales = "fixed"`, both graphs use the same scales, while `scales = "free_y"` gives each graph its own y-axis. I think `free_y` works best because it makes both patterns easier to see while keeping the time axis the same. However, the heights cannot be directly compared since the y-axes are different.
