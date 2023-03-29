


<!-- ------------- New Cell ------------ -->


# Predict Missing Wine Ratings

In this demo, we train and use multiple models to impute missing values.  We start with a dataset of wines consisting of key features like acidity. Some of the records are missing feature values. In addition, wine quality scores are given to some but not all of the wines. 

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/training_and_inference/Impute%20Missing%20Wine%20Data.ipynb).**

We will build a workflow that trains a linear model to impute the missing features from the other features and then train a decision tree model to rate the un-rated wines using the imputed features. 

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


```python
import pandas as pd
import aqueduct
from aqueduct import op, check, metric
```




<!-- ------------- New Cell ------------ -->


```python
# You can use `localhost` if you're running this notebook on the same machine as the server.
# If you're running your notebook on a separate machine from your
# Aqueduct server, change this to the address of your Aqueduct server.
address = "http://localhost:8080"

# If you're running youre notebook on a separate machine from your
# Aqueduct server, you will have to copy your API key here rather than
# using `get_apikey()`.
api_key = aqueduct.get_apikey()
client = aqueduct.Client(api_key, address)
```




<!-- ------------- New Cell ------------ -->


## Getting the Demo Data 

In this demo, we will use the wine table in the demo data warehouse.




<!-- ------------- New Cell ------------ -->


```python
demodb = client.integration("aqueduct_demo")

# wines is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
wines = demodb.sql("select * from wine;")
```




<!-- ------------- New Cell ------------ -->


```python
# This gets the head of the underlying DataFrame. Note that you can't
# pass a DataFrame as an argument to a workflow; you must use the Aqueduct
# TableArtifact!
wines.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>color</th>
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>residual_sugar</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>ph</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>white</td>
      <td>7.0</td>
      <td>0.17</td>
      <td>0.74</td>
      <td>\N</td>
      <td>0.045</td>
      <td>24.0</td>
      <td>126.0</td>
      <td>0.99420</td>
      <td>3.26</td>
      <td>0.38</td>
      <td>12.2</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>red</td>
      <td>7.7</td>
      <td>0.64</td>
      <td>0.21</td>
      <td>2.2</td>
      <td>0.077</td>
      <td>32.0</td>
      <td>133.0</td>
      <td>0.99560</td>
      <td>3.27</td>
      <td>0.45</td>
      <td>9.9</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>white</td>
      <td>6.8</td>
      <td>0.39</td>
      <td>0.34</td>
      <td>7.4</td>
      <td>0.020</td>
      <td>38.0</td>
      <td>133.0</td>
      <td>0.99212</td>
      <td>3.18</td>
      <td>0.44</td>
      <td>12.0</td>
      <td>\N</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>white</td>
      <td>6.3</td>
      <td>0.28</td>
      <td>0.47</td>
      <td>\N</td>
      <td>0.040</td>
      <td>61.0</td>
      <td>183.0</td>
      <td>0.99592</td>
      <td>3.12</td>
      <td>0.51</td>
      <td>9.5</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>white</td>
      <td>7.4</td>
      <td>0.35</td>
      <td>0.20</td>
      <td>13.9</td>
      <td>0.054</td>
      <td>63.0</td>
      <td>229.0</td>
      <td>0.99888</td>
      <td>3.11</td>
      <td>0.50</td>
      <td>8.9</td>
      <td>\N</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


Notice that we are missing some values in the `residual_sugar` column as well as the `quality` column which we would ultimately like to predict.




<!-- ------------- New Cell ------------ -->


## Cleaning the Data
There are some missing values in the residula sugar column that we need to clean.  Here we will replace the residual sugar with a value predicted by other columns




<!-- ------------- New Cell ------------ -->


```python
# The @op decorator here allows Aqueduct to run this function as
# a part of the Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@op()
def fix_residual_sugar(df):
    """
    This function takes in a DataFrame representing wines data and cleans
    the DataFrame by replacing any missing values in the `residual_sugar`
    column with the values that would be predicted based on the other columns.

    Internally, this function uses the sklearn LinearRegression model to
    predict what the values of the `residual_sugar` column should be when
    they are missing.
    """
    from sklearn.linear_model import LinearRegression

    # Convert residual_sugar back to numeric values with missing values as NaN
    df["residual_sugar"] = pd.to_numeric(df["residual_sugar"], errors="coerce")
    print("missing residual sugar values:", df["residual_sugar"].isna().sum())

    # Fit a LinearRegression model on the other numeric columns, which is everything but
    # quality, residual_sugar.
    imputer = LinearRegression()
    other_cols = df.columns[df.dtypes == "float"].difference(["quality", "residual_sugar", "id"])
    imputer.fit(df.dropna()[other_cols], df.dropna()["residual_sugar"])

    # Use our newly-trained imputer to predict the missing values of `residual_sugar`
    # and replace the NaN values with our new predicted values.
    predicted_sugar = imputer.predict(df[df["residual_sugar"].isna()][other_cols])
    df.loc[df["residual_sugar"].isna(), "residual_sugar"] = predicted_sugar
    return df
