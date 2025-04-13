![cover-header](cover-header.jpg)

Energy efficiency is a paramount concern in modern construction due to the growing demand for sustainable development. Accurately predicting energy consumption enables smarter design decisions, better resource planning, and proactive energy management. In this article, I would like to show the practical usefulness of an Machine Learning (specifically regression for non-binary target variable) in predicting the heating load of a building, depending on its parameters (compactness, wall area, height, glazing, etc.)

A few words about where the data comes from: it has been shared on the UC Irvine Machine Learning Repository ( https://archive.ics.uci.edu/dataset/242/energy+efficiency) under the CC BY 4.0 license. I would like to thank the authors of the original paper and give them credit for their work.

# Extract, Transform and Load (ETL)

Note: The data available for this article are synthetic (generated by external simulator, they are not real measurement data). This is not an ideal situation and I am aware that it causes some limitations, however is still useful to demonstrate the usability and correctness of the selected ML model.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

```python
# Import the csv, change separator
ef_data = pd.read_csv("ENB2012.csv", sep=";")
ef_data.head(10)
```

<div class="colab-df-container" id="df-6d8028bc-fe71-409e-89c3-0ac3683cae64">
  <div>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>RelativeCompactness</th>
          <th>SurfaceArea</th>
          <th>WallArea</th>
          <th>RoofArea</th>
          <th>OverallHeight</th>
          <th>Orientation</th>
          <th>GlazingArea</th>
          <th>GlazingAreaDistribution</th>
          <th>HeatingLoad</th>
          <th>CoolingLoad</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.98</td>
          <td>514.5</td>
          <td>294.0</td>
          <td>110.25</td>
          <td>7.0</td>
          <td>2</td>
          <td>0.0</td>
          <td>0</td>
          <td>15.55</td>
          <td>21.33</td>
        </tr>
     <tr>
      <th>1</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>3</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>4</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>5</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>2</td>
      <td>0.0</td>
      <td>0</td>
      <td>20.84</td>
      <td>28.28</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>3</td>
      <td>0.0</td>
      <td>0</td>
      <td>21.46</td>
      <td>25.38</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>4</td>
      <td>0.0</td>
      <td>0</td>
      <td>20.71</td>
      <td>25.16</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>5</td>
      <td>0.0</td>
      <td>0</td>
      <td>19.68</td>
      <td>29.60</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.86</td>
      <td>588.0</td>
      <td>294.0</td>
      <td>147.00</td>
      <td>7.0</td>
      <td>2</td>
      <td>0.0</td>
      <td>0</td>
      <td>19.50</td>
      <td>27.30</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.86</td>
      <td>588.0</td>
      <td>294.0</td>
      <td>147.00</td>
      <td>7.0</td>
      <td>3</td>
      <td>0.0</td>
      <td>0</td>
      <td>19.95</td>
      <td>21.97</td>
    </tr>
      </tbody>
    </table>
  </div>
</div>

I will generally skip the ETL Process, because I know it has already been done on this particular dataset. I'll do the basic checks only:

```python
# check for data types and basic features
ef_data.info()
ef_data.describe()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 768 entries, 0 to 767
    Data columns (total 10 columns):
     #   Column                   Non-Null Count  Dtype
    ---  ------                   --------------  -----
     0   RelativeCompactness      768 non-null    float64
     1   SurfaceArea              768 non-null    float64
     2   WallArea                 768 non-null    float64
     3   RoofArea                 768 non-null    float64
     4   OverallHeight            768 non-null    float64
     5   Orientation              768 non-null    int64
     6   GlazingArea              768 non-null    float64
     7   GlazingAreaDistribution  768 non-null    int64
     8   HeatingLoad              768 non-null    float64
     9   CoolingLoad              768 non-null    float64
    dtypes: float64(8), int64(2)
    memory usage: 60.1 KB

<div class="colab-df-container" id="df-a7f58b74-2b0a-44d5-a751-485bfd65d566">
    <div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>RelativeCompactness</th>
      <th>SurfaceArea</th>
      <th>WallArea</th>
      <th>RoofArea</th>
      <th>OverallHeight</th>
      <th>Orientation</th>
      <th>GlazingArea</th>
      <th>GlazingAreaDistribution</th>
      <th>HeatingLoad</th>
      <th>CoolingLoad</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.00000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.00000</td>
      <td>768.000000</td>
      <td>768.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.764167</td>
      <td>671.708333</td>
      <td>318.500000</td>
      <td>176.604167</td>
      <td>5.25000</td>
      <td>3.500000</td>
      <td>0.234375</td>
      <td>2.81250</td>
      <td>22.307201</td>
      <td>24.587760</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.105777</td>
      <td>88.086116</td>
      <td>43.626481</td>
      <td>45.165950</td>
      <td>1.75114</td>
      <td>1.118763</td>
      <td>0.133221</td>
      <td>1.55096</td>
      <td>10.090196</td>
      <td>9.513306</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.620000</td>
      <td>514.500000</td>
      <td>245.000000</td>
      <td>110.250000</td>
      <td>3.50000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>6.010000</td>
      <td>10.900000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.682500</td>
      <td>606.375000</td>
      <td>294.000000</td>
      <td>140.875000</td>
      <td>3.50000</td>
      <td>2.750000</td>
      <td>0.100000</td>
      <td>1.75000</td>
      <td>12.992500</td>
      <td>15.620000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.750000</td>
      <td>673.750000</td>
      <td>318.500000</td>
      <td>183.750000</td>
      <td>5.25000</td>
      <td>3.500000</td>
      <td>0.250000</td>
      <td>3.00000</td>
      <td>18.950000</td>
      <td>22.080000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.830000</td>
      <td>741.125000</td>
      <td>343.000000</td>
      <td>220.500000</td>
      <td>7.00000</td>
      <td>4.250000</td>
      <td>0.400000</td>
      <td>4.00000</td>
      <td>31.667500</td>
      <td>33.132500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>0.980000</td>
      <td>808.500000</td>
      <td>416.500000</td>
      <td>220.500000</td>
      <td>7.00000</td>
      <td>5.000000</td>
      <td>0.400000</td>
      <td>5.00000</td>
      <td>43.100000</td>
      <td>48.030000</td>
    </tr>
  </tbody>
</table>
</div>

```python
# check for missing values
ef_data.isna().sum()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>RelativeCompactness</th>
      <td>0</td>
    </tr>
    <tr>
      <th>SurfaceArea</th>
      <td>0</td>
    </tr>
    <tr>
      <th>WallArea</th>
      <td>0</td>
    </tr>
    <tr>
      <th>RoofArea</th>
      <td>0</td>
    </tr>
    <tr>
      <th>OverallHeight</th>
      <td>0</td>
    </tr>
    <tr>
      <th>Orientation</th>
      <td>0</td>
    </tr>
    <tr>
      <th>GlazingArea</th>
      <td>0</td>
    </tr>
    <tr>
      <th>GlazingAreaDistribution</th>
      <td>0</td>
    </tr>
    <tr>
      <th>HeatingLoad</th>
      <td>0</td>
    </tr>
    <tr>
      <th>CoolingLoad</th>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div><br><label><b>dtype:</b> int64</label>

```python
# check for duplicates
ef_data.duplicated().sum()
```

    np.int64(0)

There are no categorical values in this dataset, so class balancing will be skipped. Next step is to check for any potential outliers

```python
# check for outliers.
Q1 = ef_data.quantile(0.25)
Q3 = ef_data.quantile(0.75)
IQR = Q3 - Q1

outliers = ((ef_data < (Q1 - 1.5 * IQR)) | (ef_data > (Q3 + 1.5 * IQR))).sum()
print("Number of outliers in each column:\n", outliers)

```

    Number of outliers in each column:
     RelativeCompactness        0
    SurfaceArea                0
    WallArea                   0
    RoofArea                   0
    OverallHeight              0
    Orientation                0
    GlazingArea                0
    GlazingAreaDistribution    0
    HeatingLoad                0
    CoolingLoad                0
    dtype: int64

According to the above, there is no outliers in this dataset. A default boxplot will confirm this. But a set of boxplots for each column will also show some more interesting information about variance, ranges etc.

```python
plt.figure(figsize=(12,6))
ef_data_cols = list(ef_data.columns)
for num, col in enumerate(ef_data_cols, 1):
  sns.boxplot(x=ef_data[col])
  plt.title(col)
  plt.show()
```

![png](output_14_0.PNG)

![png](output_14_1.PNG)

![png](output_14_2.PNG)

![png](output_14_3.PNG)

![png](output_14_4.PNG)

![png](output_14_5.PNG)

![png](output_14_6.PNG)

![png](output_14_7.PNG)

![png](output_14_8.PNG)

![png](output_14_9.PNG)

One more important transformation. The following formula will generate values ​​in the TotalHeatingLoad column, based on the values ​​from the HeatingLoad, OverallHeight and RoofArea columns. The new values ​​will serve as the target variable in the Machine Learning model.

```python
ef_data["TotalHeatingLoad"] = ef_data["HeatingLoad"] * 3.5/ef_data["OverallHeight"] * ef_data["RoofArea"]
```

A quick look at the correlation matrix will give a better understanding of the relationships between variables, it will help to select the most important ones and prepare the data for training the Machine Learning model.

```python
corr_matrix = ef_data.corr(numeric_only=True)
sns.heatmap(corr_matrix, annot=True, cmap="coolwarm", fmt=".2f")
plt.show()
```

![png](output_18_0.PNG)

Analysis of correlations between variables (building parameters), in particular their connections with the target variable 'TotalHeatingLoad', plus some comments and conclusions.

1. RelativeCompactness and SurfaceArea are very closely correlated and practically the same. Therefore, SurfaceArea (the sum of the surface area of ​​walls, roof and floor) will be selected, because it is more natural. RelativeCompactness/SurfaceArea have quite strong correlation with the target variable.

2. RoofArea and OverallHeight are quite strongly correlated with TotalHeatingLoad as well.

3. GlazingArea also has quite a large impact on TotalHeatingLoad, this parameter certainly cannot be neglected in the Machine Learning process.

Correlations between the remaining parameters:

1. Very strong negative correlation: OverallHeight vs. RoofArea.
2. Also quite strong correlation SurfaceArea/RelativeCompactness vs. RoofArea. In both cases, shis is related to the specific parameters of the simulator that generated the data, but it makes sense in real-world data as well.

The remaining correlations are quite weak or very weak, so they can be omitted when selecting parameters for training the ML model.

Selection of the target variable:
The target variable will be the value of TotalHeatingLoad, (as a derivative of HeatingLoad), which best reflects the heating demand of the entire building in practice.

The data set also contains a second target variable CoolingLoad. But it is very closely related to heating load. Therefore, for simplicity, it will be omitted.

# Machine Learing Process starts here :-)

```python
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeRegressor
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
```

Target variable

```python
y = ef_data["TotalHeatingLoad"]
```

Model parameters

```python
x = ef_data[["SurfaceArea", "RoofArea", "OverallHeight", "GlazingArea"]]
```

Prepare training and test sets

```python
x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.2, shuffle=True, random_state=42)
```

DecisionTreeRegressor will be selected, as the most versatile and robust for this taks. Hyperparamet optimization will be performed beforehand.

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "max_depth": [3, 5, 7, 10],
    "min_samples_split": [2, 5, 10],
    "min_samples_leaf": [1, 2, 4]
}

