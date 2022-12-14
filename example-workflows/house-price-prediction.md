


<!-- ------------- New Cell ------------ -->


# Predicting House Prices

This notebook will take you through the process of setting up a workflow that featurizes a relatively complex [housing price dataset](https://www.kaggle.com/competitions/house-prices-advanced-regression-techniques), creating 4 models th predict housing prices, and ensembling the results by taking the average of the 4 models. If you're curious, the original Kaggle competition has a full description of the dataset. 

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/house-price-prediction/House%20Price%20Prediciton.ipynb).**

The credit for all the feature engineering that's done here goes to use Serigne on Kaggle, who put together this wonderful [notebook](https://www.kaggle.com/code/serigne/stacked-regressions-top-4-on-leaderboard/notebook) for this competition. 

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


```python
import aqueduct as aq
import pandas as pd
import numpy as np

# If you're running your notebook on a separate machine from your
# Aqueduct server, change this to the address of your Aqueduct server.
address = "http://localhost:8080"

# If you're running youre notebook on a separate machine from your
# Aqueduct server, you will have to copy your API key here rather than
# using `get_apikey()`.
api_key = aq.get_apikey()
client = aq.Client(api_key, address)
```




<!-- ------------- New Cell ------------ -->


```python
# First we'll load in our training data from a CSV file that's stored on
# the Aqueduct GitHub repo.
train_data = pd.read_csv("data/train.csv")
train_data
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>MSSubClass</th>
      <th>MSZoning</th>
      <th>LotFrontage</th>
      <th>LotArea</th>
      <th>Street</th>
      <th>Alley</th>
      <th>LotShape</th>
      <th>LandContour</th>
      <th>Utilities</th>
      <th>LotConfig</th>
      <th>...</th>
      <th>PoolArea</th>
      <th>PoolQC</th>
      <th>Fence</th>
      <th>MiscFeature</th>
      <th>MiscVal</th>
      <th>MoSold</th>
      <th>YrSold</th>
      <th>SaleType</th>
      <th>SaleCondition</th>
      <th>SalePrice</th>
    </tr>
    <tr>
      <th>Id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>60</td>
      <td>RL</td>
      <td>65.0</td>
      <td>8450</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>2</td>
      <td>2008</td>
      <td>WD</td>
      <td>Normal</td>
      <td>208500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20</td>
      <td>RL</td>
      <td>80.0</td>
      <td>9600</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>FR2</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>5</td>
      <td>2007</td>
      <td>WD</td>
      <td>Normal</td>
      <td>181500</td>
    </tr>
    <tr>
      <th>3</th>
      <td>60</td>
      <td>RL</td>
      <td>68.0</td>
      <td>11250</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>IR1</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>9</td>
      <td>2008</td>
      <td>WD</td>
      <td>Normal</td>
      <td>223500</td>
    </tr>
    <tr>
      <th>4</th>
      <td>70</td>
      <td>RL</td>
      <td>60.0</td>
      <td>9550</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>IR1</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Corner</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>2</td>
      <td>2006</td>
      <td>WD</td>
      <td>Abnorml</td>
      <td>140000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>60</td>
      <td>RL</td>
      <td>84.0</td>
      <td>14260</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>IR1</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>FR2</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>12</td>
      <td>2008</td>
      <td>WD</td>
      <td>Normal</td>
      <td>250000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1456</th>
      <td>60</td>
      <td>RL</td>
      <td>62.0</td>
      <td>7917</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>8</td>
      <td>2007</td>
      <td>WD</td>
      <td>Normal</td>
      <td>175000</td>
    </tr>
    <tr>
      <th>1457</th>
      <td>20</td>
      <td>RL</td>
      <td>85.0</td>
      <td>13175</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>MnPrv</td>
      <td>NaN</td>
      <td>0</td>
      <td>2</td>
      <td>2010</td>
      <td>WD</td>
      <td>Normal</td>
      <td>210000</td>
    </tr>
    <tr>
      <th>1458</th>
      <td>70</td>
      <td>RL</td>
      <td>66.0</td>
      <td>9042</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>GdPrv</td>
      <td>Shed</td>
      <td>2500</td>
      <td>5</td>
      <td>2010</td>
      <td>WD</td>
      <td>Normal</td>
      <td>266500</td>
    </tr>
    <tr>
      <th>1459</th>
      <td>20</td>
      <td>RL</td>
      <td>68.0</td>
      <td>9717</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>4</td>
      <td>2010</td>
      <td>WD</td>
      <td>Normal</td>
      <td>142125</td>
    </tr>
    <tr>
      <th>1460</th>
      <td>20</td>
      <td>RL</td>
      <td>75.0</td>
      <td>9937</td>
      <td>Pave</td>
      <td>NaN</td>
      <td>Reg</td>
      <td>Lvl</td>
      <td>AllPub</td>
      <td>Inside</td>
      <td>...</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>6</td>
      <td>2008</td>
      <td>WD</td>
      <td>Normal</td>
      <td>147500</td>
    </tr>
  </tbody>
</table>
<p>1460 rows × 80 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


There's a whole bunch of columns here describing the housing data, everything from the zoning of the area, the type of lot, the utilities available to the house, whether it has a basement or a pool, and its square footage. We're not going to describe all of the data here, but you can check out the original [Kaggle competition](https://www.kaggle.com/competitions/house-prices-advanced-regression-techniques) if you're curious. 

With all this data, we're naturally going to need to do some data cleaning. The first thing we're going to do is fill in any missing values with reasonable defaults in the `fill_missing_data` function below. Some of our columns will be filled with a categorical `None`, some will be filled with a 0, and others will be filled with the modal or median value of the column. There's a lot of data here, so we're not going to dive into exactly why, but if you're curious, check out the notebook written by Serigne that explains why [here](https://www.kaggle.com/code/serigne/stacked-regressions-top-4-on-leaderboard/data).




<!-- ------------- New Cell ------------ -->


```python
# The @op decorator here allows Aqueduct to run this function as
# a part of an Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@aq.op
def fill_missing_data(raw_house_data):
    """
    This function fills in any missing data from our housing dataset. Depending on the
    type of columns (categorical or numerical) and it's distributional properties (if numerical)
    we either fill missing values with None, with a zero, or with the modal value of the
    column. In addition, we:
    - drop the Utilities column because it is same for all but one of the houses
    - drop the SalePrice column because it's our prediction target
    - calculate the total square footage of the house by summing the basement and 1st and 2nd floors' SF
    """
    if "SalePrice" in raw_house_data:
        raw_house_data = raw_house_data.drop(["SalePrice"], axis=1)

    none_cols = [
        "PoolQC",
        "MiscFeature",
        "Alley",
        "Fence",
        "FireplaceQu",
        "GarageType",
        "GarageFinish",
        "GarageQual",
        "GarageCond",
        "BsmtQual",
        "BsmtCond",
        "BsmtExposure",
        "BsmtFinType1",
        "BsmtFinType2",
        "MasVnrType",
        "MSSubClass",
    ]
    for col in none_cols:
        raw_house_data[col].fillna("None", inplace=True)

    zero_cols = [
        "GarageYrBlt",
        "GarageArea",
        "GarageCars",
        "BsmtFinSF1",
        "BsmtFinSF2",
        "BsmtUnfSF",
        "TotalBsmtSF",
        "BsmtFullBath",
        "BsmtHalfBath",
        "MasVnrArea",
    ]
    for col in zero_cols:
        raw_house_data[col].fillna(0, inplace=True)

    modal_cols = ["MSZoning", "Electrical", "KitchenQual", "Exterior1st", "Exterior2nd", "SaleType"]

    for col in modal_cols:
        raw_house_data[col].fillna(raw_house_data[col].mode()[0], inplace=True)

    raw_house_data.drop(["Utilities"], axis=1, inplace=True)
    raw_house_data["Functional"].fillna("Typ", inplace=True)

    raw_house_data["LotFrontage"] = raw_house_data.groupby("Neighborhood")["LotFrontage"].transform(
        lambda x: x.fillna(x.median())
    )

    raw_house_data["TotalSF"] = (
        raw_house_data["TotalBsmtSF"] + raw_house_data["1stFlrSF"] + raw_house_data["2ndFlrSF"]
    )

    return raw_house_data
