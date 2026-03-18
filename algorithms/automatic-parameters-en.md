---
title: Automatic calculation of the parameters
toc-title: Table of contents
---

# Automatic calculation of the parameters

:::: {.cell execution_count="1"}
::: {.cell-output .cell-output-stdout}
    Impossible to load package redis
:::
::::

## Introduction

This document describes the key parameters used to characterize an
elevator (lift) and detect its operational events from acceleration
signals. These parameters allow a `Record` to correctly identify
**trips** (movement between floors) and **door operations** based on
features extracted from vertical and horizontal acceleration data.

------------------------------------------------------------------------

## Parameters

Each lift may be characterized by a set of parameters. These parameters
need to be passed to a `Record` in order to correctly identify trips and
door operations.

### `min_z_peak`

**Unit:** $\frac{m}{s^2}$

If the (smoothed) vertical acceleration exceeds this value, it means
that the lift is accelerating or decelerating. After detecting these
peaks, the start and the end of a trip are determined using a method
described in `missing reference`.

------------------------------------------------------------------------

### `min_door_len`

**Unit:** $s$

Minimum duration of a door operation.

All vibrations that exceed the required amplitude threshold but last
shorter than `min_door_len` are discarded.

------------------------------------------------------------------------

### `doors_threshold`

**Unit:** $\frac{m}{s^2}$

Minimum amplitude of horizontal acceleration after preprocessing (taking
the absolute value and smoothing).

------------------------------------------------------------------------

### `mean_acc_z`

**Unit:** $\frac{m}{s^2}$

Mean value of the vertical acceleration. Ideally, it should be equal to
the gravitational acceleration of the Earth.

It is required for further signal processing, since it needs to be
subtracted so that the signal oscillates around zero.

------------------------------------------------------------------------

# Calculation Methods

## `min_z_peak`

The assumption underlying the calculation is that vibrations can be
grouped into three separate classes:

-   Oscillations close to zero (background noise)
-   Other distinguishable vibrations (e.g. door openings)
-   Peaks during acceleration and deceleration
-   All of the above but with negative sign

These classes are identified using the **k-means clustering algorithm**.

The value of `min_z_peak` is determined by taking a safe fraction of the
mean of the highest cluster. Currently, it is set to **75% of the
cluster mean**.

------------------------------------------------------------------------

## `min_door_len`

This value is calculated as the **mean duration of all intervals**
during which the horizontal acceleration exceeds the `doors_threshold`.

------------------------------------------------------------------------

## `doors_threshold`

Its calculation is based on horizontal vibrations.

First, the signal is transformed by taking its **absolute value** and
applying **moving average smoothing**. If the signal exceeds the
threshold value for at least `min_door_len` seconds, the segment is
considered a **door operation**.

The threshold is calculated using the mean and standard deviation of the
smoothed signal:

``` python
doors_threshold = np.mean(self.y_smooth) + 0.25 * self.std_y_smooth
```

An example of the smoothed signal during a door opening can be seen in
the figure below as highlighted in blue. The threshold is represented as
the dashed line.

:::: {.cell execution_count="2"}
::: {.cell-output .cell-output-display}
![](automatic-parameters-en_files/figure-markdown/cell-3-output-1.png)
:::
::::

## Audit log
| Date | User | Descripption |
|----------|----------|----------|
| 16.03.2026 | Radosław Lech | First version |