grid_search = GridSearchCV(DecisionTreeRegressor(random_state=42), param_grid, cv=5, scoring="neg_mean_squared_error")
grid_search.fit(x_train, y_train)

print("Best parameters:", grid_search.best_params_)
best_model = grid_search.best_estimator_
```

    Best parameters: {'max_depth': 10, 'min_samples_leaf': 1, 'min_samples_split': 2}

```python
# create and train DecisionTreeRegressor model with quasi-optimal parametrers
dtr = DecisionTreeRegressor(max_depth=10,  min_samples_leaf=1, min_samples_split=2, random_state=42)
dtr.fit(x_train, y_train)
```

<style>#sk-container-id-1 {
  /* Definition of color scheme common for light and dark mode */
  --sklearn-color-text: #000;
  --sklearn-color-text-muted: #666;
  --sklearn-color-line: gray;
  /* Definition of color scheme for unfitted estimators */
  --sklearn-color-unfitted-level-0: #fff5e6;
  --sklearn-color-unfitted-level-1: #f6e4d2;
  --sklearn-color-unfitted-level-2: #ffe0b3;
  --sklearn-color-unfitted-level-3: chocolate;
  /* Definition of color scheme for fitted estimators */
  --sklearn-color-fitted-level-0: #f0f8ff;
  --sklearn-color-fitted-level-1: #d4ebff;
  --sklearn-color-fitted-level-2: #b3dbfd;
  --sklearn-color-fitted-level-3: cornflowerblue;

  /* Specific color for light theme */
  --sklearn-color-text-on-default-background: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, black)));
  --sklearn-color-background: var(--sg-background-color, var(--theme-background, var(--jp-layout-color0, white)));
  --sklearn-color-border-box: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, black)));
  --sklearn-color-icon: #696969;

  @media (prefers-color-scheme: dark) {
    /* Redefinition of color scheme for dark theme */
    --sklearn-color-text-on-default-background: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, white)));
    --sklearn-color-background: var(--sg-background-color, var(--theme-background, var(--jp-layout-color0, #111)));
    --sklearn-color-border-box: var(--sg-text-color, var(--theme-code-foreground, var(--jp-content-font-color1, white)));
    --sklearn-color-icon: #878787;
  }
}

#sk-container-id-1 {
  color: var(--sklearn-color-text);
}

#sk-container-id-1 pre {
  padding: 0;
}

#sk-container-id-1 input.sk-hidden--visually {
  border: 0;
  clip: rect(1px 1px 1px 1px);
  clip: rect(1px, 1px, 1px, 1px);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
}

#sk-container-id-1 div.sk-dashed-wrapped {
  border: 1px dashed var(--sklearn-color-line);
  margin: 0 0.4em 0.5em 0.4em;
  box-sizing: border-box;
  padding-bottom: 0.4em;
  background-color: var(--sklearn-color-background);
}

#sk-container-id-1 div.sk-container {
  /* jupyter's `normalize.less` sets `[hidden] { display: none; }`
     but bootstrap.min.css set `[hidden] { display: none !important; }`
     so we also need the `!important` here to be able to override the
     default hidden behavior on the sphinx rendered scikit-learn.org.
     See: https://github.com/scikit-learn/scikit-learn/issues/21755 */
  display: inline-block !important;
  position: relative;
}

#sk-container-id-1 div.sk-text-repr-fallback {
  display: none;
}

div.sk-parallel-item,
div.sk-serial,
div.sk-item {
  /* draw centered vertical line to link estimators */
  background-image: linear-gradient(var(--sklearn-color-text-on-default-background), var(--sklearn-color-text-on-default-background));
  background-size: 2px 100%;
  background-repeat: no-repeat;
  background-position: center center;
}

/* Parallel-specific style estimator block */

#sk-container-id-1 div.sk-parallel-item::after {
  content: "";
  width: 100%;
  border-bottom: 2px solid var(--sklearn-color-text-on-default-background);
  flex-grow: 1;
}

#sk-container-id-1 div.sk-parallel {
  display: flex;
  align-items: stretch;
  justify-content: center;
  background-color: var(--sklearn-color-background);
  position: relative;
}

#sk-container-id-1 div.sk-parallel-item {
  display: flex;
  flex-direction: column;
}

#sk-container-id-1 div.sk-parallel-item:first-child::after {
  align-self: flex-end;
  width: 50%;
}

#sk-container-id-1 div.sk-parallel-item:last-child::after {
  align-self: flex-start;
  width: 50%;
}

#sk-container-id-1 div.sk-parallel-item:only-child::after {
  width: 0;
}

/* Serial-specific style estimator block */

#sk-container-id-1 div.sk-serial {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: var(--sklearn-color-background);
  padding-right: 1em;
  padding-left: 1em;
}


/* Toggleable style: style used for estimator/Pipeline/ColumnTransformer box that is
clickable and can be expanded/collapsed.
- Pipeline and ColumnTransformer use this feature and define the default style
- Estimators will overwrite some part of the style using the `sk-estimator` class
*/

/* Pipeline and ColumnTransformer style (default) */

#sk-container-id-1 div.sk-toggleable {
  /* Default theme specific background. It is overwritten whether we have a
  specific estimator or a Pipeline/ColumnTransformer */
  background-color: var(--sklearn-color-background);
}

/* Toggleable label */
#sk-container-id-1 label.sk-toggleable__label {
  cursor: pointer;
  display: flex;
  width: 100%;
  margin-bottom: 0;
  padding: 0.5em;
  box-sizing: border-box;
  text-align: center;
  align-items: start;
  justify-content: space-between;
  gap: 0.5em;
}

#sk-container-id-1 label.sk-toggleable__label .caption {
  font-size: 0.6rem;
  font-weight: lighter;
  color: var(--sklearn-color-text-muted);
}

#sk-container-id-1 label.sk-toggleable__label-arrow:before {
  /* Arrow on the left of the label */
  content: "▸";
  float: left;
  margin-right: 0.25em;
  color: var(--sklearn-color-icon);
}

#sk-container-id-1 label.sk-toggleable__label-arrow:hover:before {
  color: var(--sklearn-color-text);
}

/* Toggleable content - dropdown */

#sk-container-id-1 div.sk-toggleable__content {
  max-height: 0;
  max-width: 0;
  overflow: hidden;
  text-align: left;
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content.fitted {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content pre {
  margin: 0.2em;
  border-radius: 0.25em;
  color: var(--sklearn-color-text);
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-toggleable__content.fitted pre {
  /* unfitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

#sk-container-id-1 input.sk-toggleable__control:checked~div.sk-toggleable__content {
  /* Expand drop-down */
  max-height: 200px;
  max-width: 100%;
  overflow: auto;
}

#sk-container-id-1 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {
  content: "▾";
}

/* Pipeline/ColumnTransformer-specific style */

#sk-container-id-1 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-label.fitted input.sk-toggleable__control:checked~label.sk-toggleable__label {
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Estimator-specific style */

/* Colorize estimator box */
#sk-container-id-1 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-estimator.fitted input.sk-toggleable__control:checked~label.sk-toggleable__label {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-2);
}

#sk-container-id-1 div.sk-label label.sk-toggleable__label,
#sk-container-id-1 div.sk-label label {
  /* The background is the default theme color */
  color: var(--sklearn-color-text-on-default-background);
}

/* On hover, darken the color of the background */
#sk-container-id-1 div.sk-label:hover label.sk-toggleable__label {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-unfitted-level-2);
}

/* Label box, darken color on hover, fitted */
#sk-container-id-1 div.sk-label.fitted:hover label.sk-toggleable__label.fitted {
  color: var(--sklearn-color-text);
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Estimator label */

#sk-container-id-1 div.sk-label label {
  font-family: monospace;
  font-weight: bold;
  display: inline-block;
  line-height: 1.2em;
}

#sk-container-id-1 div.sk-label-container {
  text-align: center;
}

/* Estimator-specific */
#sk-container-id-1 div.sk-estimator {
  font-family: monospace;
  border: 1px dotted var(--sklearn-color-border-box);
  border-radius: 0.25em;
  box-sizing: border-box;
  margin-bottom: 0.5em;
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-0);
}

#sk-container-id-1 div.sk-estimator.fitted {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-0);
}

/* on hover */
#sk-container-id-1 div.sk-estimator:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-2);
}

#sk-container-id-1 div.sk-estimator.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-2);
}

/* Specification for estimator info (e.g. "i" and "?") */

/* Common style for "i" and "?" */

.sk-estimator-doc-link,
a:link.sk-estimator-doc-link,
a:visited.sk-estimator-doc-link {
  float: right;
  font-size: smaller;
  line-height: 1em;
  font-family: monospace;
  background-color: var(--sklearn-color-background);
  border-radius: 1em;
  height: 1em;
  width: 1em;
  text-decoration: none !important;
  margin-left: 0.5em;
  text-align: center;
  /* unfitted */
  border: var(--sklearn-color-unfitted-level-1) 1pt solid;
  color: var(--sklearn-color-unfitted-level-1);
}

.sk-estimator-doc-link.fitted,
a:link.sk-estimator-doc-link.fitted,
a:visited.sk-estimator-doc-link.fitted {
  /* fitted */
  border: var(--sklearn-color-fitted-level-1) 1pt solid;
  color: var(--sklearn-color-fitted-level-1);
}

/* On hover */
div.sk-estimator:hover .sk-estimator-doc-link:hover,
.sk-estimator-doc-link:hover,
div.sk-label-container:hover .sk-estimator-doc-link:hover,
.sk-estimator-doc-link:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

div.sk-estimator.fitted:hover .sk-estimator-doc-link.fitted:hover,
.sk-estimator-doc-link.fitted:hover,
div.sk-label-container:hover .sk-estimator-doc-link.fitted:hover,
.sk-estimator-doc-link.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

/* Span, style for the box shown on hovering the info icon */
.sk-estimator-doc-link span {
  display: none;
  z-index: 9999;
  position: relative;
  font-weight: normal;
  right: .2ex;
  padding: .5ex;
  margin: .5ex;
  width: min-content;
  min-width: 20ex;
  max-width: 50ex;
  color: var(--sklearn-color-text);
  box-shadow: 2pt 2pt 4pt #999;
  /* unfitted */
  background: var(--sklearn-color-unfitted-level-0);
  border: .5pt solid var(--sklearn-color-unfitted-level-3);
}

.sk-estimator-doc-link.fitted span {
  /* fitted */
  background: var(--sklearn-color-fitted-level-0);
  border: var(--sklearn-color-fitted-level-3);
}

.sk-estimator-doc-link:hover span {
  display: block;
}

/* "?"-specific style due to the `<a>` HTML tag */

#sk-container-id-1 a.estimator_doc_link {
  float: right;
  font-size: 1rem;
  line-height: 1em;
  font-family: monospace;
  background-color: var(--sklearn-color-background);
  border-radius: 1rem;
  height: 1rem;
  width: 1rem;
  text-decoration: none;
  /* unfitted */
  color: var(--sklearn-color-unfitted-level-1);
  border: var(--sklearn-color-unfitted-level-1) 1pt solid;
}

#sk-container-id-1 a.estimator_doc_link.fitted {
  /* fitted */
  border: var(--sklearn-color-fitted-level-1) 1pt solid;
  color: var(--sklearn-color-fitted-level-1);
}

/* On hover */
#sk-container-id-1 a.estimator_doc_link:hover {
  /* unfitted */
  background-color: var(--sklearn-color-unfitted-level-3);
  color: var(--sklearn-color-background);
  text-decoration: none;
}

#sk-container-id-1 a.estimator_doc_link.fitted:hover {
  /* fitted */
  background-color: var(--sklearn-color-fitted-level-3);
}
</style><div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>DecisionTreeRegressor(max_depth=10, random_state=42)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator fitted sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label fitted sk-toggleable__label-arrow"><div><div>DecisionTreeRegressor</div></div><div><a class="sk-estimator-doc-link fitted" rel="noreferrer" target="_blank" href="https://scikit-learn.org/1.6/modules/generated/sklearn.tree.DecisionTreeRegressor.html">?<span>Documentation for DecisionTreeRegressor</span></a><span class="sk-estimator-doc-link fitted">i<span>Fitted</span></span></div></label><div class="sk-toggleable__content fitted"><pre>DecisionTreeRegressor(max_depth=10, random_state=42)</pre></div> </div></div></div></div>

The above model will be used to generate predictions for the target variables. Appropriate evaluation metrics will be computed afterwards to assess the model's performance.

```python
# calculate predictions for training and test set
y_train_pred = dtr.predict(x_train)
y_test_pred = dtr.predict(x_test)

