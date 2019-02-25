
# Bike Sharing Data Science Project 

This notebook documents the analysis and model development for the Bike Sharing Dataset. It contains the following steps:

- About the Bike Sharing Dataset
- Descriptive Analysis
- Missing Value Analysis
- Outlier Analysis
- Correlation Analysis
- Overview metrics
- Model Selection
    -  Ridge Regession
    - Support Vector Regression
    - Essemble Regressor
    - Random Forest Regressor
- Random Forest
    - Random Forest Model
    - Feature importance
- Conclusion
- Future Work

## About the Bike Sharing Dataset

### Overview

Bike sharing systems are a new generation of traditional bike rentals where the whole process from membership, rental and return back has become automatic. Through these systems, the user is able to easily rent a bike from a particular position and return back at another position. Currently, there are about over 500 bike-sharing programs around the world which are composed of over 500 thousands bicycles. Today, there exists great interest in these systems due to their important role in traffic, environmental and health issues. 

Apart from interesting real-world applications of bike sharing systems, the characteristics of data being generated by these systems make them attractive for the research. Opposed to other transport services such as bus or subway, the duration of travel, departure, and arrival position is explicitly recorded in these systems. This feature turns the bike sharing system into a virtual sensor network that can be used for sensing mobility in the city. Hence, it is expected that most of the important events in the city could be detected via monitoring these data.

### Attribute Information

Both hour.csv and day.csv have the following fields, except hr which is not available in day.csv

- instant: record index
- dteday : date
- season : season (1:springer, 2:summer, 3:fall, 4:winter)
- yr : year (0: 2011, 1:2012)
- mnth : month ( 1 to 12)
- hr : hour (0 to 23)
- holiday : weather day is holiday or not 
- weekday : day of the week
- workingday : if day is neither weekend nor holiday is 1, otherwise is 0.
+ weathersit : 
    - 1: Clear, Few clouds, Partly cloudy, Partly cloudy
    - 2: Mist + Cloudy, Mist + Broken clouds, Mist + Few clouds, Mist
    - 3: Light Snow, Light Rain + Thunderstorm + Scattered clouds, Light Rain + Scattered clouds
    - 4: Heavy Rain + Ice Pallets + Thunderstorm + Mist, Snow + Fog
    - temp : Normalized temperature in Celsius. The values are derived via (t-t_min)/(t_max-t_min), t_min=-8, t_max=+39 (only in hourly scale)
    - atemp: Normalized feeling temperature in Celsius. The values are derived via (t-t_min)/(t_max-t_min), t_min=-16, t_max=+50 (only in hourly scale)
- hum: Normalized humidity. The values are divided to 100 (max)
- windspeed: Normalized wind speed. The values are divided to 67 (max)
- casual: count of casual users
- registered: count of registered users
- cnt: count of total rental bikes including both casual and registered

## Setup

```python
from dataloader import Dataloader
import seaborn as sns
import matplotlib.pyplot as plt
from prettytable import PrettyTable
import numpy as np
# Sklearn model delection
from sklearn.model_selection import RandomizedSearchCV
# Sklearn metrics
from sklearn.metrics import mean_squared_error, mean_squared_log_error
# Sklearn models
from sklearn.linear_model import Lasso, ElasticNet, Ridge, SGDRegressor
from sklearn.svm import SVR, NuSVR
from sklearn.ensemble import BaggingRegressor, RandomForestRegressor
from sklearn.neighbors import KNeighborsClassifier
from sklearn.cluster import KMeans

from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import GradientBoostingClassifier

from sklearn.linear_model import LinearRegression

%matplotlib inline
```

## Descriptive Analysis

Provide data set splits for training, validation, and testing:


```python
dataloader = Dataloader('Bike-Sharing-Dataset/hour.csv')
train, val, test = dataloader.getData()
fullData = dataloader.getFullData()

category_features = ['season', 'holiday', 'mnth', 'hr', 'weekday', 'workingday', 'weathersit']
number_features = ['temp', 'atemp', 'hum', 'windspeed']

features= category_features + number_features
target = ['cnt']
```

Get column names of the pandas data frame:


```python
print(list(fullData.columns))
```

    ['instant', 'dteday', 'season', 'yr', 'mnth', 'hr', 'holiday', 'weekday', 'workingday', 'weathersit', 'temp', 'atemp', 'hum', 'windspeed', 'casual', 'registered', 'cnt']
    

Print the first two samples of the dataset to explore the data:


