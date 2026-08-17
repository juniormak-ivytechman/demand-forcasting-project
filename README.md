# Demand Forecasting and Inventory Recommendation

## Problem

Retailers carrying thousands of SKUs across multiple stores need a repeatable way to decide how much stock to hold and when to reorder, without either running out on peak days or tying up cash in excess inventory. This project builds that pipeline end to end for a single representative item (Item 1, aggregated across 10 stores) using five years of daily sales history, and turns the resulting forecast into a concrete reorder point and safety stock recommendation that a planner could act on directly.

## Approach

The work runs across three notebooks. `01_eda.ipynb` explores the full dataset (913,000 daily store-item records spanning 2013 to 2017) to confirm data quality and surface the patterns that matter for forecasting: sales run higher on weekends and peak seasonally through the mid year months. `02_baseline_forecast.ipynb` builds a naive "same day last week" baseline and a first default Prophet model on Item 1's aggregated demand, splitting the last 90 days out as a proper time based holdout rather than a random split, since shuffling time series data destroys the very structure being forecast. `03_refined_model.ipynb` tunes Prophet with explicit weekly and yearly seasonality terms and a lower changepoint prior scale, tests a SARIMA(1,1,1)(1,1,1,7) model as an alternative, and uses the final model's forecast and residual error to calculate a reorder point and safety stock figure assuming a 7 day supplier lead time and roughly a 95 percent service level.

## Finding

Tuned Prophet cut forecast error by 20.4 percent over the naive "same day last week" baseline, from an MAE of 19.03 units (9.07 percent MAPE) down to 15.15 units (7.11 percent MAPE). Worth noting honestly: the default, untuned Prophet model scored a marginally lower MAE (14.42) than the tuned version, so the seasonality tuning bought tighter percentage error and clearer control over the seasonal components more than a clean MAE win. SARIMA was ruled out, coming in well behind both Prophet variants at an MAE of 31.27 (15.84 percent MAPE). Tuned Prophet was carried forward as the forecasting model, producing a 30 day forward forecast that translates into a recommended reorder point of 1,723 units and a safety stock of 80 units at a 7 day lead time.

## Business impact

This gives a planner a concrete, defensible trigger point instead of a gut feel reorder threshold: hold at least 80 units of buffer stock and place a new order once inventory drops to 1,723 units, sized against both average daily demand (234.8 units) and the model's actual forecast error rather than an arbitrary round number. The same pipeline (EDA, baseline, tuned model, reorder point calculation) generalises directly to any other item or store in the catalog, and is built to be re-run on a rolling basis as new sales data comes in, since a forecast is only as good as how recently it was refreshed.

## Repo structure

```
demand-forecasting-project/
├── README.md
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_baseline_forecast.ipynb
│   └── 03_refined_model.ipynb
├── memo/
│   └── inventory_recommendation.md
└── requirements.txt
```

## Note on data

This project uses the actual Kaggle "Store Item Demand Forecasting Challenge" dataset (10 stores, 50 items, daily sales 2013 to 2017, 913,000 training records).