```

```python
# calculate metrics for the training set
mse_train = mean_squared_error(y_train, y_train_pred)
rmse_train = np.sqrt(mse_train)
mae_train = mean_absolute_error(y_train, y_train_pred)
r2_train = r2_score(y_train, y_train_pred)
```

```python
# calculate metrics for the test set
mse_test = mean_squared_error(y_test, y_test_pred)
rmse_test = np.sqrt(mse_test)
mae_test = mean_absolute_error(y_test, y_test_pred)
r2_test = r2_score(y_test, y_test_pred)
```

```python
# display the results
print(f"======= Training Set =======")
print(f"Mean Squared Error:          {mse_train:.4f}")
print(f"Root Mean Squared Error:     {rmse_train:.4f}")
print(f"Mean Absolute Error:         {mae_train:.4f}")
print(f"R²(Coeff. of Determination): {r2_train:.4f}")

print(f"\n======= Test Set =======")
print(f"Mean Squared Error:          {mse_test:.4f}")
print(f"Root Mean Squared Error:     {rmse_test:.4f}")
print(f"Mean Absolute Error:         {mae_test:.4f}")
print(f"R²(Coeff. of Determination): {r2_test:.4f}")
```

    ======= Training Set =======
    Mean Squared Error:          3445.0150
    Root Mean Squared Error:     58.6943
    Mean Absolute Error:         37.5888
    R²(Coeff. of Determination): 0.9926

    ======= Test Set =======
    Mean Squared Error:          4002.0717
    Root Mean Squared Error:     63.2619
    Mean Absolute Error:         42.4149
    R²(Coeff. of Determination): 0.9917

Interpretation of the results. Keep in mind that the target variables ranges aproximately from 1000 to 4000

1. Mean Squared Error and Root Mean Squared Error. RMSE of 63.26 corresponds to a relative error of less than 3%. This indicates a very good model performance.
1. Mean Absolute Error. The MAE of 42.41 is less than 2% of the target variable range. Also very good result.
1. R² (R-squared score or Coefficient of Determination). The R² values of 0.9917 (test set) and 0.9926 (training set) indicate that the model explains over 99% of the variance in the data, which is an excellent outcome.

Visualizing the results will allow you to quickly and intuitively assess the quality of the trained model. The most common way to visualize in such cases is a graph of actual vs. predicted values.

```python
# Scatterplot for actual and predicted values
plt.figure(figsize=(8, 6))
sns.scatterplot(x=y_test, y=y_test_pred, alpha=0.7)
# Line of ideal fit
plt.plot([min(y_test), max(y_test)], [min(y_test), max(y_test)], color='red', linestyle='-')
plt.xlabel("Actual values")
plt.ylabel("Predicted values")
plt.title("Actual vs. Predicted Values - Test Set")
plt.grid(True)
plt.show()
```

![png](output_38_0.PNG)

Another way to present the outcome is to overlay the actual and predicted values of the target variable from the test set.

```python
# Sort the indexes to make the graph clear and easy to read
sorted_indices = np.argsort(y_test.index)
y_test_sorted = y_test.iloc[sorted_indices]
y_test_pred_sorted = y_test_pred[sorted_indices]