```python
print(fullData.head(2))
```

       instant      dteday  season  yr  mnth  hr  holiday  weekday  workingday  \
    0        1  2011-01-01       1   0     1   0        0        6           0   
    1        2  2011-01-01       1   0     1   1        0        6           0   
    
       weathersit  temp   atemp   hum  windspeed  casual  registered  cnt  
    0           1  0.24  0.2879  0.81        0.0       3          13   16  
    1           1  0.22  0.2727  0.80        0.0       8          32   40  
    

Get data statistics for each column:


```python
print(fullData[number_features].describe())
```

                   temp         atemp           hum     windspeed
    count  17379.000000  17379.000000  17379.000000  17379.000000
    mean       0.496987      0.475775      0.627229      0.190098
    std        0.192556      0.171850      0.192930      0.122340
    min        0.020000      0.000000      0.000000      0.000000
    25%        0.340000      0.333300      0.480000      0.104500
    50%        0.500000      0.484800      0.630000      0.194000
    75%        0.660000      0.621200      0.780000      0.253700
    max        1.000000      1.000000      1.000000      0.850700
    


```python
print(fullData[category_features].astype('category').describe())
```

            season  holiday   mnth     hr  weekday  workingday  weathersit
    count    17379    17379  17379  17379    17379       17379       17379
    unique       4        2     12     24        7           2           4
    top          3        0      7     17        6           1           1
    freq      4496    16879   1488    730     2512       11865       11413
    

## Missing Value Analysis

Check any NULL values in data:


```python
print(fullData.isnull().any())
```

    instant       False
    dteday        False
    season        False
    yr            False
    mnth          False
    hr            False
    holiday       False
    weekday       False
    workingday    False
    weathersit    False
    temp          False
    atemp         False
    hum           False
    windspeed     False
    casual        False
    registered    False
    cnt           False
    dtype: bool
    

## Outlier Analysis

### Box plots


```python
fig, axes = plt.subplots(nrows=3,ncols=2)
fig.set_size_inches(15, 15)
sns.boxplot(data=train,y="cnt",orient="v",ax=axes[0][0])
sns.boxplot(data=train,y="cnt",x="mnth",orient="v",ax=axes[0][1])
sns.boxplot(data=train,y="cnt",x="weathersit",orient="v",ax=axes[1][0])
sns.boxplot(data=train,y="cnt",x="workingday",orient="v",ax=axes[1][1])
sns.boxplot(data=train,y="cnt",x="hr",orient="v",ax=axes[2][0])
sns.boxplot(data=train,y="cnt",x="temp",orient="v",ax=axes[2][1])

axes[0][0].set(ylabel='Count',title="Box Plot On Count")
axes[0][1].set(xlabel='Month', ylabel='Count',title="Box Plot On Count Across Months")
axes[1][0].set(xlabel='Weather Situation', ylabel='Count',title="Box Plot On Count Across Weather Situations")
axes[1][1].set(xlabel='Working Day', ylabel='Count',title="Box Plot On Count Across Working Day")
axes[2][0].set(xlabel='Hour Of The Day', ylabel='Count',title="Box Plot On Count Across Hour Of The Day")
axes[2][1].set(xlabel='Temperature', ylabel='Count',title="Box Plot On Count Across Temperature")

```




    [Text(0, 0.5, 'Count'),
     Text(0.5, 0, 'Temperature'),
     Text(0.5, 1.0, 'Box Plot On Count Across Temperature')]