```




<!-- ------------- New Cell ------------ -->


```python
# Calling `.local()` on an @op-annotated function allows us to execute the
# function locally for testing purposes. When a function is called with
# `.local()`, Aqueduct does not capture the function execution as a part of
# the definition of a workflow.
filled = fill_missing_data.local(train_data)
```




<!-- ------------- New Cell ------------ -->


Now that we have our data cleaned, the next thing we need to do is encode our categorical columns. Some of our categorical columns are going to have ordinal properties, where the categories are meaningfully ordered, and so we use scikit-learn's `LabelEncoder` which encodes values in the order that they are seen. 




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def encode_labels(cleaned_data):
    """
    In this function, we take a subset of our categorical cols (listed in the
    `categorical_cols` variable below), and we use scikit-learn's LabelEncoder to
    encode the values in those columns because the category values here are
    ordinally meaningful.
    """
    cleaned_data["MSSubClass"].astype(str, copy=False)
    cleaned_data["OverallCond"].astype(str, copy=False)
    cleaned_data["YrSold"].astype(str, copy=False)
    cleaned_data["MoSold"].astype(str, copy=False)

    from sklearn.preprocessing import LabelEncoder

    categorical_cols = [
        "FireplaceQu",
        "BsmtQual",
        "BsmtCond",
        "GarageQual",
        "GarageCond",
        "ExterQual",
        "ExterCond",
        "HeatingQC",
        "PoolQC",
        "KitchenQual",
        "BsmtFinType1",
        "BsmtFinType2",
        "Functional",
        "Fence",
        "BsmtExposure",
        "GarageFinish",
        "LandSlope",
        "LotShape",
        "PavedDrive",
        "Street",
        "Alley",
        "CentralAir",
        "MSSubClass",
        "OverallCond",
        "YrSold",
        "MoSold",
    ]

    for col in categorical_cols:
        enc = LabelEncoder()
        enc.fit(list(cleaned_data[col].values))
        cleaned_data[col] = enc.transform(list(cleaned_data[col].values))

    return cleaned_data
```




