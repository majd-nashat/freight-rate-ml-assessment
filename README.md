# Freight Rate Prediction

## Project Overview

This project predicts freight rates for loads using historical freight data.

The task is treated as a forecasting problem because the validation period comes after the training period. The model is trained on historical development data and then used to generate predictions for the validation loads and the December 2025 fixed-load chart.

## Data Preparation

The data cleaning process includes:

* Negative weights are converted to absolute values.
* Missing weights are filled using the median weight from the training data.
* Missing `market_index` values are filled using the median value from the same day.
* Implausible target values are identified using rate-per-mile thresholds.
* Target outliers are excluded from model fitting but are kept for evaluation and prediction.

There are 646 flagged target outliers.

## Validation Strategy

A chronological expanding-window validation approach is used instead of a random split.

| Fold | Training Period  | Test Period         |
| ---- | ---------------- | ------------------- |
| 1    | January – April  | May – June          |
| 2    | January – June   | July – August       |
| 3    | January – August | September – October |

The models are evaluated using:

* MAE
* MAPE
* Median APE
* RMSE

Results are reported for both all rows and clean rows.

## Feature Engineering

The final model uses:

* Distance
* Equipment type
* Weight
* Pickup latitude and longitude
* Delivery latitude and longitude
* Day of week

Coordinates are used instead of city names because some validation cities are unseen during training.

## Model

The final model uses LightGBM to predict:

`log(rate per mile)`

The predicted rate is then calculated from the predicted rate per mile and the load distance.

The model uses a Huber loss and averages predictions across five random seeds.

The main model configuration uses:

* 400 trees
* Learning rate: 0.05
* 15 leaves
* Minimum 40 samples per leaf

## Model Comparison

The final model is compared with:

* A distance-band and equipment baseline
* The final model with `market_index`
* The final model with `quote_signal`
* The final model with calendar features
* The alternative workflow from the original notebook
* An alternative workflow using cleaned weight and coordinates

The validation analysis is performed using the same chronological folds for all candidates.

## Final Predictions

After model comparison, the final model is trained on the available development data while excluding the flagged target outliers from model fitting.

Predictions are generated for:

* Validation loads
* The 31 December 2025 chart rows

The validation predictions contain:

* `load_id`
* `predicted_rate`

Sanity checks verify that predictions are present and positive.

## December 2025 Chart

The notebook generates a December 2025 prediction curve for a fixed load from Lexington to Fort Wayne.

The December predictions are also compared with the alternative workflow.

Because there are no December target labels, the true December rate level cannot be directly verified. This is identified as an important limitation of the December forecast.

## Results

The validation results and summary are included in:

* `validation_results.csv`
* `validation_summary.csv`

The main notebook containing the full analysis, model development, validation, predictions, and December chart is:

* `Freight_Rate_last.ipynb`

## Project Files

```text
Freight_Rate_last.ipynb
validation_results.csv
validation_summary.csv
README.md
```
