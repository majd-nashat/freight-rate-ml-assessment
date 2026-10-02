# Freight Rate Prediction - Spotter Labs ML Assessment

Predicts the posted rate of freight loads. Final model: LightGBM on `log(rate per mile)`,
averaged over 5 seeds, using distance, equipment, weight, pickup/delivery coordinates and day of week.

## Setup

```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
```

Place the provided files in `data/` with these names:
`train_test.csv`, `validation.csv`, `validation_predictions_template.csv`, `december_chart_inputs.csv`.

## Run

```bash
python -m src.validate     # time-based validation + ablations -> results/*.csv  (~1 min)
python -m src.predict      # final model -> validation_predictions.csv + December predictions
python score.py --predictions validation_predictions.csv --december-predictions data/december_chart_inputs.csv
```

`score.py` validates both files and creates `scorer_results/candidate_december.png`.

## Structure

```
src/config.py     constants, feature list, model parameters
src/data.py       loading and cleaning (applied identically to train and validation)
src/features.py   feature building, city -> coordinates lookup
src/model.py      LightGBM model (log rate per mile, seed-averaged)
src/metrics.py    MAE, MAPE, MedAPE, RMSE
src/validate.py   expanding-window time-based validation, baseline, ablations
src/predict.py    final fit and submission files
notebooks/01_eda.ipynb   exploratory analysis and data-quality findings
results/          validation_results.csv, validation_summary.csv
```

## Key decisions

- **Split:** time-based, expanding window (3 folds). `validation.csv` lies after all training data, so the task is a forecast; a random split is reported only as a reference and is optimistic.
- **Cleaning:** negative `weight` values (sign errors) -> absolute value; missing `weight` -> training median. 646 training rows (1.3%) with an implausible rate per mile (<1 or >5) are excluded from fitting, never from evaluation or prediction.
- **Excluded features:** `quote_signal` matches the rate per mile in some months, is distorted in others, and is flat in the validation period. `market_index` helps in some folds and hurts in others, shifts in distribution, and is absent from the December inputs.
- **Unseen cities:** 8 cities appear only in `validation.csv` (12% of rows), so lanes are described by coordinates and distance, not by city names.
- **December chart:** city coordinates come from a lookup built from the provided files; the model needs no other inputs, so the chart shows the pure date effect (day of week).
