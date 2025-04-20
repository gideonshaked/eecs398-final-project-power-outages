# Forecasting Power Outage Duration Using Predictive Models

**Author:** Gideon Shaked ([gshaked@umich.edu](mailto:gshaked@umich.edu))

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

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

<iframe src="assets/plots/univar/Outage_Duration_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>

Outages typically last under 3 days, though long tails exist due to extreme events.

We also looked at how outages are distributed over time:

<iframe src="assets/plots/univar/Power_Outages_by_Year_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/univar/Average_Outage_Duration_by_Month_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>

Outage frequency peaks in summer, likely reflecting storm seasons, while average duration spikes in winter--potentially due to ice storms and slower repairs.

### Bivariate Analysis

To examine cause-specific impacts:

<iframe src="assets/plots/bivar/Outage_Duration_by_Cause_Category_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>

Weather-related outages generally have higher durations than equipment failures or intentional attacks.

We also explored how outage causes vary over time and geography:

<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Month_pct_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Climate_Region_pct_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>
<iframe src="assets/plots/bivar/Distribution_of_Each_Cause_Category_by_Year_pct_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>

Finally, we visualized how time-of-day and work hours influence outage durations:

<iframe src="assets/plots/bivar/Outage_Duration_Distribution-_Work_Hours_vs_Off_Hours_by_Region_700x350.html" width="700" height="350" frameborder="0" scrolling="no"></iframe>

It looks lie outages during off-hours tend to last longer, especially in certain climate zones.

### Aggregated Tables

We aggregated data to understand regional variation in outage impact, which helped us identify which combinations of region and cause led to particularly long or severe outages.

#### Number of Days Affected by Cause and Climate Region

This table shows how many days each climate region was affected by different types of power outage causes. It provides insight into which causes are most frequent across regions. For example, severe weather is consistently a leading cause, especially in the Northeast, Southeast, and Central regions.

| Climate Region       | Equipment Failure | Fuel Supply Emergency | Intentional Attack | Islanding | Public Appeal | Severe Weather | System Operability Disruption |
|----------------------|------------------:|-----------------------:|-------------------:|----------:|--------------:|----------------:|-------------------------------:|
|:-------------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Central            |                   5 |                       4 |                   34 |           3 |               2 |              133 |                              10 |
| East North Central |                   3 |                       4 |                   20 |           1 |               2 |              104 |                               3 |
| Northeast          |                   5 |                      14 |                  131 |           1 |               4 |              175 |                              14 |
| Northwest          |                   2 |                       1 |                   85 |           3 |               2 |               25 |                               4 |
| South              |                   9 |                       4 |                   28 |           2 |              42 |              106 |                              27 |
| Southeast          |                   4 |                       0 |                    9 |           0 |               5 |              116 |                              16 |
| Southwest          |                   5 |                       1 |                   61 |           1 |               1 |               10 |                               9 |
| West               |                  21 |                      10 |                   31 |          28 |               9 |               67 |                              39 |
| West North Central |                   1 |                       0 |                    4 |           5 |               2 |                4 |                               0 |

#### Number of Customers Affected by Cause and Climate Region

This table tracks the scale of impact by showing the number of customers affected in each climate region for each cause of power outage. It complements the first table by highlighting not just the frequency of outages but the magnitude. For instance, although equipment failures occurred less frequently in the West, they affected a large number of customers, showing the importance of evaluating not just how often events happen, but how severe they are in terms of customer impact.

| Climate Region       | Equipment Failure | Fuel Supply Emergency | Intentional Attack | Islanding | Public Appeal | Severe Weather | System Operability Disruption |
|:-------------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Central            |             87750   |                     0   |             110.714  |     9666.67 |            0    |         148707   |                        210450   |
| East North Central |                 0   |                     0   |             660.111  |        0    |         7600    |         134972   |                        759738   |
| Northeast          |             28575.8 |                     0.5 |            1055.58   |        0    |        18600    |         169467   |                        530759   |
| Northwest          |             46651.5 |                     0   |              92.5926 |        0    |         4000    |         169284   |                         35000   |
| South              |             62721.7 |                     0   |            1042.83   |    14500    |         4917.64 |         223231   |                        227102   |
| Southeast          |            145420   |                     0   |               0      |        0    |            0    |         206523   |                         75555.5 |
| Southwest          |             55666.7 |                     0   |             327.423  |    35230    |            0    |          85138.4 |                        135656   |
| West               |            198608   |                     0   |           14060      |     5039.19 |            0    |         361041   |                        152040   |
| West North Central |                 0   |                     0   |               0      |        0    |        34500    |          74178   |                             0   |

## Framing a Prediction Problem

We decided to frame a regression problem:

> Given all of the data known at the time of the outage, predict the duration of the outage.

This problem has practical importance: better duration forecasts allow for smarter resource allocation and improved communication with customers. The target variable is `outage.duration`, and we evaluated models using **Mean Squared Error (MSE)**, to capture the size of deviation in duration estimates.

## Baseline Model

### Feature Selection and Encoding

We selected **21 features** used to predict the target variable: `outage.duration`.

#### Quantitative Features (7)

These features are continuous or count-based. Missing values were imputed using the mean, and values were standardized using `StandardScaler`.