```




<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to execute fix_residual_sugar on wines
# as a part of our workflow. However, nothing is published (yet) until we
# call `publish_flow()` below. Calling `.get()` gives us a preview
# of the results, but the resulting value is not stored or published
# anywhere.
wines_cleaned = fix_residual_sugar(wines)
wines_cleaned.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>color</th>
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>residual_sugar</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>ph</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>white</td>
      <td>7.0</td>
      <td>0.17</td>
      <td>0.74</td>
      <td>10.027690</td>
      <td>0.045</td>
      <td>24.0</td>
      <td>126.0</td>
      <td>0.99420</td>
      <td>3.26</td>
      <td>0.38</td>
      <td>12.2</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>red</td>
      <td>7.7</td>
      <td>0.64</td>
      <td>0.21</td>
      <td>2.200000</td>
      <td>0.077</td>
      <td>32.0</td>
      <td>133.0</td>
      <td>0.99560</td>
      <td>3.27</td>
      <td>0.45</td>
      <td>9.9</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>white</td>
      <td>6.8</td>
      <td>0.39</td>
      <td>0.34</td>
      <td>7.400000</td>
      <td>0.020</td>
      <td>38.0</td>
      <td>133.0</td>
      <td>0.99212</td>
      <td>3.18</td>
      <td>0.44</td>
      <td>12.0</td>
      <td>\N</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>white</td>
      <td>6.3</td>
      <td>0.28</td>
      <td>0.47</td>
      <td>10.697218</td>
      <td>0.040</td>
      <td>61.0</td>
      <td>183.0</td>
      <td>0.99592</td>
      <td>3.12</td>
      <td>0.51</td>
      <td>9.5</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>white</td>
      <td>7.4</td>
      <td>0.35</td>
      <td>0.20</td>
      <td>13.900000</td>
      <td>0.054</td>
      <td>63.0</td>
      <td>229.0</td>
      <td>0.99888</td>
      <td>3.11</td>
      <td>0.50</td>
      <td>8.9</td>
      <td>\N</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


We also want to encode the color column as a boolean rather than a string. 




<!-- ------------- New Cell ------------ -->


```python
@op()
def encode_color(df):
    """
    This function takes in a DataFrame with data about wines
    and encodes whether the wine is a red wine or white wine
    as boolean value. This allows us to treat this as a categorical
    variable for a future model-training step.
    """
    df["is_red"] = (df["color"] == "red").astype("float")
    return df
```




<!-- ------------- New Cell ------------ -->


```python
# Again, we execute `encode_color` on `wines_cleaned` and use
# `.get()` to retrieve a preview of the results, but no resulting
# data is stored here.
featurized_wines = encode_color(wines_cleaned)
featurized_wines.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>color</th>
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>residual_sugar</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>ph</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
      <th>is_red</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>white</td>
      <td>7.0</td>
      <td>0.17</td>
      <td>0.74</td>
      <td>10.027690</td>
      <td>0.045</td>
      <td>24.0</td>
      <td>126.0</td>
      <td>0.99420</td>
      <td>3.26</td>
      <td>0.38</td>
      <td>12.2</td>
      <td>8.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>red</td>
      <td>7.7</td>
      <td>0.64</td>
      <td>0.21</td>
      <td>2.200000</td>
      <td>0.077</td>
      <td>32.0</td>
      <td>133.0</td>
      <td>0.99560</td>
      <td>3.27</td>
      <td>0.45</td>
      <td>9.9</td>
      <td>5.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>white</td>
      <td>6.8</td>
      <td>0.39</td>
      <td>0.34</td>
      <td>7.400000</td>
      <td>0.020</td>
      <td>38.0</td>
      <td>133.0</td>
      <td>0.99212</td>
      <td>3.18</td>
      <td>0.44</td>
      <td>12.0</td>
      <td>\N</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>white</td>
      <td>6.3</td>
      <td>0.28</td>
      <td>0.47</td>
      <td>10.697218</td>
      <td>0.040</td>
      <td>61.0</td>
      <td>183.0</td>
      <td>0.99592</td>
      <td>3.12</td>
      <td>0.51</td>
      <td>9.5</td>
      <td>6.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>white</td>
      <td>7.4</td>
      <td>0.35</td>
      <td>0.20</td>
      <td>13.900000</td>
      <td>0.054</td>
      <td>63.0</td>
      <td>229.0</td>
      <td>0.99888</td>
      <td>3.11</td>
      <td>0.50</td>
      <td>8.9</td>
      <td>\N</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