<!-- ------------- New Cell ------------ -->


```python
encoded = encode_labels.local(filled)
```




<!-- ------------- New Cell ------------ -->


Next, we'll unskew our numerical features:




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def unskew_features(all_features):
    """
    This function uses scipy's boxcox1p method to unskew any numerical columns
    that have a skewness of at least 0.75, as calculated by scipy.stats.skew.
    In our training dataset, this function unskews 59 of the numerical features.
    """
    from scipy.special import boxcox1p
    from scipy.stats import skew

    numeric_feats = all_features.dtypes[all_features.dtypes != "object"].index
    skewed_feats = (
        all_features[numeric_feats].apply(lambda x: skew(x.dropna())).sort_values(ascending=False)
    )

    skewness = pd.DataFrame({"Skew": skewed_feats})
    skewness = skewness[abs(skewness) > 0.75]

    skewed_features = skewness.index
    lam = 0.15
    for feat in skewed_features:
        all_features[feat] = boxcox1p(all_features[feat], lam)

    return all_features
```




<!-- ------------- New Cell ------------ -->


```python
unskewed = unskew_features.local(encoded)
```




<!-- ------------- New Cell ------------ -->


Finally, we'll one-hot encode any remaining categorical variables:




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def one_hot_encode(all_features):
    """
    This function takes in an almost-featurized housing dataset and uses
    pandas' `get_dummies` function to one-hot encode any categorical variables
    that haven't previously been touched by our featurization process.
    """
    return pd.get_dummies(all_features)
```




<!-- ------------- New Cell ------------ -->


```python
featurized = one_hot_encode.local(unskewed)
```




<!-- ------------- New Cell ------------ -->


Now that we've finished featurizing our dataset, we're going to train a few models on our training data. We will train an ElasticNet model, a LASSO Regression model, a Gradient Boosting Regressor, and a Kernel Ridge Regessor model, and we'll put all of them in a list called `models`:




<!-- ------------- New Cell ------------ -->