- `year`
- `month`
- `hour`
- `demand.loss.mw`
- `population`
- `popden_urban`
- `popden_rural`

#### Nominal Categorical Features (14)

These features have no intrinsic order. They were encoded using `OneHotEncoder`.

- `season`
- `day_period`
- `is_work_hours`
- `is_weekend`
- `u.s._state`
- `postal.code`
- `nerc.region`
- `climate.region`
- `climate.category`
- `hurricane.names`
- `cause.category`
- `cause.category.detail`
- `poppct_urban`
- `anomaly.level`
  - Note: this feature is technically ordinal, but treating it as such made the MSE increase massively so it was encoded as nominal.

#### Target Variable

- `outage.duration`: The duration of the power outage in hours (regression target -- quantitative).

### Evaluation Metrics

To assess model performance, I used **Root Mean Squared Error (RMSE)** and the **Coefficient of Determination ($R^2$)**.

#### Root Mean Squared Error (RMSE)

RMSE is a widely used metric that measures the average magnitude of the prediction error, with larger errors penalized more heavily due to squaring. It is defined as:

$$
\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^n (\hat{y}_i - y_i)^2}
$$

- **Advantages**: RMSE is intuitive and retains the original units of the target variable, making it easy to interpret. It is particularly useful when large errors are undesirable, as it emphasizes them more than metrics like Mean Absolute Error (MAE).

#### Coefficient of Determination ($R^2$)

$R^2$ measures how well the model explains the variability of the target variable:

$$
R^2 = 1 - \frac{\sum_{i=1}^n (\hat{y}_i - y_i)^2}{\sum_{i=1}^n (y_i - \bar{y})^2}
$$

- **Advantages**: $R^2$ provides a normalized measure of fit, ranging from 0 (no explanatory power) to 1 (perfect prediction). It complements RMSE by showing how well the model captures overall variance rather than just minimizing error.

Together, RMSE and $R^2$ offer a comprehensive view of model performance, with one emphasizing prediction error magnitude, and the other explaining variance captured.

### Baseline Model Results

#### Parameter Selection

After performing a grid search with 5-fold cross-validation, the baseline model’s best parameters were:

- `n_estimators`: 200  
- `max_depth`: 3  
- `learning_rate`: 0.05  
- `min_samples_leaf`: 2  
- `min_samples_split`: 2

#### Performance Metrics

- **RMSE:** 103.01
- **$R^2$:** 0.37

#### Performance Visualization

The following plot shows the model performance for outage duration over time, along with the absolute error.

<iframe src="assets/plots/eval/eval_outage_duration_900x800.html" width="900" height="800" frameborder="0" scrolling="no"></iframe>

#### Baseline Model Evaluation

We can see from the plot above that the model tends to fail to predict outlier outage events, which suggests that there is an extraneous variable influencing the outage duration that is not given in our dataset. This is also supported by the low $R^2$, which suggests that the variance of the actual outage duration is not captured by the prediction model.

That being said, for the majority of outages the model's performance is decent. Overall, I think that this model's performance is good.

## Final Model

### New Features

To improve on our baseline, we engineered six new features:

- `year_offset`: relative year from baseline
- `had_hurricane`: binary hurricane flag
- `cause_combined`: composite of cause and sub-cause
- `popden_diff`: difference between urban and rural density
- `demand_loss_log`: log-transformed demand loss
- `is_peak_hour`: binary peak hour flag

The same model, model architecture, hyperparameter grid, and preprocessing steps were used.

### Final Model Results

#### Parameter Selection

After performing a grid search with 5-fold cross-validation, the final model’s best parameters were:

- `n_estimators`: 200  
- `max_depth`: 3  
- `learning_rate`: 0.05  
- `min_samples_leaf`: 2  
- `min_samples_split`: 5 (**changed from baseline model**)

#### Performance Metrics

- **RMSE:** 102
- **$R^2$:** 0.38

#### Performance Visualization

The following plot shows the final model's performance for outage duration over time compared to the baseline model's performance, along with the absolute error as before.

<iframe src="assets/plots/eval/final_model_eval_900x800.html" width="900" height="800" frameborder="0" scrolling="no"></iframe>

#### Final Model Evaluation

The final model is only a slight improvement on the baseline model, with the $R^2$ improving from 0.37 to 0.38 and the RMSE improving from 103 to 102.

That being said, I believe that the added features add additional robustness to the model, and that they will make it more generalizable to unseen data.

## Conclusion

In this project, we set out to understand how the cause of a power outage affects its overall impact--specifically the number of customers affected and the time it takes to restore service. Through a combination of exploratory analysis and predictive modeling, we found that outages caused by severe weather tend to be more disruptive and longer-lasting than those from technical failures or other sources.

We trained a regression model to predict outage duration using only information that would be available at the time the outage begins. Our baseline model already showed reasonable performance, but after engineering a few additional features, like whether a hurricane was involved or how urban the affected area was. With these added features, we were able to reduce the prediction error even further.

While there’s still plenty of room for improvement, this project shows that it’s possible to make meaningful predictions about outage duration using static metadata alone. That kind of insight could be valuable for improving communication with customers and planning emergency responses more effectively.