## Tracking number of a Labeled wines 

As a sanity check, we want to make sure there are enough wines with quality scores to render reliable predictions.




<!-- ------------- New Cell ------------ -->


```python
# The @metric dectorator is similar to the @op decorator from
# above. The only difference is that a metric returns a numerical
# value that is tracked over and visualized over time.
@metric()
def get_number_labeled_wines(df):
    """
    This function takes in a DataFrame of wine data and returns
    how many wines are missing a quality value. This function is based
    on the assumption that missing values are encoded as `\\N` in the
    underlying DataFrame. The typical, non-null value is expected to
    be numeric.
    """
    return (df["quality"] != "\\N").sum().astype(float)
```




<!-- ------------- New Cell ------------ -->


```python
# Similar to executing an @op-annotated function, executing a
# @metric-annotated function tells Aqueduct that get_number_labeled_wines
# will be a part of our workflow but does not publish any information
# till `publish_flow()` is called.
num_labeled = get_number_labeled_wines(featurized_wines)
num_labeled.get()
```
**Output:**


```
5997.0
```






<!-- ------------- New Cell ------------ -->


To avoid making potentially risky predictions, we also don't want to render predictions if we have fewer than 1000 labeled wines. 




<!-- ------------- New Cell ------------ -->


```python
# We add a lower bound of 1000 on our number of labeled wines.
# If this value drops below 1000, the severity argument tells
# us that we should automatically fail this workflow and not
# publish the results.
num_labeled.bound(lower=1000, severity="error")
```
**Output:**


```
<aqueduct.check_artifact.CheckArtifact at 0x1345126a0>
```






<!-- ------------- New Cell ------------ -->


## Predicting the Quality of Wines

In the following operator we:
1. Fit a decision tree model to the wines that do have quality ratings
2. Make quality rating predictions for all the wines in the table.




<!-- ------------- New Cell ------------ -->


```python
@op()
def predict_quality(df):
    """
    This function takes in data about wines and fills in any missing
    values for the wine quality by building a machine learning model
    that predicts the quality of the wine itself. The expectation for
    this function is that many or most of the wines will already be labeled
    with their quality. This function uses the existing wine quality
    labels as guidance to train its model and fills in missing
    values with the model.

    Under the hood, this function uses sklearn's DecisionTreeRegressor
    model to predict the missing wines' qualities.
    """
    from sklearn.tree import DecisionTreeRegressor

    # Convert the quality column to numerica and replace the "\N" with NaN
    df["quality"] = pd.to_numeric(df["quality"], errors="coerce")

    # Fit a model to the columns that are of numerical types but aren't the wine's
    # ID or the quality that we're predicting.
    quality_model = DecisionTreeRegressor(max_depth=3)
    feature_columns = df.columns[df.dtypes == "float"].difference(["quality", "id"])
    print("Feature Columns", feature_columns)
    quality_model.fit(df.dropna()[feature_columns], df.dropna()["quality"])

    # Add a `pred_quality` column with the predicted quality for each wine.
    df["pred_quality"] = quality_model.predict(df[feature_columns])
    return df
```




<!-- ------------- New Cell ------------ -->


