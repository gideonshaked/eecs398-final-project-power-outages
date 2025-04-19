# Forecasting Power Outage Duration Using Predictive Models

**Author:** Gideon Shaked ([gshaked@umich.edu](mailto:gshaked@umich.edu))

## Introduction

Power outages are costly disruptions that can stem from various causes, ranging from severe weather to technical failures. Understanding the relationships between the *cause* of an outage and its *impact*, both in terms of restoration time and number of customers affected, is vital for both policymakers and utility companies.

This project investigates the question of **how different aspects of an outage (e.g., weather vs. technical failure) impact the restoration time**

The dataset spans major U.S. power outages from 2000 to 2016 and contains detailed outage characteristics including cause, duration, geographic metadata, and economic indicators.

- **Rows**: ~1,500 outage events  
- **Relevant Columns**
  - **`outage.start.date`**: The date that the outage started.
  - **`outage.start.date`**: The time that the outage started.
  - **`u.s._state`**: Full name of the U.S. state where the outage occurred.
  - **`postal.code`**: State-level postal code abbreviation (e.g. MI, CA).
  - **`nerc.region`**: NERC (North American Electric Reliability Corporation) reliability region associated with the outage.
  - **`climate.region`**: Climate region classification based on geography (e.g. Southeast, Northwest).
  - **`anomaly.level`**: A numerical indicator of how abnormal the weather was relative to historical averages.
  - **`climate.category`**: Broader climate category classification used in the dataset.
  - **`hurricane.names`**: Name of the hurricane associated with the outage, if any.
  - **`cause.category`**: High-level classification of outage cause (e.g. Severe Weather, Equipment Failure).
  - **`cause.category.detail`**: More specific cause label (e.g. Lightning, Thunderstorm, Vandalism).
  - **`demand.loss.mw`**: The megawatts of electricity demand lost during the outage.
  - **`population`**: Total population in the affected area.
  - **`popden_urban`**: Urban population density (people per square mile).
  - **`popden_rural`**: Rural population density (people per square mile).
  - **`poppct_urban`**: Percentage of the local population living in urban areas.

## Data Cleaning and Exploratory Data Analysis

### Univariate Analysis

We began by visualizing the distribution of outage durations:

<iframe src="assets/plots/univar/Outage_Duration_1400x700.html" width="1400" height="700" frameborder="0" scrolling="no"></iframe>

Outages typically last under 3 days, though long tails exist due to extreme events.

We also looked at how outages are distributed over time:

<iframe src="assets/plots/univar/Power_Outages_by_Year_1000x400.html" width="1000" height="400" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/univar/Average_Outage_Duration_by_Month_1400x700.html" width="1400" height="700" frameborder="0" scrolling="no"></iframe>

Outage frequency peaks in summer, likely reflecting storm seasons, while average duration spikes in winter--potentially due to ice storms and slower repairs.

### Bivariate Analysis

To examine cause-specific impacts:

<iframe src="assets/plots/bivar/Outage_Duration_by_Cause_Category_1400x700.html" width="1400" height="700" frameborder="0" scrolling="no"></iframe>

Weather-related outages generally have higher durations than equipment failures or intentional attacks.

We also explored how outage causes vary over time and geography:

<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Month_pct_1000x600.html" width="1000" height="600" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Climate_Region_pct_1000x600.html" width="1000" height="600" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Year_pct_1000x600.html" width="1000" height="600" frameborder="0" scrolling="no"></iframe>

Finally, we visualized how time-of-day and work hours influence outage durations:

<iframe src="assets/plots/bivar/Outage_Duration_Distribution-_Work_Hours_vs_Off_Hours_by_Region_1000x600.html" width="1000" height="600" frameborder="0" scrolling="no"></iframe>

Outages during off-hours tend to last longer, especially in certain climate zones.

### Aggregated Tables

We aggregated data to understand regional variation in outage impact.

