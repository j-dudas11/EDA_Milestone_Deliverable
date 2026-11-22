
---
title: "EDA Milestone Deliverable"
output:
  github_document:
    pandoc_args: ["--wrap=none"]
---

# EDA Milestone Deliverable

**DS 201 — Principles of Data Science**

**Reading:** [R for Data Science, Chapter 10](https://r4ds.hadley.nz/EDA.html)

**Assigned Sections:** 10.3.3 and 10.4.1

For this assignment, I used R to answer seven questions about exploratory data analysis. I included R code, graphs, and explanations to support my answers.

```{r setup, include=FALSE} id="aqr01"
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE,
  fig.width = 8,
  fig.height = 4.5,
  fig.align = "center"
)
```

```{r packages} id="aqr02"
library(tidyverse)
library(nycflights13)

theme_set(theme_minimal(base_size = 12))
```

# 10.3.3 Exercises — Variation

## Question 1: Distributions of x, y, and z

**Question:** Explore the distribution of each of the x, y, and z variables in diamonds. What do you learn?

I compared the three measurements and checked for unusual values.

```{r q1-summary} id="aqr03"
diamonds |>
  select(x, y, z) |>
  pivot_longer(
    everything(),
    names_to = "dimension",
    values_to = "mm"
  ) |>
  group_by(dimension) |>
  summarise(
    min_mm = min(mm),
    median_mm = median(mm),
    max_mm = max(mm),
    number_equal_to_zero = sum(mm == 0),
    number_over_20_mm = sum(mm > 20),
    .groups = "drop"
  ) |>
  knitr::kable(digits = 2)
```

### Plot 1: Full Distributions

```{r q1-full-distributions, fig.height=7} id="aqr04"
diamonds |>
  pivot_longer(
    cols = c(x, y, z),
    names_to = "dimension",
    values_to = "mm"
  ) |>
  ggplot(aes(x = mm)) +
  geom_histogram(binwidth = 0.25, boundary = 0) +
  facet_wrap(~ dimension, ncol = 1) +
  labs(
    title = "Distributions of Diamond Dimensions",
    x = "Dimension in Millimeters",
    y = "Number of Diamonds"
  )
```

### Plot 2: Zoomed Distributions

```{r q1-zoomed-distributions, fig.height=7} id="aqr05"
diamonds |>
  pivot_longer(
    cols = c(x, y, z),
    names_to = "dimension",
    values_to = "mm"
  ) |>
  ggplot(aes(x = mm)) +
  geom_histogram(binwidth = 0.20, boundary = 0) +
  facet_wrap(~ dimension, ncol = 1, scales = "free_y") +
  coord_cartesian(xlim = c(0, 12)) +
  labs(
    title = "Diamond Dimensions (0–12 mm)",
    x = "Dimension in Millimeters",
    y = "Number of Diamonds"
  )
```

**Answer:** Based on the histograms, x and y have very similar measurements, while z is generally smaller. This makes sense because x and y represent the length and width, while z measures the depth of a diamond. I also found some measurements of zero and a few unusually large values, which likely indicate errors in the data.

---

## Question 2: Distribution of Price

**Question:** Explore the distribution of price. Do you discover anything unusual or surprising?

I tried different bin widths to see how the price distribution changes.

### Plot 1: $1,000 Bin Width

```{r q2-wide} id="aqr06"
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 1000, boundary = 0) +
  labs(
    title = "Diamond Prices — $1,000 Bins",
    x = "Price (USD)",
    y = "Number of Diamonds"
  )
```

### Plot 2: $100 Bin Width

```{r q2-medium} id="aqr07"
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 100, boundary = 0) +
  labs(
    title = "Diamond Prices — $100 Bins",
    x = "Price (USD)",
    y = "Number of Diamonds"
  )
```

### Plot 3: $5 Bin Width

```{r q2-fine} id="aqr08"
ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 5, boundary = 0) +
  coord_cartesian(xlim = c(1400, 1600)) +
  labs(
    title = "Diamond Prices Near $1,500",
    x = "Price (USD)",
    y = "Number of Diamonds"
  )
```

### Summary of Diamond Prices

```{r q2-counts} id="aqr09"
diamonds |>
  summarise(
    minimum_price = min(price),
    median_price = median(price),
    maximum_price = max(price),
    priced_1455_through_1545 =
      sum(between(price, 1455, 1545))
  ) |>
  knitr::kable()
```

**Answer:** Most diamonds are priced on the lower end, with fewer diamonds at higher prices. When I used smaller bin widths, I noticed there were no diamonds priced between $1,455 and $1,545. This shows how changing the bin width can reveal patterns that are harder to see with larger bins.

---

## Question 3: 0.99 Carat vs. 1.00 Carat

**Question:** How many diamonds are 0.99 carat? How many are 1 carat? What do you think causes the difference?

### Diamond Counts

```{r q3-counts} id="aqr10"
diamonds |>
  filter(carat %in% c(0.99, 1.00)) |>
  count(carat, name = "number_of_diamonds") |>
  knitr::kable()
```

### Plot: Diamonds Near 1 Carat

```{r q3-plot} id="aqr11"
diamonds |>
  filter(carat >= 0.90, carat <= 1.10) |>
  count(carat) |>
  ggplot(aes(x = carat, y = n)) +
  geom_col(width = 0.009) +
  scale_x_continuous(
    breaks = seq(0.90, 1.10, by = 0.02)
  ) +
  labs(
    title = "Counts of Diamonds Near 1 Carat",
    x = "Carat",
    y = "Number of Diamonds"
  )
```

**Answer:** There are 23 diamonds at 0.99 carat compared with 1,558 at 1.00 carat. I think this is because 1 carat is an important selling point, so diamond cutters may try to keep the weight at or above that amount. However, the data alone cannot confirm this.

---

## Question 4: coord_cartesian() vs. xlim() and ylim()

**Question:** Compare and contrast coord_cartesian() versus xlim() and ylim() when zooming in on a histogram. What happens if binwidth is unset? What happens if only half a bar is visible?

First, I compared the zooming methods using the same bin width.

### Plot 1: coord_cartesian()

```{r q4-fixed-coord} id="aqr12"
price_hist <- ggplot(diamonds, aes(x = price)) +
  geom_histogram(binwidth = 500, boundary = 0) +
  labs(
    x = "Price (USD)",
    y = "Number of Diamonds"
  )

price_hist +
  coord_cartesian(xlim = c(1150, 1650)) +
  labs(title = "Zooming With coord_cartesian()")
```

### Plot 2: xlim()

```{r q4-fixed-xlim} id="aqr13"
price_hist +
  xlim(1150, 1650) +
  labs(title = "Zooming With xlim()")
```

Next, I compared the two methods without specifying a bin width.

### Plot 3: Automatic Bins With coord_cartesian()

```{r q4-auto-coord} id="aqr14"
auto_hist <- ggplot(diamonds, aes(x = price)) +
  geom_histogram() +
  labs(
    x = "Price (USD)",
    y = "Number of Diamonds"
  )

auto_hist +
  coord_cartesian(xlim = c(1000, 5000)) +
  labs(title = "Automatic Bins: coord_cartesian()")
```

### Plot 4: Automatic Bins With xlim()

```{r q4-auto-xlim} id="aqr15"
auto_hist +
  xlim(1000, 5000) +
  labs(title = "Automatic Bins: xlim()")
```

Finally, I compared the effects of zooming in on the y-axis.

### Plot 5: coord_cartesian() on Y-Axis

```{r q4-y-coord} id="aqr16"
carat_hist <- ggplot(diamonds, aes(x = carat)) +
  geom_histogram(binwidth = 0.10, boundary = 0) +
  labs(
    x = "Carat",
    y = "Number of Diamonds"
  )

carat_hist +
  coord_cartesian(ylim = c(0, 500)) +
  labs(title = "Y-Axis: coord_cartesian()")
```

### Plot 6: ylim() on Y-Axis

```{r q4-y-ylim} id="aqr17"
carat_hist +
  ylim(0, 500) +
  labs(title = "Y-Axis: ylim()")
```

**Answer:** coord_cartesian() zooms in without removing any data, while xlim() and ylim() can remove observations or bars outside their limits. If I do not set binwidth, R automatically uses 30 bins, and using xlim() can change how those bins are calculated. If the zoom cuts through a bar, coord_cartesian() shows only the visible part of it. Therefore, I would use coord_cartesian() to zoom in without changing the original histogram.

---

# 10.4.1 Exercises — Unusual Values

## Question 1: Missing Values in Histograms and Bar Charts

**Question:** What happens to missing values in a histogram? What happens to missing values in a bar chart? Why are they treated differently?

I used a small example with missing values to compare the graphs.

### Example Data

```{r q5-data} id="aqr18"
example_data <- tibble(
  measurement = c(1, 2, 2, 3, NA_real_),
  category = c("A", "B", "A", NA_character_, "B")
)

example_data |>
  summarise(
    missing_measurements = sum(is.na(measurement)),
    missing_categories = sum(is.na(category))
  ) |>
  knitr::kable()

example_data |> count(category)
```

### Plot 1: Histogram With Missing Values

```{r q5-histogram} id="aqr19"
ggplot(example_data, aes(x = measurement)) +
  geom_histogram(
    binwidth = 1,
    boundary = 0,
    na.rm = TRUE
  ) +
  labs(
    title = "Histogram: Missing Values Are Omitted",
    x = "Measurement",
    y = "Count"
  )
```

### Plot 2: Bar Chart With Missing Values

```{r q5-bar-chart} id="aqr20"
ggplot(example_data, aes(x = category)) +
  geom_bar() +
  scale_x_discrete(na.translate = TRUE) +
  labs(
    title = "Bar Chart: Missing Values Appear as NA",
    x = "Category",
    y = "Count"
  )
```

**Answer:** Histograms leave out missing values because an NA cannot be placed into a numerical range. Bar charts can show missing values as their own NA category. This is because histograms group numerical values into intervals, while bar charts count categories.

---

## Question 2: na.rm = TRUE

**Question:** What does na.rm = TRUE do in mean() and sum()?

```{r q6-na-rm} id="aqr21"
values <- c(2, 4, NA, 8)

# Mean without removing NA
mean(values)

# Mean after removing NA
mean(values, na.rm = TRUE)

# Sum without removing NA
sum(values)

# Sum after removing NA
sum(values, na.rm = TRUE)
```

**Answer:** Using na.rm = TRUE tells R to ignore missing values when calculating a mean or sum. Without it, both functions return NA in this example. When I use it, the mean is about 4.67 and the sum is 14.

---

## Question 3: Scheduled Departure Times and Cancelled Flights

**Question:** Recreate the frequency plot of scheduled departure times colored by whether a flight was cancelled. Also facet by cancelled and experiment with different scales.

I converted the scheduled departure times to hours and labeled flights with missing departure times as cancelled.

### Preparing the Flight Data

```{r q7-prepare-flights} id="aqr22"
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
  knitr::kable()
```

### Plot 1: Original Frequency Plot

```{r q7-original-plot} id="aqr23"
ggplot(
  flights_eda,
  aes(x = scheduled_dep_time, color = cancelled)
) +
  geom_freqpoly(
    binwidth = 1/4,
    linewidth = 0.8
  ) +
  scale_x_continuous(
    breaks = seq(0, 24, by = 3)
  ) +
  labs(
    title = "Scheduled Departure Times by Cancellation",
    x = "Scheduled Departure Hour",
    y = "Number of Flights",
    color = "Cancelled"
  )
```

### Plot 2: Fixed Scales

```{r q7-facet-fixed, fig.height=5.7} id="aqr24"
ggplot(
  flights_eda,
  aes(x = scheduled_dep_time, color = cancelled)
) +
  geom_freqpoly(
    binwidth = 1/4,
    linewidth = 0.8
  ) +
  facet_wrap(
    ~ cancelled,
    ncol = 1,
    scales = "fixed"
  ) +
  scale_x_continuous(
    breaks = seq(0, 24, by = 3)
  ) +
  labs(
    title = "Departure Times: Fixed Scales",
    x = "Scheduled Departure Hour",
    y = "Number of Flights",
    color = "Cancelled"
  )
```

### Plot 3: Free Y-Axis Scales

```{r q7-facet-free-y, fig.height=5.7} id="aqr25"
ggplot(
  flights_eda,
  aes(x = scheduled_dep_time, color = cancelled)
) +
  geom_freqpoly(
    binwidth = 1/4,
    linewidth = 0.8
  ) +
  facet_wrap(
    ~ cancelled,
    ncol = 1,
    scales = "free_y"
  ) +
  scale_x_continuous(
    breaks = seq(0, 24, by = 3)
  ) +
  labs(
    title = "Departure Times: Free Y-Axis",
    x = "Scheduled Departure Hour",
    y = "Number of Flights",
    color = "Cancelled"
  )
```

### Plot 4: Both Axes Free

```{r q7-facet-free, fig.height=5.7} id="aqr26"
ggplot(
  flights_eda,
  aes(x = scheduled_dep_time, color = cancelled)
) +
  geom_freqpoly(
    binwidth = 1/4,
    linewidth = 0.8
  ) +
  facet_wrap(
    ~ cancelled,
    ncol = 1,
    scales = "free"
  ) +
  labs(
    title = "Departure Times: Both Axes Free",
    x = "Scheduled Departure Hour",
    y = "Number of Flights",
    color = "Cancelled"
  )
```

**Answer:** There are many more non-cancelled flights than cancelled flights, which makes the cancelled-flight pattern difficult to see. With scales = "fixed", both graphs use the same scales, while scales = "free_y" gives each graph its own y-axis. I think free_y works best because it makes both patterns easier to see while keeping the time axis the same. However, the heights cannot be directly compared since the y-axes are different.
