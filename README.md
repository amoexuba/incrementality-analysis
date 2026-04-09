# Incrementality Analysis with NeuralProphet

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![NeuralProphet](https://img.shields.io/badge/Forecasting-NeuralProphet-2E8B57)
![Status](https://img.shields.io/badge/Use%20Case-ASO%20%26%20UA%20Incrementality-0A66C2)

Measure the true impact of ASO and user acquisition events by training a baseline forecast on historical data, comparing that forecast to actual performance, and quantifying the incremental lift.

The main workflow lives in [`incrementality-analysis-NeuralProphet-1.3.ipynb`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality-analysis-NeuralProphet-1.3.ipynb).

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Repository Contents](#repository-contents)
- [Sample Output](#sample-output)
- [Requirements](#requirements)
- [Expected Input Data](#expected-input-data)
- [Configuration](#configuration)
- [How to Run](#how-to-run)
- [Outputs](#outputs)
- [How to Interpret Results](#how-to-interpret-results)
- [Use Cases](#use-cases)
- [Notes](#notes)

## Overview

This notebook follows a forecast-based incrementality method:

1. Train a `NeuralProphet` model on pre-event history.
2. Forecast the expected baseline through the event and post-event period.
3. Compare baseline vs. actual observations.
4. Estimate absolute lift and percent lift.
5. Test whether the observed lift is statistically significant.

It is useful for daily time series such as installs, impressions, downloads, revenue, or other app growth metrics.

## Features

- In-notebook dependency installation for a quick start
- Synthetic demo data for an end-to-end dry run
- Real-data loading patterns for App Store Connect, Google Play Console, and AppTweak exports
- Configurable event windows and post-event measurement period
- `NeuralProphet` baseline forecasting with weekly and yearly seasonality
- 95% confidence interval handling when quantile outputs are available
- Incremental lift calculation at the daily and total level
- One-sample t-test significance check for event and post-event windows
- Exported chart and CSV output for reporting

## Repository Contents

- [`incrementality-analysis-NeuralProphet-1.3.ipynb`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality-analysis-NeuralProphet-1.3.ipynb): main notebook
- [`incrementality_result.png`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality_result.png): saved visualization
- [`incrementality_results.csv`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality_results.csv): exported day-level results

## Sample Output

### Forecast vs. Actuals

![Incrementality chart](incrementality_result.png)

The chart highlights:

- historical pre-event trend
- model baseline forecast
- 95% confidence band
- event window
- daily incremental lift bars

### Exported Results Preview

Example rows from [`incrementality_results.csv`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality_results.csv):

| ds | actual | baseline | lower_95 | upper_95 | lift_abs | lift_pct | period |
|---|---:|---:|---:|---:|---:|---:|---|
| 2024-11-05 | 2588.00 | 1356.19 | 1272.22 | 1520.07 | 1231.81 | 90.83 | event |
| 2024-11-06 | 2451.00 | 1216.26 | 1125.22 | 1386.07 | 1234.74 | 101.52 | event |
| 2024-11-07 | 2003.00 | 1142.36 | 1062.67 | 1291.53 | 860.65 | 75.34 | event |
| 2024-11-08 | 2280.00 | 1186.46 | 1113.86 | 1394.13 | 1093.54 | 92.17 | event |

## Requirements

The notebook installs these packages:

- `neuralprophet`
- `pandas`
- `numpy`
- `matplotlib`
- `scipy`

It uses a standard Python 3 Jupyter kernel and works with `Python 3.11.15`.

## Expected Input Data

The analysis expects a DataFrame with exactly two columns:

- `ds`: date column
- `y`: daily metric to analyze

Example:

```text
ds,y
2024-01-01,1523
2024-01-02,1487
2024-01-03,1602
```

The notebook includes commented loading examples for:

- App Store Connect CSV exports
- Google Play Console CSV exports
- AppTweak CSV exports

Synthetic demo data is included by default so you can run the notebook before wiring in production data.

## Configuration

The core parameters are defined in the event setup section:

```python
EVENT_DATE      = '2024-11-05'
EVENT_END       = '2024-11-12'
POST_EVENT_DAYS = 31
MODEL           = 'extrapolation'
```

Parameter guide:

- `EVENT_DATE`: first day of the ASO or UA event
- `EVENT_END`: last day of the event
- `POST_EVENT_DAYS`: extra number of days to track lingering impact
- `MODEL`: analysis mode

Supported modes:

- `extrapolation`: train only on pre-event data and forecast forward
- `interpolation`: train on pre-event plus far post-event data while excluding the event window

## How to Run

1. Open [`incrementality-analysis-NeuralProphet-1.3.ipynb`](https://github.com/amoexuba/incrementality-analysis/blob/main/incrementality-analysis-NeuralProphet-1.3.ipynb) in Jupyter or VS Code.
2. Run the dependency installation cell.
3. Keep the synthetic demo data or replace it with your own `ds` and `y` dataset.
4. Set `EVENT_DATE`, `EVENT_END`, `POST_EVENT_DAYS`, and `MODEL`.
5. Run the notebook through training, forecasting, significance testing, and export.

## Outputs

The notebook generates:

- `incrementality_result.png`: visualization of actuals, baseline, confidence interval, and incremental lift
- `incrementality_results.csv`: exported daily analysis results

CSV columns:

- `ds`
- `actual`
- `baseline`
- `lower_95`
- `upper_95`
- `lift_abs`
- `lift_pct`
- `period`

## How to Interpret Results

Key metrics printed by the notebook:

- average daily lift during the event window
- average lift percentage during the event window
- p-value for the event window
- average daily lift after the event
- p-value for the post-event period
- total incremental installs across the full analysis window

Interpretation guide:

- positive `lift_abs` means actual performance was above the estimated baseline
- negative `lift_abs` means actual performance came in below baseline
- lower p-values indicate stronger evidence that the observed lift is not random noise
- a p-value below `0.05` is typically treated as statistically significant in this notebook

## Use Cases

This workflow is useful for measuring the impact of:

- App Store Optimization metadata updates
- store listing experiments
- paid acquisition bursts
- feature launches
- campaign-related changes in installs, impressions, or revenue

## Notes

- The notebook contains a compatibility monkey-patch for `NeuralProphet` with newer `pandas` versions.
- Holiday effects can be added by uncommenting `model.add_country_holidays(...)`.
- The workflow assumes daily data with reasonably complete historical coverage.
- Missing dates are checked before modeling and should be addressed before relying on the results.