![output_23_1](https://user-images.githubusercontent.com/6838540/53339788-96426a00-3907-11e9-918e-744dff9d2310.png)


__Interpretation:__ The working day and holiday box plots indicate that more bicycles are rent during normal working days than on weekends or holidays. The hourly box plots show a local maximum at 8 am and one at 5 pm which indicates that most users of the bicycle rental service use the bikes to get to work or school. Another important factor seems to be the temperature: higher temperatures lead to an increasing number of bike rents and lower temperatures not only decrease the average number of rents but also shows more outliers in the data.

### Remove outliers from data


```python
sns.distplot(train[target[-1]]);
```


![output_26_0](https://user-images.githubusercontent.com/6838540/53339790-96db0080-3907-11e9-95a1-b986fd859bcd.png)


The distribution plot of the count values reveals that the count values do not match a normal distribution. We will use the median and interquartile range (IQR) to identify and remove outliers from the data. (An alternative approach would be the transformation of the target values to a normal distribution and using mean and standard deviation.)  


```python
print("Samples in train set with outliers: {}".format(len(train)))
q1 = train.cnt.quantile(0.25)
q3 = train.cnt.quantile(0.75)
iqr = q3 - q1
lower_bound = q1 -(1.5 * iqr) 
upper_bound = q3 +(1.5 * iqr) 
train_preprocessed = train.loc[(train.cnt >= lower_bound) & (train.cnt <= upper_bound)]
print("Samples in train set without outliers: {}".format(len(train_preprocessed)))
sns.distplot(train_preprocessed.cnt);
```

    Samples in train set with outliers: 15641
    Samples in train set without outliers: 15179
    


![output_28_1](https://user-images.githubusercontent.com/6838540/53339791-96db0080-3907-11e9-916b-184fbe301add.png)


## Correlation Analysis


```python
matrix = train[number_features + target].corr()
heat = np.array(matrix)
heat[np.tril_indices_from(heat)] = False
fig,ax= plt.subplots()
fig.set_size_inches(20,10)
sns.heatmap(matrix, mask=heat,vmax=1.0, vmin=0.0, square=True,annot=True, cmap="Reds")
```




    <matplotlib.axes._subplots.AxesSubplot at 0x19fb1e16ef0>




![output_30_1](https://user-images.githubusercontent.com/6838540/53339792-96db0080-3907-11e9-9922-9b1da0bcd734.png)


__Conclusion:__ At the end of the descriptive analysis chapter, we can note the following points:

- Casual and registered contain direct information about the bike sharing count which is to predict (data leakage). Therefore they are not considered in the feature set.
- The variables "temp" and "atemp" are strongly correlated. To reduce the dimensionality of the predictive model, the feature "atemp" is dismissed.
- The variables "hr" and "temp" seem to be promising features for the bike sharing count prediction.



```python
features.remove('atemp')
```

## Overview Metrics

### Mean Squared Error (MSE)

MSE = $\sqrt{\frac{1}{N} \sum_{i=1}^N (x_i - y_i)^2}$

### Root Mean Squared Logarithmic Error (RMSLE)

RMSLE = $\sqrt{ \frac{1}{N} \sum_{i=1}^N (\log(x_i) - \log(y_i))^2 }$

### $R^2$ Score

$R^2=1-\frac{\sum_{i=1}^{n}e_i^2}{\sum_{i=1}^{n}(y_i-\bar{y})^2}$

## Model Selection

The characteristics of the given problem are:

- __Regression:__ The target variable is a quantity.
- __Small dataset:__ Less than 100K samples.
- __Few features should be important:__ The correlation matrix indicates that a few features contain the information to predict the target variable.

This characteristic makes the following methods most promising: Ridge Regression, Support Vector Regression, Ensemble Regressor, Random Forest Regressor.

We will evaluate the performance of these models in the following:


```python
x_train = train_preprocessed[features].values
y_train = train_preprocessed[target].values.ravel()
# Sort validation set for plots
val = val.sort_values(by=target)
x_val = val[features].values
y_val = val[target].values.ravel()
x_test = test[features].values

table = PrettyTable()
table.field_names = ["Model", "Mean Squared Error", "R² score"]

models = [
    SGDRegressor(max_iter=1000, tol=1e-3),
    Lasso(alpha=0.1),
    ElasticNet(random_state=0),
    Ridge(alpha=.5),
    SVR(gamma='auto', kernel='linear'),
    SVR(gamma='auto', kernel='rbf'),
    BaggingRegressor(),
    BaggingRegressor(KNeighborsClassifier(), max_samples=0.5, max_features=0.5),
    NuSVR(gamma='auto'),
    RandomForestRegressor( random_state=0, n_estimators=300)
]

for model in models:
    model.fit(x_train, y_train) 
    y_res = model.predict(x_val)

    mse = mean_squared_error(y_val, y_res)
    score = model.score(x_val, y_val)    

    table.add_row([type(model).__name__, format(mse, '.2f'), format(score, '.2f')])

print(table)
```

    +-----------------------+--------------------+----------+
    |         Model         | Mean Squared Error | R² score |
    +-----------------------+--------------------+----------+
    |      SGDRegressor     |      37929.42      |   0.07   |
    |         Lasso         |      35295.76      |   0.14   |
    |       ElasticNet      |      35555.92      |   0.13   |
    |         Ridge         |      35297.38      |   0.14   |
    |          SVR          |      40015.04      |   0.02   |
    |          SVR          |      30071.93      |   0.26   |
    |    BaggingRegressor   |      13068.24      |   0.68   |
    |    BaggingRegressor   |      34869.47      |   0.15   |
    |         NuSVR         |      28777.35      |   0.30   |
    | RandomForestRegressor |      12686.57      |   0.69   |
    +-----------------------+--------------------+----------+
    

## Random Forest

### Random Forest Model


```python
# Table setup
table = PrettyTable()
table.field_names = ["Model", "Dataset", "MSE", 'RMSLE', "R² score"]
# Model training
model = RandomForestRegressor( random_state=0, n_estimators=100)
model.fit(x_train, y_train) 

def evaluate(x, y, dataset):
    pred = model.predict(x)

    mse = mean_squared_error(y, pred)
    score = model.score(x, y)    
    rmsle = np.sqrt(mean_squared_log_error(y, pred))

    table.add_row([type(model).__name__, dataset, format(mse, '.2f'), format(rmsle, '.2f'), format(score, '.2f')])
    

evaluate(x_train, y_train, 'training')
evaluate(x_val, y_val, 'validation')

print(table)
```

    +-----------------------+------------+----------+-------+----------+
    |         Model         |  Dataset   |   MSE    | RMSLE | R² score |
    +-----------------------+------------+----------+-------+----------+
    | RandomForestRegressor |  training  |  445.53  |  0.18 |   0.98   |
    | RandomForestRegressor | validation | 12713.94 |  0.47 |   0.69   |
    +-----------------------+------------+----------+-------+----------+
    


```python
# Number of trees in random forest
n_estimators = [int(x) for x in np.linspace(start = 200, stop = 2000, num = 10)]
# Number of features to consider at every split
max_features = ['auto', 'sqrt']
# Maximum number of levels in tree
max_depth = [int(x) for x in np.linspace(10, 110, num = 11)]
max_depth.append(None)
# Minimum number of samples required to split a node
min_samples_split = [2, 5, 10]
# Minimum number of samples required at each leaf node
min_samples_leaf = [1, 2, 4]
# Method of selecting samples for training each tree
bootstrap = [True, False]
# Create the random grid
random_grid = {'n_estimators': n_estimators,
               'max_features': max_features,
               'max_depth': max_depth,
               'min_samples_split': min_samples_split,
               'min_samples_leaf': min_samples_leaf,
               'bootstrap': bootstrap}
print(random_grid)

# Use the random grid to search for best hyperparameters
# First create the base model to tune
rf = RandomForestRegressor()
# Random search of parameters, using 3 fold cross validation, 
# search across 100 different combinations, and use all available cores
model = RandomizedSearchCV(estimator = rf, param_distributions = random_grid, n_iter = 100, cv = 3, verbose=2, random_state=0)
# Fit the random search model
model.fit(x_train, y_train)
print(model)

# Table setup
table = PrettyTable()
table.field_names = ["Model", "Dataset", "MSE", 'RMSLE', "R² score"]

evaluate(x_train, y_train, 'training')
evaluate(x_val, y_val, 'validation')

print(table)
```

### Feature importance


```python
importances = model.feature_importances_
std = np.std([tree.feature_importances_ for tree in model.estimators_], axis=0)
indices = np.argsort(importances)[::-1]
```


```python
# Print the feature ranking
print("Feature ranking:")

for f in range(x_val.shape[1]):
    print("%d. feature %s (%f)" % (f + 1, features[indices[f]], importances[indices[f]]))
```

    Feature ranking:
    1. feature hr (0.599712)
    2. feature temp (0.177191)
    3. feature hum (0.062913)
    4. feature workingday (0.045378)
    5. feature windspeed (0.033074)
    6. feature mnth (0.025497)
    7. feature weathersit (0.022410)
    8. feature weekday (0.020978)
    9. feature season (0.009541)
    10. feature holiday (0.003305)
    


```python
# Plot the feature importances of the forest
plt.figure(figsize=(18,5))
plt.title("Feature importances")
plt.bar(range(x_val.shape[1]), importances[indices], color="cornflowerblue", yerr=std[indices], align="center")
plt.xticks(range(x_val.shape[1]), [features[i] for i in indices])
plt.xlim([-1, x_val.shape[1]])
plt.show()
```


![output_50_0](https://user-images.githubusercontent.com/6838540/53339793-96db0080-3907-11e9-8067-f1fafe628881.png)


__Interpretation:__ The result corresponds to the high correlation of the hour and temperature variable with the bicycle sharing count in the feature correlation matrix.

## Future Work

Here are some ideas of future work to improve the performance of the data model further:

- Distribution adjustment of the target variable: Some predictive models assume a normal distribution of the target variable - a transformation in the data preprocessing could improve the performance of such methods. 
- Large scale dataset implementation of random forests. For large scale datasets (> 10 Mio. samples) the used sklearn python implementation of random forests will extremely slow down if it is unable to hold all samples in the working memory or can run into serious memory problems. A solution could be the [woody](https://github.com/gieseke/woody) implementation with top trees for pre-classification and flat random forests implemented in C at the leaves of the top trees.
