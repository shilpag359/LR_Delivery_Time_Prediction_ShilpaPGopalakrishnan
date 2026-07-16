# Delivery Time Prediction (Linear Regression)

Predicting order delivery time for a food/grocery delivery platform (Porter) using order, restaurant, and logistics features.

## Problem Statement

When a customer places an order, how long will it actually take to arrive? This project builds a regression model that predicts delivery time using features like order size, restaurant location, order protocol, and delivery partner availability — the kind of prediction that powers the "arriving in X minutes" estimate shown to customers.

## Dataset

The dataset contains historical Porter delivery records with fields including:
- Order timestamps (`created_at`, `actual_delivery_time`)
- Order details (`total_items`, `num_distinct_items`, `min_item_price`, `max_item_price`, `market_id`)
- Delivery partner availability (`total_onshift_dashers`, `total_busy_dashers`, `total_outstanding_orders`)
- Time context (`order_hour`, day of week)

Target variable: **`time_taken_hours`** — the actual time between order creation and delivery, engineered from the timestamp fields.

## Approach

1. **Data Cleaning & Feature Engineering**
   - Converted timestamp fields to proper datetime format
   - Engineered the target variable (`time_taken_hours`) from `created_at` and `actual_delivery_time`
   - Extracted order hour and day-of-week as features
   - Converted categorical fields (store category, order protocol, day) to proper categorical types

2. **Exploratory Data Analysis**
   - Analyzed distributions of numerical and categorical features
   - Examined correlations against the target variable via heatmap
   - Detected and handled outliers using the IQR method
   - Dropped weakly correlated features to reduce noise

3. **Modeling**
   - Built a baseline Linear Regression model using scikit-learn
   - Compared scaled vs. unscaled features (via `StandardScaler`) to interpret coefficient magnitudes
   - Applied **Recursive Feature Elimination (RFE)**, sweeping from 9 features down to 1 to find the best performance/complexity trade-off:

     | # Features | R² |
     |---|---|
     | 9 | 0.581 |
     | 8 | 0.575 |
     | 7 | 0.572 |
     | 6 | 0.570 |
     | 5 | 0.541 |
     | 4 | 0.496 |
     | 3 | 0.409 |
     | 2 | 0.383 |
     | 1 | 0.076 |

     Best result: **9 features, R² = 0.581** — `market_id`, `total_items`, `num_distinct_items`, `min_item_price`, `max_item_price`, `total_onshift_dashers`, `total_busy_dashers`, `total_outstanding_orders`, `order_hour`
   - **Caught and corrected a data leakage bug**: an intermediate column (`time_taken`, the raw-minutes version of the target) had been left in the feature set in two separate places in the notebook, initially producing a suspicious R² of 1.0. Removing it from both produced honest, generalizable results.

4. **Evaluation**
   - Residual analysis (residuals vs. predicted, Q-Q plot, histogram) to check regression assumptions
   - Coefficient analysis to interpret feature impact on delivery time

## Results

| Metric | Value |
|---|---|
| MAE | 0.52 hours |
| MSE | 0.42 |
| **RMSE** | **0.65 hours** |
| **R²** | **0.58** |

**Top features driving delivery time:**
1. **Total outstanding orders** — a busier queue at the restaurant/partner level increases wait time
2. **Total items / num distinct items** — larger, more complex orders take longer to prepare
3. **Total onshift / busy dashers** — delivery partner availability directly affects how quickly an order gets picked up

## Tech Stack

`Python` · `pandas` · `NumPy` · `scikit-learn` · `matplotlib` · `seaborn` · `statsmodels`

## Key Takeaway

The most important part of this project wasn't the final R² — it was catching a data leakage issue where a duplicate form of the target variable had slipped into the features, producing a perfect (and meaningless) score. Diagnosing that and rebuilding the feature set is what turned this into a legitimate, trustworthy model with an R² of 0.58 on unseen data.

## How to Run

1. Clone this repo
2. Ensure `porter_data_1.xlsx` is in the project directory
3. Open `LR_Delivery_Time_Prediction.ipynb` in Jupyter
4. Run all cells top to bottom (Kernel → Restart & Run All)