```python
from sklearn.linear_model import ElasticNet, Lasso
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.kernel_ridge import KernelRidge
from sklearn.preprocessing import RobustScaler
from sklearn.pipeline import make_pipeline

# Train 4 different kinds of scikit-learn models on our training data.
# For the reasons why these 4 models were chosen, you can check out the
# original Kaggle competition linked earlier in this notebook.
ENet = make_pipeline(RobustScaler(), ElasticNet(alpha=0.0005, l1_ratio=0.9, random_state=3))
lasso = make_pipeline(RobustScaler(), Lasso(alpha=0.0005, random_state=1))
KRR = KernelRidge(alpha=0.6, kernel="polynomial", degree=2, coef0=2.5)
GBoost = GradientBoostingRegressor(
    n_estimators=3000,
    learning_rate=0.05,
    max_depth=4,
    max_features="sqrt",
    min_samples_leaf=15,
    min_samples_split=10,
    loss="huber",
    random_state=5,
)

models = [ENet, lasso, KRR, GBoost]

# Here, we use the SalePrice from the original training dataset as our y
# column, and we used the featurized DF we've created thus far as our X
# matrix.
y = train_data["SalePrice"]
X = featurized

for model in models:
    model.fit(X, y)
```




<!-- ------------- New Cell ------------ -->


Finally, now that we've trained our models, we can write a prediction function. In this case, our prediction function is going to be quite simple: We're going to take all four of the models we trained above, we're going to compute their house price predictions, and we're going to compute the average of the 4 models' predictions:




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def predict(raw_data, featurized_data):
    """
    This function takes in a fully featurized dataset, and it returns the average of the
    house price predictions made by the 4 models we've used. The average is unweighted, and it
    is computed by column-stacking the 4 prediction vectors, taking the average of the
    4 predictions.

    The results of this function are appended to the end of the original dataset.
    """
    predictions = []
    for model in models:
        predictions.append(model.predict(featurized_data))

    predictions = np.column_stack(predictions)
    raw_data["PredictedSalePrice"] = np.mean(predictions, axis=1)

    return raw_data
```




<!-- ------------- New Cell ------------ -->


Now that we've defined all of our functions, we can create a full Aqueduct pipeline in a few function calls. First, we load the latest copy (rather than our previous snapshot) of the housing data from the Aqueduct demo database. In this example, the data is the same, but presumably, your database or data warehouse gets updated more often than our examples. 🙂

We then invoke the Aqueduct operators that we defined above, and Aqueduct will automatically construct a pipeline for us. At the end of this cell, we see a preview of our predicted sale prices:




<!-- ------------- New Cell ------------ -->


```python
demo_db = client.integration("aqueduct_demo")
raw_data = demo_db.sql("select * from house_prices;")

filled_data = fill_missing_data(raw_data)
encoded_data = encode_labels(filled_data)
unskewed_data = unskew_features(encoded_data)
featurized_data = one_hot_encode(unskewed_data)

predictions = predict(raw_data, featurized_data)

df = predictions.get()
df[["PredictedSalePrice"]]
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PredictedSalePrice</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>212066.834996</td>
    </tr>
    <tr>
      <th>1</th>
      <td>185468.400665</td>
    </tr>
    <tr>
      <th>2</th>
      <td>215006.389258</td>
    </tr>
    <tr>
      <th>3</th>
      <td>164454.782472</td>
    </tr>
    <tr>
      <th>4</th>
      <td>292068.915282</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>2914</th>
      <td>84032.078128</td>
    </tr>
    <tr>
      <th>2915</th>
      <td>66575.662944</td>
    </tr>
    <tr>
      <th>2916</th>
      <td>170290.855793</td>
    </tr>
    <tr>
      <th>2917</th>
      <td>113928.671777</td>
    </tr>
    <tr>
      <th>2918</th>
      <td>226171.347077</td>
    </tr>
  </tbody>
</table>
<p>2919 rows × 1 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to save the results in predictions
# back to the demo DB we configured earlier, into a table called
# predicted_house_prices.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published below.
demo_db.save(predictions, table_name="predicted_house_prices", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


And we're done! We can now call `publish_flow`, give our workflow a name, and tell Aqueduct which artifacts to publish as a part of it — in this case our `predictions`. Aqueduct will automatically detect everything that is required to publish those artifacts and create a workflow out of the code we've written in this notebook. If you navigate to the link provided in the response, you will see an interactive graph of your workflow that allows you to see the code that ran, the data it generated, and any logs or error messages. 




<!-- ------------- New Cell ------------ -->


```python
# This publishes all of the logic needed to create predicted_mpg
# and rmse to Aqueduct and schedules the workflow
# to run on an hourly basis. The URL below will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
client.publish_flow(name="House Price Predictor", artifacts=[predictions])
```
**Output:**


```
<aqueduct.flow.Flow at 0x147cffcd0>
```