#### Climate Region & Cause Analysis here

| climate.region     |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |
public appeal |   severe weather |   system operability disruption |
|:-------------------|--------------------:|------------------------:|---------------------:|------------:|--------
--------:|-----------------:|--------------------------------:|
| Central            |                   5 |                       4 |                   34 |           3 |
2 |              133 |                              10 |
| East North Central |                   3 |                       4 |                   20 |           1 |
2 |              104 |                               3 |
| Northeast          |                   5 |                      14 |                  131 |           1 |
4 |              175 |                              14 |
| Northwest          |                   2 |                       1 |                   85 |           3 |
2 |               25 |                               4 |
| South              |                   9 |                       4 |                   28 |           2 |
42 |              106 |                              27 |
| Southeast          |                   4 |                       0 |                    9 |           0 |
5 |              116 |                              16 |
| Southwest          |                   5 |                       1 |                   61 |           1 |
1 |               10 |                               9 |
| West               |                  21 |                      10 |                   31 |          28 |
9 |               67 |                              39 |
| West North Central |                   1 |                       0 |                    4 |           5 |
2 |                4 |                               0 |

- <table on Seasonal Duration here>
- <table on Economic Impact (sales, price, customers) here>
- <table on Impact Severity Score (duration × affected fraction) here>

These summaries helped us identify which combinations of region and cause led to particularly long or severe outages.

## Framing a Prediction Problem

We framed a **regression** problem:  
**Predict the outage duration (in hours)** based on available metadata *known at the time of the outage.*

This problem has practical importance: better duration forecasts allow for smarter resource allocation and improved communication with customers. The target variable is `outage.duration`, and we evaluated models using **Mean Squared Error (MSE)**, to capture the size of deviation in duration estimates.

## Baseline Model

We built a pipeline using a **Gradient Boosting Regressor**, including both categorical encodings and numerical standardization.

- **Quantitative features**: year, hour, anomaly level, demand loss, urban %  
- **Categorical features**: state, postal code, cause, climate zone, etc.  

The baseline model’s best parameters were:

- `n_estimators`: 200  
- `max_depth`: 3  
- `learning_rate`: 0.05  
- `min_samples_leaf`: 2  
- `min_samples_split`: 2

The test MSE: **10,625**

### Performance Visualization

<iframe src="assets/plots/eval/Actual_vs_Predicted_Outage_Duration_with_Absolute_Error_1000x700.html" width="1000" height="700" frameborder="0" scrolling="no"></iframe>

## Final Model

To improve on our baseline, we engineered six new features:

- `year_offset`: relative year from baseline
- `had_hurricane`: binary hurricane flag
- `cause_combined`: composite of cause and sub-cause
- `popden_diff`: difference between urban and rural density
- `demand_loss_log`: log-transformed demand loss
- `is_peak_hour`: binary peak hour flag

The same model architecture and hyperparameter grid were used. Final MSE: **10,410**, an improvement over the baseline.

### Comparison Visualization

<iframe src="assets/plots/eval/plot_1_2025-04-19_19-22-38.867170_1200x700.html" width="1200" height="700" frameborder="0" scrolling="no"></iframe>

## Conclusion

In this project, we set out to understand how the cause of a power outage affects its overall impact--specifically the number of customers affected and the time it takes to restore service. Through a combination of exploratory analysis and predictive modeling, we found that outages caused by severe weather tend to be more disruptive and longer-lasting than those from technical failures or other sources.

We trained a regression model to predict outage duration using only information that would be available at the time the outage begins. Our baseline model already showed reasonable performance, but after engineering a few additional features, like whether a hurricane was involved or how urban the affected area was. With these added features, we were able to reduce the prediction error even further.

While there’s still plenty of room for improvement, this project shows that it’s possible to make meaningful predictions about outage duration using static metadata alone. That kind of insight could be valuable for improving communication with customers and planning emergency responses more effectively.
