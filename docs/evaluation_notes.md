# Evaluation and reproducibility notes

These notes accompany the original submitted notebook. They clarify observed implementation details without rewriting or retroactively correcting its saved experiments.

## 1. Time coverage

The merged file contains 8,736 observations from January 1, 2021 at 00:00 UTC through December 31, 2021 at 00:00 UTC. February 12, 2021 at 14:00 UTC is missing. There are no duplicate timestamps and no missing target values. A one-row shift therefore is not always an exact one-hour lag at the gap.

The final experiment beginning December 25 contains 145 observations. Its requested 168-observation slice is limited by available data, so the printed 168-hour metric does not assess a complete 168-hour horizon. Its horizon plot is cumulative MAE over the available prefix, not error at exactly the labeled lead time. Very short horizons begin at night and are not representative of daytime performance.

## 2. Feature definitions

The engineered feature builder produces six weather inputs, four cyclic time inputs, three lag/rolling inputs, two interactions and one daytime indicator: 16 features. Labels claiming 17 features are retained in the original notebook and dashboard, but do not match the code.

`diff_Nrg_lag1` and `diff_Nrg_lag2` are one and two previous observations, not yesterday's production. The rolling feature uses previous target values, shifted before averaging.

## 3. Baselines and comparisons

The full-period persistence MAE of 1.8502 kWh is independently reproducible from the supplied CSV when the first lag is filled with zero. The earlier variant that excludes the first observation reports approximately 1.8504 kWh.

The dashboard's 48.06% comparison uses that whole-period baseline against mean cross-validation MAE. The baseline's dates and the CV prediction dates differ. For a fair skill score, evaluate both on identical timestamps with identical weights. The earlier five-fold comparison does use matching test periods and reports Random Forest 1.157 kWh versus persistence 1.998 kWh.

An irradiance-only model's own label "Baseline (cglo only)" is separate from the persistence reference. Neural-network improvements must be read from their individual rows; the approximately 48% result belongs to the Random Forest, not all neural networks.

## 4. Validation and information availability

Initial starter cells intentionally scale before splitting. Later solutions move scaling into train-only fitting and pipelines. Monthly holdouts cover seasons but are not strictly forward-only forecasts because training includes observations later than some held-out periods.

TimeSeriesSplit grows its training window across folds; it does not use a fixed 2.5-month training window in every fold. Test folds exhibit different seasonal production ranges.

In engineered one-step CV, previous observed test-period production is available to subsequent predictions. This can be appropriate for rolling one-step operation, but is distinct from issuing an entire week's forecast at once.

The final autoregressive loop replaces production lags recursively but retains recorded future weather. Its markdown statement of no future weather knowledge does not match the implementation. Interpret it under the course's perfect-weather assumption.

The late-December period is also included in earlier model-selection/CV experiments. It is not a completely untouched final holdout for the overall notebook workflow. Future work should reserve a final chronological test period before model selection.

## 5. Reproducibility and code limitations

Some models fix random seeds, while others and the sequence of Keras training runs are not guaranteed to reproduce identical metrics. Fold-specific and pooled R-squared are different statistics. Neural-network validation results from the final 15% split are not interchangeable with cross-validation results.

Some metric helpers silently trim mismatched arrays; robust future evaluation should align timestamps explicitly and reject unexpected mismatches. Warnings are suppressed in several original cells. Legacy plot labels and narrative performance claims should be checked against code and saved numeric outputs.

The feature function uses a global target variable rather than accepting it explicitly. Later cells depend on earlier state. The pandas upper bound in requirements preserves existing fillna(method=...) calls. No full clean-environment execution or model retraining was performed while preparing this repository.

## Future improvements

Use a complete hourly index with a documented missing-data strategy; separate forecast-weather availability from observed weather; fix seeds and record exact package versions; compare matched-fold baselines; reserve a final untouched test period; repeat forecasts over multiple origins and seasons; and report lead-specific errors separately from cumulative errors.
