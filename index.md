# Forecasting Power Outage Duration Using Predictive Models

**Author:** Gideon Shaked ([gshaked@umich.edu](mailto:gshaked@umich.edu))

## Introduction

Power outages are costly disruptions that can stem from various causes, ranging from severe weather to technical failures. Understanding the relationships between the *cause* of an outage and its *impact*--both in terms of restoration time and number of customers affected--is vital for both policymakers and utility companies.

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

<iframe src="assets/plots/univar/Outage_Duration.html" width="100%" height="100%" scrolling="no" frameborder="0"></iframe>

Outages typically last under 3 days, though long tails exist due to extreme events.

We also looked at how outages are distributed over time:

<iframe src="assets/plots/univar/Power_Outages_by_Year.html" width="100%" height="100%" frameborder="0"></iframe>

<iframe src="assets/plots/univar/Average_Outage_Duration_by_Month.html" width="100%" height="100%" frameborder="0"></iframe>

Outage frequency peaks in summer, likely reflecting storm seasons, while average duration spikes in winter--potentially due to ice storms and slower repairs.

### Bivariate Analysis

To examine cause-specific impacts:

<iframe src="assets/plots/bivar/Outage_Duration_by_Cause_Category.html" width="100%" height="100%" frameborder="0"></iframe>

Weather-related outages generally have higher durations than equipment failures or intentional attacks.

We also explored how outage causes vary over time and geography:

<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Month_pct.html" width="100%" height="100%" frameborder="0"></iframe>  
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Climate_Region_pct.html" width="100%" height="100%" frameborder="0"></iframe>  
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Year_pct.html" width="100%" height="100%" frameborder="0"></iframe>

Finally, we visualized how time-of-day and work hours influence outage durations:

<iframe src="assets/plots/bivar/Outage_Duration_Distribution-_Work_Hours_vs_Off_Hours_by_Region.html" width="100%" height="100%" frameborder="0"></iframe>

Outages during off-hours tend to last longer, especially in certain climate zones.

### Aggregated Tables

We aggregated data to understand regional variation in outage impact:

- <table on Climate Region & Cause Analysis here>
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

<iframe src="assets/plots/eval/Actual_vs_Predicted_Outage_Duration_with_Absolute_Error.html" width="100%" height="100%" frameborder="0"></iframe>

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

<iframe src="assets/plots/eval/plot_1_2025-04-19_18-19-23.644105.html" width="100%" height="100%" frameborder="0"></iframe>