# Create the graph
plt.figure(figsize=(10, 6))
plt.plot(y_test_sorted.values, label="Actual values", linestyle='-', color='blue', alpha=0.7)
plt.plot(y_test_pred_sorted, label="Predicted values", linestyle='-', color='red', alpha=0.7)

# Title and labels
plt.xlabel("Observation no.")
plt.ylabel("Target variable")
plt.title("Comparison of Actual and Predicted Values - Test Set")
plt.legend()
plt.grid(True)

# display the plot
plt.show()
```

![png](output_40_0.PNG)

# Summary and Conclusions

The DecisionTreeRegressor machine learning model from the Python Scikit Learn library proved to be very effective in coping with the given task of predicting the heating load of a building. This is confirmed by very low relative errors and very good model fit coefficients as well as visualizations of the actual and predicted values of the target variable from the test set.

The article could end here. However, I would like to show some practital aspects of how it can be used.
Firstly, the model can be used in a dashboard, where user can input building parameters and get the predicted heating load as an outcome.

```python
import ipywidgets as widgets
from IPython.display import display

dtr  # <- this is the DecisionTreeRegressor ML Model trained above

# The list of input features from the dataset
feature_names = ['SurfaceArea', 'RoofArea', 'OverallHeight', 'GlazingArea']

# The range of the sliders correrponds to min and max values of each column
feature_ranges = {
    "SurfaceArea": (515, 810, 5),  # (min, max, step)
    "RoofArea": (110, 220, 1),
    "OverallHeight": (3.5, 7, 0.5),
    "GlazingArea": (0, 0.5, 0.1)
}