```python
predicted_quality = predict_quality(featurized_wines)
predicted_quality.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>color</th>
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>residual_sugar</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>ph</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
      <th>is_red</th>
      <th>pred_quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>white</td>
      <td>7.0</td>
      <td>0.17</td>
      <td>0.74</td>
      <td>10.027690</td>
      <td>0.045</td>
      <td>24.0</td>
      <td>126.0</td>
      <td>0.99420</td>
      <td>3.26</td>
      <td>0.38</td>
      <td>12.2</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>6.653566</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>red</td>
      <td>7.7</td>
      <td>0.64</td>
      <td>0.21</td>
      <td>2.200000</td>
      <td>0.077</td>
      <td>32.0</td>
      <td>133.0</td>
      <td>0.99560</td>
      <td>3.27</td>
      <td>0.45</td>
      <td>9.9</td>
      <td>5.0</td>
      <td>1.0</td>
      <td>5.569721</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>white</td>
      <td>6.8</td>
      <td>0.39</td>
      <td>0.34</td>
      <td>7.400000</td>
      <td>0.020</td>
      <td>38.0</td>
      <td>133.0</td>
      <td>0.99212</td>
      <td>3.18</td>
      <td>0.44</td>
      <td>12.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>6.653566</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>white</td>
      <td>6.3</td>
      <td>0.28</td>
      <td>0.47</td>
      <td>10.697218</td>
      <td>0.040</td>
      <td>61.0</td>
      <td>183.0</td>
      <td>0.99592</td>
      <td>3.12</td>
      <td>0.51</td>
      <td>9.5</td>
      <td>6.0</td>
      <td>0.0</td>
      <td>5.255291</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>white</td>
      <td>7.4</td>
      <td>0.35</td>
      <td>0.20</td>
      <td>13.900000</td>
      <td>0.054</td>
      <td>63.0</td>
      <td>229.0</td>
      <td>0.99888</td>
      <td>3.11</td>
      <td>0.50</td>
      <td>8.9</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>5.255291</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


## Checking Our Predictions

As a sanity check, we also verify that the wine quality predictions are reasonable. We'll do this by defining another `metric` on the `predicted_quality` table that calculates the RMSE of the predictions for the wines for which we have actual labels.




<!-- ------------- New Cell ------------ -->


```python
@metric()
def get_rmse(df):
    """
    This metric function takes in a DataFrame and assumes it has two columns,
    `quality` and `pred_quality`. It uses numpy to calculate the root mean squared
    error of the predicted quality column. It ignores any rows for which the quality
    column does not have a valid value.
    """
    import numpy as np

    residuals = (df["quality"] - df["pred_quality"]).dropna()
    print("Computing error using:", len(residuals), "rows.")
    return np.sqrt((residuals * residuals).mean())
```




<!-- ------------- New Cell ------------ -->


```python
rmse = get_rmse(predicted_quality)
rmse.get()
```
**Output:**


```
0.7440468072891235
```






<!-- ------------- New Cell ------------ -->


Here we set two bounds on the error.  The first is a warning and the second we set to an error to avoid rendering potentially egregiously bad predictions.




<!-- ------------- New Cell ------------ -->


```python
# We set two bounds on the RMSE. If the RMSE is above 1.0,
# we will get an error, but we will not fail the workflow;
# however, if the RMSE is above 3.0, we will fail the whole
# workflow.
rmse.bound(upper=1.0)
rmse.bound(upper=3.0, severity="error")
```
**Output:**


```
<aqueduct.check_artifact.CheckArtifact at 0x13451cd30>
```






<!-- ------------- New Cell ------------ -->


## Saving the Predicted Wine Quality





<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to save the results in predicted_quality
# back to the demo DB we configured earlier.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published in the next cell.
demodb.save(predicted_quality, table_name="pred_wine_quality", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


## Publishing the Workflow





<!-- ------------- New Cell ------------ -->


```python
# This publishes all of the logic needed to create predicted_quality
# rmse, and num_labeled to Aqueduct. The URL below will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
from textwrap import dedent

client.publish_flow(
    "Wine Ratings",
    dedent(
        """
        This workflow builds a model to predict missing ratings for wines 
        and then uses that model to fill in missing ratings.
        """
    ),
    # Uncomment the following line to schedule on a daily basis.
    # schedule=aqueduct.daily(),
    artifacts=[predicted_quality, rmse, num_labeled],
)
```
**Output:**


```
<aqueduct.flow.Flow at 0x134430fa0>
```






<!-- ------------- New Cell ------------ -->


Clicking on the URL above will take you to the Aqueudct UI where you can see the workflow that we just created! On the Aqueduct UI, you'll be able to see the DAG of operators we just created, click into any of those operators, and see the data and metadata associated with each stage of the pipeline.

