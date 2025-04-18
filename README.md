# Dengue Fever Project

## Intro & Problem Statement

Dengue fever is a public health concern in tropical and subtropical regions. Early detection of outbreaks can help health officials to take proactive measures to contain the impact.

The goal of this project, as part of [DengAI competition](https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/page/80/), is to develop a predictive model that accurately forecasts the total number of dengue fever cases in San Juan and Iquitos based on historical, climatic, and environmental data.

## Dataset files:

To predict the total_cases label for each (city, year, weekofyear), we will be working with follwing [datasets](https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/page/81/) which can be downloaded.

- Training features: dengue_features_train.csv
- Training labels : dengue_labels_train.csv
- Test features for prediction: dengue_features_test.csv
- The format for submission: submission_format.csv

The training and test data contains data for two cities (San Juan, Puerto Rico and Iquitos, Peru). In the Training labels, we have total cases labels of two cities for each combination of (city, year, weekofyear) for corresponding climate features given in dengue_features_train.csv. The test data contains the climate features for each city which spans 5 and 3 years respectively.

### Feature files

Each feature datafile contains the following columns:

| city | year | weekofyear | week_start_date | ndvi_ne | ndvi_nw | ndvi_se | ndvi_sw | precipitation_amt_mm | reanalysis_air_temp_k | reanalysis_avg_temp_k | reanalysis_dew_point_temp_k | reanalysis_max_air_temp_k | reanalysis_min_air_temp_k | reanalysis_precip_amt_kg_per_m2 | reanalysis_relative_humidity_percent | reanalysis_sat_precip_amt_mm | reanalysis_specific_humidity_g_per_kg | reanalysis_tdtr_k | station_avg_temp_c | station_diur_temp_rng_c | station_max_temp_c | station_min_temp_c | station_precip_mm |
| ---- | ---- | ---------- | --------------- | ------- | ------- | ------- | ------- | -------------------- | --------------------- | --------------------- | --------------------------- | ------------------------- | ------------------------- | ------------------------------- | ------------------------------------ | ---------------------------- | ------------------------------------- | ----------------- | ------------------ | ----------------------- | ------------------ | ------------------ | ----------------- |

#### Description:

- city – City abbreviations: sj for San Juan and iq for Iquitos
- week_start_date – Date given in yyyy-mm-dd format
- station_max_temp_c – Maximum temperature
- station_min_temp_c – Minimum temperature
- station_avg_temp_c – Average temperature
- station_precip_mm – Total precipitation
- station_diur_temp_rng_c – Diurnal temperature range
- precipitation_amt_mm – Total precipitation
- reanalysis_sat_precip_amt_mm – Total precipitation
- reanalysis_dew_point_temp_k – Mean dew point temperature
- reanalysis_air_temp_k – Mean air temperature
- reanalysis_relative_humidity_percent – Mean relative humidity
- reanalysis_specific_humidity_g_per_kg – Mean specific humidity
- reanalysis_precip_amt_kg_per_m2 – Total precipitation
- reanalysis_max_air_temp_k – Maximum air temperature
- reanalysis_min_air_temp_k – Minimum air temperature
- reanalysis_avg_temp_k – Average air temperature
- reanalysis_tdtr_k – Diurnal temperature range

### Labels file

Training labels file (dengue_labels_train.csv) contains following columns.

| city | year | weekofyear | total_cases |
| ---- | ---- | ---------- | ----------- |

## Approach

### 1. Checking the Data

Getting familiar with the data:

![img.png](./images/img.png)

Checking for seasonality

![img_1.png](./images/img_1.png)

Checking for correlation
![img_2.png](./images/img_2.png)

Checking for missing values

```city 0
year                                       0
weekofyear                                 0
week_start_date                            0
ndvi_ne                                  194
ndvi_nw                                   52
ndvi_se                                   22
ndvi_sw                                   22
precipitation_amt_mm                      13
reanalysis_air_temp_k                     10
reanalysis_avg_temp_k                     10
reanalysis_dew_point_temp_k               10
reanalysis_max_air_temp_k                 10
reanalysis_min_air_temp_k                 10
reanalysis_precip_amt_kg_per_m2           10
reanalysis_relative_humidity_percent      10
reanalysis_sat_precip_amt_mm              13
reanalysis_specific_humidity_g_per_kg     10
reanalysis_tdtr_k                         10
station_avg_temp_c                        43
station_diur_temp_rng_c                   43
station_max_temp_c                        20
station_min_temp_c                        14
station_precip_mm                         22
total_cases                                0
```

### 2. Handling Missing Values with KNN Imputer

We used the KNNImputer to fill in the missing values in the dataset.

```
from sklearn.impute import KNNImputer
imputer = KNNImputer(n_neighbors=5)
df_filled = imputer.fit_transform(df)
```

The idea is to find the 5 closest neighbors to the missing value and take the average of those 5 neighbors to fill in the missing value.

Going forward it would have been interesting to try different values for n_neighbors to see if it would have improved the model.

Also we could try different imputing methods to see if it would have improved the model.

### 3. Detecting & Handling Outliers - Keep or Remove?

We decided to first impute the missing values and then detect and handle outliers because we assumed that outliers in the dataset still contain valuable information that could be useful for the model.

We used the z-score method to detect outliers in the dataset.

Overall we got mixed results. One team member founding that removing outliers improved the model while the other team member found that keeping the outliers improved the model.

More testing needed to establish whether or not to keep outliers

### 4. Scaling the Data with MaxAbsScaler

We used the MaxAbsScaler to scale the data.

### 5. Identifying the Best Model to Use

Although we were not given a validation set, we decided to still use the training dataset and split into a training and validation set to be able to test models and parameters.

We tried with three models to see which one performs the best:

```
models = [RandomForestRegressor(),LinearRegression(),SVR()]
for model in models:
    model.fit(X_train,y_train)
    y_pred = model.predict(X_test)
    print(f'{model} MSE: {mean_squared_error(y_test,y_pred)}')
```

The MSE for each model was as follows:
RandomForestRegressor() MSE: 406.71694216027873
LinearRegression() MSE: 466.33090156794424
SVR() MSE: 515.257572877534

### 6. Hyperparameter Tuning

We used GridSearchCV to find the best hyperparameters for the RandomForestRegressor model.

```
from sklearn.model_selection import GridSearchCV
param_grid = {
    'n_estimators':[100,200,300],
    'max_depth':[3,5,7,10],
    'min_samples_split':[2,5,10],
    'min_samples_leaf':[1,2,4]
}
grid_search = GridSearchCV(RandomForestRegressor(),param_grid,cv=5,scoring='neg_mean_squared_error')
grid_search.fit(X_train,y_train)
grid_search.best_params_
```

Results of the GridSearchCV:

{'max_depth': 7,
'min_samples_leaf': 4,
'min_samples_split': 5,
'n_estimators': 200}

### 7. Experimentation & Model Improvement

Our Backlog of assumptions & experiment ideas:

1. Cyclical enconding for the week of the year - IMPROVED
2. Rolling averages (rainfall, temperature, etc.) as new features - DID NOT IMPROVE
3. Keeping rather than removing outliers - MIXED RESULTS
4. Split by cities & build a seperate model for each city - DID NOT IMPROVE
5. Try with pyCaret - PENDING

Other factors that may have (accidentally) improved model (needs to be tested):

1. Merging the train and test datasets potentially improved imputation >> model performance
2. Rounding the target variable rather than directly converting outome to integer

### 8. Score & Ranking

- Best Score: 25.3413
- Rank: 1,335