# Create the sliders
inputs = {name: widgets.FloatSlider(value=(min_val + max_val) / 2,
                                    min=min_val, max=max_val, step=step,
                                    description=name, continuous_update=False)
          for name, (min_val, max_val, step) in feature_ranges.items()}

# Button to triger prediction calculations after the features are set
btn_predict = widgets.Button(
    description="Predict Heating Load",
    style={'button_color': 'brown', 'font_weight': 'bold'},
    layout=widgets.Layout(width='200px')
)
output = widgets.Output()

def predict_heating_load(_):
    # Get data from the inputs and arrange them in a DataFrame
    input_data = pd.DataFrame([[inputs[name].value for name in x_train.columns]], columns=x_train.columns)

    # Use the model to predict heating load
    heating_load = dtr.predict(input_data)[0]

    # Display the outcome
    with output:
        output.clear_output()
        print(f"Predicted Total Heating Load: {heating_load:.2f} kWh")

# Assign the function to "on click" action
btn_predict.on_click(predict_heating_load)

# Display the dashboard
display(*inputs.values(), btn_predict, output)
```

![slider-dashboard](slider-dashboard.jpg)

Machine learning models developed in Jupyter Notebook (.ipynb) can also be successfully exported and applied in real-world scenarios. One effective way to deploy such models is by using MLflow – an open-source platform that allows to turn ML models into REST APIs and export them to popular cloud environments (Docker, Apache Spark, AWS SageMaker, Azure ML). This provides a fast and convenient path to production deployment, enabling the use of trained models in commercial applications.
