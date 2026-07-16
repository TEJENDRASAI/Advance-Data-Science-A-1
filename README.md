# Advance-Data-Science
A 104-Week Stress Test of German Electricity-Load Forecasts

> Candidate: Tejendra Sontineni  
> Case-study window: January 2015 to September 2020  
> Primary artefacts: `Tejendra.ipynb` and `Tejendra.pdf`

This project investigates German electricity demand using statistical, machine-learning, and neural-network forecasting methods. The analysis begins with time-series diagnostics and simple benchmarks before progressing to SARIMA, SARIMAX, Gradient Boosting, Random Forest, and LSTM models.

The principal aim is to determine whether more complex models improve upon a strong annual seasonal-naive benchmark while also considering interpretability, data availability, and operational use.

## Repository contents

- `Tejendra.ipynb` - complete analysis, model training, evaluation, and visualisation
- `Tejendra.pdf` - eight-page report discussing the methods, results, limitations, and recommendations
- `README.md` - project setup and reproduction guidance

## Data

Electricity-demand observations are obtained from the Open Power System Data time-series collection:

- <https://data.open-power-system-data.org/time_series/>

The analysis selects Germany (`DE`) and retains observations from 1 January 2015 onward. Hourly demand is used for the LSTM experiment, while weekly mean demand is used for the benchmarks, SARIMA/SARIMAX, Gradient Boosting, and Random Forest models.

Daily Berlin temperature data are obtained from the Open-Meteo historical archive:

- <https://archive-api.open-meteo.com/v1/archive>

German public-holiday indicators are generated using the Python `holidays` package. Temperature values in the hold-out period are observed historical measurements, not archived weather forecasts. Models using these values should therefore be interpreted as conditional forecasting experiments.

## Analysis workflow

The notebook follows this sequence:

1. Load and validate German electricity-demand data.
2. Examine missing values, temporal coverage, and demand patterns.
3. Aggregate the series to daily and weekly frequencies.
4. inspect trend and annual seasonality using visualisation and decomposition.
5. Test stationarity with ADF and KPSS statistics.
6. Examine first and seasonal differences together with ACF and PACF behaviour.
7. Establish mean, naive, drift, and seasonal-naive benchmarks.
8. Search candidate SARIMA orders using training-sample AIC.
9. Add temperature, degree-day, and holiday regressors through SARIMAX.
10. Construct lagged-load, rolling-history, weather, holiday, and cyclical-calendar features.
11. Compare Gradient Boosting and Random Forest configurations.
12. Compare LSTM lookback windows using hourly demand.
13. Evaluate forecasts with RMSE, MAE, and MAPE.
14. Compare model behaviour, interpretability, and practical limitations.

## Models

| Model family | Purpose |
|---|---|
| Mean, naive, and drift | Provide simple reference forecasts |
| Seasonal naive | Test whether the corresponding week from the previous year is sufficient |
| SARIMA | Represent autoregressive, moving-average, differencing, and annual seasonal structure |
| SARIMAX | Examine whether weather and holidays provide additional explanatory information |
| Gradient Boosting | Learn nonlinear relationships among lagged demand and external features |
| Random Forest | Provide a second nonlinear ensemble model and feature-importance comparison |
| LSTM | Explore short-term sequential patterns in hourly electricity demand |

## Recorded results

The following values are stored in the executed notebook:

| Model | RMSE | MAE | MAPE (%) |
|---|---:|---:|---:|
| LSTM, weekly aggregation | 296.45 | 221.24 | 0.415 |
| Hourly LSTM | 2100.47 | 1525.76 | 2.89 |
| Random Forest | 2456.34 | 1799.71 | 3.45 |
| Gradient Boosting | 2484.77 | 1874.09 | 3.58 |
| Seasonal naive | 3006.76 | 2318.52 | 4.41 |
| SARIMAX, full weather and holidays | 4908.74 | 4012.85 | 7.62 |
| SARIMA | 9230.67 | 7924.20 | 14.89 |

These values should not all be treated as a single like-for-like ranking. The weekly statistical and feature models, hourly LSTM, and weekly aggregation of LSTM outputs operate at different frequencies and evaluation conditions. In addition, later LSTM test predictions use recently observed demand when constructing input sequences. The aggregated LSTM score is consequently reported as a separate diagnostic result rather than definitive evidence of superiority over the fixed-origin weekly forecasts.

## Installation

Python 3.10 or later is recommended. Create and activate an isolated environment before installing the dependencies.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib seaborn scipy statsmodels scikit-learn tensorflow holidays requests jupyter
```

On Windows, activate the environment with:

```powershell
.venv\Scripts\activate
```

## Running the analysis

Start Jupyter from the repository root:

```bash
jupyter notebook
```

Open `Tejendra.ipynb`, select the environment created above, and run the cells from top to bottom. Internet access is required when the notebook requests historical weather data. The electricity-demand CSV must be available at the location expected by the data-loading cell.

For a reproducibility check, restart the kernel and select **Run All**. Confirm that:

- the input data are found;
- the weather request completes successfully;
- all model cells execute without an exception;
- the tables contain finite metric values; and
- every figure displays readable titles, labels, units, and legends.

TensorFlow training can produce slightly different results between systems unless all random sources and deterministic operations are controlled. Hardware and library versions can also influence training time.

## Evaluation notes

RMSE gives greater weight to large errors, MAE reports the average absolute error in the original load units, and MAPE expresses the average proportional error as a percentage. Model selection should not be performed using the final test-period errors. A production-quality extension should introduce a chronological validation period or rolling-origin cross-validation and reserve the final hold-out period for one evaluation only.

For operational deployment, measured future temperature must be replaced with weather forecasts that would genuinely have been available at each forecast origin. Holiday and calendar variables do not have the same restriction because they are known in advance.

## Main conclusion

German electricity demand contains a strong annual pattern, making the seasonal-naive forecast an important baseline. In the recorded weekly results, Random Forest and Gradient Boosting improve upon this benchmark, whereas the fitted SARIMA and SARIMAX specifications do not. Random Forest offers a practical balance between weekly predictive accuracy and interpretability through feature importance. The LSTM demonstrates useful short-horizon behaviour, but its results require a common forecast protocol before they can be compared directly with the fixed-origin weekly models.
