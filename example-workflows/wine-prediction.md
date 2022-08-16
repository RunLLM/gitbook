


<!-- ------------- New Cell ------------ -->


# Wine Quality Predictions




<!-- ------------- New Cell ------------ -->


### The Standard Model Development Process




<!-- ------------- New Cell ------------ -->


There is a rich ecosystem of tools for developing and training models in Python. 
This is the fun part of data science, however we see a fundamental problem in where this process ends.





<!-- ------------- New Cell ------------ -->


### What Happens Next?

The model should make predictions that everyone can use, so they can drink better wine :) 
but delivering predictions to an organization is:

1. manual, complicated and time-consuming.
2. error prone.
3. difficult to integrate with the business.

In short, these ^ are all examples of engineering complexity

We built Aqueduct, so you can deliver predictions without all the engineering complexity.




<!-- ------------- New Cell ------------ -->


This demo should take no more than 10 minutes and show you how Aqueduct handles all of the core components of **production data science**:

1) How to **deploy** a model, so you can run it anywhere 

2) How to **connect** your input and output data sources, so you can use the most recent input data and deliver fresh predictions where they need to be

3) How you can **add tests** and detect bad predictions, before they are published to external stakeholders

4) How our **web UI** helps you understand your workflows and see everything from a single surface

## Getting Started




<!-- ------------- New Cell ------------ -->


```python
# pip install aqueduct-ml
# aqueduct start
```




<!-- ------------- New Cell ------------ -->


```python
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.tree import DecisionTreeRegressor

import aqueduct as aq
from aqueduct import op, check, metric
```




<!-- ------------- New Cell ------------ -->


### Run and scale your predictions




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def featurize(df):
    # Convert back to numeric with missing values as NaN
    df['residual_sugar'] = pd.to_numeric(df['residual_sugar'], errors='coerce')
    print("missing residual sugar values:", df['residual_sugar'].isna().sum())
    
    # Replace the missing values by fitting a linear model to on other numeric columns
    imputer = LinearRegression()
    other_cols = df.columns[df.dtypes == 'float'].difference(['quality', 'residual_sugar', 'id'])
    imputer.fit(df.dropna()[other_cols], df.dropna()['residual_sugar'])
    predicted_sugar = imputer.predict(df[df['residual_sugar'].isna()][other_cols])
    df.loc[df['residual_sugar'].isna(), 'residual_sugar'] = predicted_sugar
    
    # Binary encode the wine color
    df['is_red'] = (df['color'] == 'red').astype('float')
    return df

@aq.op
def train_and_predict(df):
    
    # Convert the quality column to numerica and replace the "\\N" with NaN
    df['quality'] = pd.to_numeric(df['quality'], errors='coerce')
    
    # Fit a model to the columns that
    quality_model = DecisionTreeRegressor(max_depth=3)
    feature_columns = df.columns[df.dtypes == 'float'].difference(['quality', 'id'])
    print("Feature Columns", feature_columns)
    quality_model.fit(df.dropna()[feature_columns], df.dropna()['quality'])
    df['pred_quality'] = quality_model.predict(df[feature_columns])
    return df
```




<!-- ------------- New Cell ------------ -->


Adding this decorator to your code allows Aqueduct to package up this code so it can be run as a part of a workflow anywhere (cloud, server, laptop). 

It’s your code; run it anywhere.




<!-- ------------- New Cell ------------ -->


### Initializing Aqueduct Client




<!-- ------------- New Cell ------------ -->


```python
import aqueduct
client = aqueduct.Client("81QR4HYWFCX2G096MDZA3S5VKIO7PTNL","localhost:8080")
```




<!-- ------------- New Cell ------------ -->


### Connecting to your data




<!-- ------------- New Cell ------------ -->


```python
demodb = client.integration('aqueduct_demo')
wines = demodb.sql("select * from wine;")

```




<!-- ------------- New Cell ------------ -->


### Defining the pipeline




<!-- ------------- New Cell ------------ -->


```python
features = featurize(wines)
predictions = train_and_predict(features)

```




<!-- ------------- New Cell ------------ -->


The cool thing about this is that you can write your Python code, like you do regularly, and call your Python functions, just like you do regularly. But, now that you added the @op decorator, everything can run seamlessly as a part of an Aqueduct workflow.




<!-- ------------- New Cell ------------ -->


```python
predictions.save(demodb.config("pred_wine_quality", update_mode="replace"))
```




<!-- ------------- New Cell ------------ -->


```python
from textwrap import dedent
client.publish_flow(
    "Wine Ratings",
    dedent(
        """
        This workflow builds a model to predict missing ratings for wines
        and then uses that model to fill in missing ratings.
        """),
    schedule=aq.daily(),
    artifacts=[predictions]
)
```
**Output:**


```
<aqueduct.flow.Flow at 0x15080c250>
```






<!-- ------------- New Cell ------------ -->


At this point, we’ve defined a full workflow. 
1) we have told aqueduct where to get some data
2) we have defined the transformations that execute as a part of our workflow
3) we’ve set an output destination and published the workflow

All we need to do in the cell above is provide a name and a schedule for the workflow. 




<!-- ------------- New Cell ------------ -->


This pipeline will run as scheduled and all is well in the world, until things break.

Why do things break? 
We see a lot of reasons like changes in input data and model drift to name a few. 
The tricky part about these breaks is they don’t stop the predictions from being served, they just create bad predictions.




<!-- ------------- New Cell ------------ -->


### Monitoring your pipeline with Metrics and Checks




<!-- ------------- New Cell ------------ -->


Now what we can do, is update our workflow and put in metrics and checks, to ensure our workflow runs as expected and delivers accurate predictions. 

The neat part about this is you can define exactly what “wrong” looks like and get notified or even stop your pipeline from publishing the predictions when things go wrong.




<!-- ------------- New Cell ------------ -->


```python
@aq.metric()
def get_rmse(df):
    import numpy as np
    residuals = (df['quality'] - df['pred_quality']).dropna()
    print("Computing error using:", len(residuals), "rows.")
    return np.sqrt((residuals * residuals).mean())
```




<!-- ------------- New Cell ------------ -->


```python
rmse = get_rmse(predictions)
rmse.bound(upper = 1.0)
rmse.bound(upper = 3.0, severity="error")
```
**Output:**


```
<aqueduct.check_artifact.CheckArtifact at 0x150831bb0>
```






<!-- ------------- New Cell ------------ -->


```python
client.publish_flow(
    "Wine Ratings",
    dedent(
        """
        This workflow builds a model to predict missing ratings for wines
        and then uses that model to fill in missing ratings.
        """),
    schedule=aq.daily(),
    artifacts=[predictions, rmse]
)

```
**Output:**


```
<aqueduct.flow.Flow at 0x150859850>
```






<!-- ------------- New Cell ------------ -->


Aqueduct makes it easy to check your predictions so you can:

1. Prevent publication of bad predictions.
2. Know when, where and why your pipelines fail, so you can easily see what has been impacted, locate the error and fix it.

We let you *Sleep at night!*

Now onto the UI

