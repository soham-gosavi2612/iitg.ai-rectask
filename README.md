# Electricity Demand Forecasting for PGCB

## 1.Overview

Our aim was to develop a machine-learning model to forecast electricity demand (MW) PGCB. The goal is to predict the next hour's demand using historical load data, weather information, and economic indicators. The final model achieves a **Mean Absolute Percentage Error (MAPE) of approximately 7.59%** on unseen test data.

## 2. Data Collection & Description

The analysis integrates three diverse datasets:

- **PGCB Load Data (`PGCB_date_power_demand.xlsx`)**: Contains hourly power generation, demand, load shedding, and generation mix (gas, liquid fuel, coal, hydro, solar, wind, imports from India).
- **Weather Data (`weather_data.xlsx`)**: Includes hourly meteorological measurements such as temperature, humidity, precipitation, dew point, wind direction, cloud cover, and sunshine duration.
- **Economic Data (`economic_full_1.csv`)**: A World-Bank sourced dataset with annual economic indicators like GDP, population, inflation, electricity consumption, and industrial output.

## 3. Exploratory Data Analysis (EDA)

Preliminary analysis was conducted to understand patterns and relationships:

- **Time Series Trends**: Observed a slight upward trend in electricity demand over the years with clear seasonal and daily cycles.
- **Daily Demand Pattern**: Demand is lowest around 5 AM and peaks around 8 PM, correlating with typical human activity.
- **Weather Impact**: Demand exhibits a non-linear, U-shaped relationship with temperature, with high demand during both hot and cold extremes.
- **Economic Correlation**: GDP and industrial activity show a moderate positive correlation with long-term demand growth.
- **Missing Values**: Columns with >90% missing data (e.g., `india_adani`, `nepal`) were dropped. `solar` was filled with 0 (assuming zero generation at night), while `wind` was retained despite missingness.

## 4. Data Preprocessing & Feature Engineering

### 4.1 Data Cleaning
- Removed columns with excessive missing values.
- Handled duplicate datetime entries by averaging corresponding values.
- Capped extreme outliers in `demand_mw` using the Interquartile Range (IQR) method.
- Converted economic data from wide to long format and aligned it by year.
- The following final features from economic dataset were extracted and then merged with the other two datasets with time as the axis.
  
    **Core demand drivers**
   -'Electric power consumption (kWh per capita)',
   -'Population, total',
   -'Access to electricity (% of population)',
    
    **Economic scale & growth**
   - 'GDP (current US$)',
   - 'GDP growth (annual %)',
    
    **Industrial activity**
   - 'Industry (including construction), value added (% of GDP)',
    
    **System efficiency / losses**
   - 'Electric power transmission and distribution losses (% of output)',
    
    **Energy structure (kept ONLY 1–2, not all)**
   - 'Electricity production from natural gas sources (% of total)',
   - 'Electricity production from hydroelectric sources (% of total)',
    
    **Macro condition**
   - 'Inflation, consumer prices (annual %)'
]

### 4.2 Feature Engineering
#### Temporal Features
- Extracted `hour`, `dayofweek`, `month`, and created a binary `is_weekend` flag.

#### Lag Features
- `lag_1`, `lag_2`, `lag_3`: Short-term memory.
- `lag_24`: Same hour from the previous day.
- `lag_168`: Same hour from the previous week.

#### Rolling Statistics (for Smoothing)
- `rolling_mean_3`: 3-hour average demand.
- `rolling_mean_24`: Average demand over the last 24 hours (`lag_1` shifted to prevent leakage).
- `rolling_std_24`: Volatility over the last 24 hours.

#### Derived Economic Features
- `gdp_log`: Log-transformed GDP to linearize growth.
- `gdp_pct_change`: Year-over-year GDP growth.
- `pop_log`: Log-transformed population.
- `pop_pct_change`: Year-over-year population growth rate.
- `gdp_growth_lag1`: Previous year's GDP growth (used to avoid look-ahead bias).

#### Target Variable
- Shifted `demand_mw` back by one hour (`target = demand_mw.shift(-1)`) to predict the *next* hour’s demand using current hour’s features.

## 5. Modeling Approach

### 5.1 Model Selection
Two tree-based ensemble models were evaluated:
- **Random Forest Regressor**
- **XGBoost Regressor**

Both models are well-suited for structured/tabular data, handle non-linear relationships, and provide feature importance.

### 5.2 Training/Test Split
Data was split chronologically:
- **Training**: All data before 2023-01-01 (≈70k hours)
- **Testing**: Data from 2023-01-01 onward (≈22k hours)

This ensures no future data leaks into the training process.

### 5.3 Hyperparameter Tuning
Key parameters for the final Random Forest model:
- `n_estimators=200`
- `max_depth=10` (limits overfitting)
- `random_state=42`
- `n_jobs=-1` (parallel processing)

### 5.4 Feature Importance Analysis
The model relied heavily on **current generation**, followed by the **24-hour lag** and **hour of the day**. Weather and economic features contributed moderately.
so we decided to drop the generation_mw column. With the generation_mw column the MAPE with both Random forest and XGBoost was around **4.37%** but after removing these columns to 
make the model more realistic the final lowest MAPE of **7.590%** was obtained with the Random Forest Regressor Model. 

## 6. Results

| Model         | MAPE (%) |
----------------------------
| Random Forest | 7.590    |
| XGBoost       | 10.716   |

- The **Random Forest** model achieved the lowest MAPE of **7.590%**.
- Removing the highly influential `demand_mw` feature (to force a more realistic scenario) increased MAPE slightly to **7.590%** from **4.37%** but validated the model's reliance on other features like `generation_mw` and `lag_24`.
- so final MAPE score = **7.590%** with Random Forest Regressor Model
## 7. Conclusion & Future Work

This project successfully built a high-accuracy electricity demand forecasting model using historical load, weather, and economic data. The model achieves **<8% MAPE**, indicating strong predictive capability.

**Potential improvements for the future:**
- Incorporate real-time **fuel price data** (coal, gas).
- Integrate **holiday calendars** to capture special-day demand shifts.
- Experiment with other high level models (LSTM/GRU) for better long-range temporal dependencies.
---
