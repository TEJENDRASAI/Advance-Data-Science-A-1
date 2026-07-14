# Advance-Data-Science
A 104-Week Stress Test of German Electricity-Load Forecasts

> Candidate: Tejendra Sontineni  
> Case-study window: January 2015 to September 2020  
> Primary artefacts: `Tejendra.ipynb` and `Tejendra.pdf`

## Sixty-second orientation

This case study asks a deliberately demanding question: after two full years are withheld, does a fitted forecasting method retain more useful information than simply repeating the load observed in the corresponding week of the previous year?

German hourly demand is examined at two clocks. Weekly averages support a common 104-week stress test for benchmarks and seasonal state-space models. Hourly observations support a separate recurrent-network experiment. Berlin weather and German public holidays are introduced only after the univariate evidence has been established. The analysis therefore treats seasonal persistence as evidence to defeat, not as a weak preliminary baseline.

The implementation covers exploratory diagnosis, dual stationarity testing, exhaustive SARIMA order search, conditional SARIMAX, two tree ensembles and three LSTM configurations. RMSE, MAE and MAPE are retained together so that the decision is not determined by one loss function.

## Evidence map

| Analytical obligation | Where it is handled | Evidence produced |
|---|---|---|
| Acquire and prepare German load | Opening data block | UTC-indexed hourly series plus daily and weekly averages |
| Establish temporal structure | Diagnostic block | annual profile, decomposition, change distribution, ACF/PACF and ADF/KPSS results |
| Set a minimum performance standard | Benchmark block | mean, naive, seasonal-naive and drift forecasts over 104 weeks |
| Search the required SARIMA space | Statistical-model block | all 147 `(p,d,q)` combinations, training AIC ranking, residual checks and intervals |
| Test external information | Covariate block | temperature-, degree-day- and holiday-conditioned SARIMAX variants |
| Examine nonlinear regression | Ensemble block | Gradient Boosting, Random Forest and importance diagnostics |
| Learn from the hourly sequence | Recurrent block | three memory lengths, loss histories and held-out hourly predictions |
| Support the final judgement | Closing block | metric tables, error-shape graphics and model-selection discussion |

## Forecast contracts

A numerical comparison is meaningful only when the frequency, origin and information set agree. The following contracts describe the experiments in the notebook.

| Contract | Target and update rule | Methods covered |
|---|---|---|
| **W-FIXED** | Weekly load; one origin immediately before the 104-week hold-out; no observed test load supplied | mean, naive, seasonal naive, drift and SARIMA |
| **W-COND** | Same weekly horizon; realised test weather supplied as an explanatory scenario | SARIMAX weather specifications |
| **W-ROLL** | Weekly target; earlier realised test weeks enter later lag features | Gradient Boosting and Random Forest in the stored implementation |
| **H-ROLL** | Hourly target; each prediction receives the preceding observed hours | LSTM memory experiments |

`W-COND`, `W-ROLL` and `H-ROLL` answer useful questions, but they are not interchangeable with `W-FIXED`. In particular, averaging rolling hourly predictions into weeks reduces high-frequency error; it does not create an independent 104-week-ahead forecast.

## Reproducing the computation

### Colab route

1. Upload `Tejendra.ipynb` to Google Colab.
2. Choose a Python runtime with sufficient memory; a GPU is optional for the small recurrent models.
3. Restart the runtime so that no object survives from an earlier session.
4. Execute every cell in document order.
5. Keep the browser session connected while the 147-model SARIMA search is running.
6. Confirm that the final comparison objects are regenerated rather than read from stored output.

The load CSV and Berlin weather are requested over the internet, so network access must remain enabled.

The complete execution is expected to take several hours. Most of that time is spent fitting seasonal state-space models with a 52-week period, not training the neural networks.

## Source ledger

| Item | Endpoint or field | Treatment |
|---|---|---|
| German actual load | [OPSD 60-minute single-index release](https://data.open-power-system-data.org/time_series/2020-10-06/time_series_60min_singleindex.csv) | retain `DE_load_actual_entsoe_transparency`; unit is MW |
| Berlin air temperature | [Open-Meteo historical archive](https://archive-api.open-meteo.com/v1/archive) | daily mean converted to weekly summaries and degree-day terms |
| German calendar effects | `holidays.Germany` | weekly count constructed from known public-holiday dates |

The retained electricity interval begins at `2015-01-01 00:00 UTC` and ends at `2020-09-30 23:00 UTC`. The stored run reports 50,400 hourly observations, 2,100 daily averages and 301 week-labelled averages. The last weekly label falls on 4 October because pandas labels a partial Sunday-ending week by its closing date.

No missing German load values are reported in the retrieved sample. The interpolation safeguard therefore changes no observation in the stored run.

## Metric ledger from the stored run

The entries below reproduce the values embedded in the submitted notebook. They are partitioned by contract to prevent a misleading single leaderboard.

| Method | Contract | RMSE (MW) | MAE (MW) | MAPE |
|---|---|---:|---:|---:|
| Seasonal naive | W-FIXED | 3006.76 | 2318.52 | 4.41% |
| SARIMA: non-seasonal `(2,2,6)`; seasonal `(1,1,1)` at lag 52 | W-FIXED | 9230.67 | 7924.20 | 14.89% |
| SARIMAX: temperature + degree days + holidays | W-COND | 4908.74 | 4012.85 | 7.62% |
| Random Forest | W-ROLL | 2456.34 | 1799.71 | 3.45% |
| Gradient Boosting | W-ROLL | 2484.77 | 1874.09 | 3.58% |
| One-week-memory LSTM | H-ROLL | 2117.02 | 1593.38 | 3.02% |

The weekly average of the `H-ROLL` LSTM predictions has RMSE 328.74 MW, MAE 255.01 MW and MAPE 0.48%. It is recorded as an aggregation diagnostic only. It must not be used to declare victory over the `W-FIXED` methods.
