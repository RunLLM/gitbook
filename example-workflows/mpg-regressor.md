# MPG Regressor

If you've ever taken a data science course, you've undoubtedly come across the classic MPG dataset, with information about the engine specs, make, and model year of cars from the 1970s and early 1980s.

**You can find and download this notebook on GitHub** [**here**](https://github.com/aqueducthq/aqueduct/blob/main/examples/mpg-regressor/Predicting%20MPG.ipynb)**.**

As a simple introduction to how Aqueduct works, we're going to build a simple linear regression model in scikit-learn that takes the data we have about these cars and predicts their MPG. It's likely that many of you have built this exact model in the past, and we're not going to do anything innovative here, but we'd like to take the chance to show you how Aqueduct works.

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**

First, we're going to import the libraries we'll use — `numpy`, `pandas`, and `sklearn` in addition to Aqueduct — and create an Aqueduct client. In the cell below, we use `aq.get_apikey()` to get our Aqueduct API key, but **note that this only works if you're running your notebook on the same machine as the Aqueduct server**. If you're running your notebook elsewhere, make sure to grab your API key and insert it as the first argument below and to modify the address of the Aqueduct server as well.

```python
import aqueduct as aq
import numpy as np
import pandas as pd
import sklearn.linear_model

# If you're running your notebook on a separate machine from your
# Aqueduct server, change this to the address of your Aqueduct server.
address = "http://localhost:8080"

# If you're running youre notebook on a separate machine from your
# Aqueduct server, you will have to copy your API key here rather than
# using `get_apikey()`.
api_key = aq.get_apikey()
client = aq.Client(api_key, address)
```

Once we have our client, the first thing we'll do is load our data. Aqueduct has the ability to most common databases and storage systems (check out the Resources page on the Aqueduct UI). Here, we'll load a connection to the default `aqueduct_demo` database, which comes preloaded with a number of [common datasets](https://docs.aqueducthq.com/example-workflows/demo-data-warehouse).

Once we have a connection to the demo DB, we can run a SQL query to retrieve our base data.

```python
demodb = client.resource("aqueduct_demo")

# mpg_data is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
mpg_data = demodb.sql("SELECT * FROM mpg;")
```

When we get a handle to a dataset in Aqueduct, we can call `.get()` to materialize the underlying object (in this case, a Pandas `DataFrame`).

```python
# This gets the data in the underlying DataFrame. Note that you can't
# pass a DataFrame as an argument to a workflow; you must use the Aqueduct
# TableArtifact!
mpg_df = mpg_data.get()
mpg_df.dtypes
```

**Output:**

```
id                int64
mpg             float64
cylinders         int64
displacement    float64
horsepower       object
weight            int64
acceleration    float64
model_year        int64
origin           object
name             object
dtype: object
```

You might've noticed that while most of the data here is numerical, our `DataFrame` has type `object` for horsepower, which we'd normally expect to be numerical as well.

This likely means there's some invalid data buried in our horsepower column, so let's look for any non-numerical values that pop up in the horsepower column:

```python
# Pick out all the rows which have a horsepower column that does not match
# a regex that finds multiple integers.
mpg_df[mpg_df["horsepower"].str.match(r"^\d+$").notna()]
```

**Output**

|     | id  | mpg  | cylinders | displacement | horsepower | weight | acceleration | model\_year | origin | name                 |
| --- | --- | ---- | --------- | ------------ | ---------- | ------ | ------------ | ----------- | ------ | -------------------- |
| 32  | 32  | 25.0 | 4         | 98.0         |            | 2046   | 19.0         | 71          | usa    | ford pinto           |
| 126 | 126 | 21.0 | 6         | 200.0        |            | 2875   | 17.0         | 74          | usa    | ford maverick        |
| 330 | 330 | 40.9 | 4         | 85.0         |            | 1835   | 17.3         | 80          | europe | renault lecar deluxe |
| 336 | 336 | 23.6 | 4         | 140.0        |            | 2905   | 14.3         | 80          | usa    | ford mustang cobra   |
| 354 | 354 | 34.5 | 4         | 100.0        |            | 2320   | 15.8         | 81          | europe | renault 18i          |
| 374 | 374 | 23.0 | 4         | 151.0        |            | 3035   | 20.5         | 82          | usa    | amc concord dl       |

There seem to be 6 rows with  as the value for `horsepower`. We'll assume that means that we didn't have valid measurements for those fields, so we'll replace the `horsepower` field with the average of all the valid horsepower values we have. Ideally, we'd encode some information about the car itself here, but we'll keep things simple for now.

Let's go ahead and write a function called `clean_horsepower` that takes the average of all the valid `horsepower` values and replaces the invalid values with the average:

```python
# The @op decorator here allows Aqueduct to run this function as
# a part of an Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@aq.op
def clean_horsepower(mpg_df):
    """
    clean_horsepower takes in a DataFrame with MPG data about existing cars
    and cleans the horsepower column in that DataFrame. It does this by
    first calculating the average horsepower of all of the valid entries,
    then replacing any invalid entry (`\\N`) with the previously-calculated
    average value.
    """
    # Calculate the average horsepower for all of the valid values in the horsepower column.
    avg_valid_hp = mpg_df[mpg_df["horsepower"].str.match(r"^\d+$").isna()]["horsepower"].mean()

    # Replace all the invalid values with the new average value and convert to integers.
    mpg_df.loc[mpg_df["horsepower"].str.match(r"^\d+$").notna(), "horsepower"] = avg_valid_hp
    mpg_df["horsepower"] = mpg_df["horsepower"].astype(int)

    return mpg_df
```

You might've noticed that we added a decorator above `clean_horsepower` called `@aq.op`. What this decorator does is tell Aqueduct that we're going to use this function as a part of a workflow at some point. When we call `clean_horsepower` Aqueduct will note down that this function will process some data.

However, for testing purposes, we can run `clean_horsepower` locally (not as a part of a workflow) by calling `clean_horsepower.local`:

```python
# Calling `.local()` on an @op-annotated function allows us to execute the
# function locally for testing purposes. When a function is called with
# `.local()`, Aqueduct does not capture the function execution as a part of
# the definition of a workflow.
mpg_df = clean_horsepower.local(mpg_df)
mpg_df
```

**Output**

|     | id  | mpg  | cylinders | displacement | horsepower | weight | acceleration | model\_year | origin | name                      |
| --- | --- | ---- | --------- | ------------ | ---------- | ------ | ------------ | ----------- | ------ | ------------------------- |
| 0   | 0   | 18.0 | 8         | 307.0        | 130        | 3504   | 12.0         | 70          | usa    | chevrolet chevelle malibu |
| 1   | 1   | 15.0 | 8         | 350.0        | 165        | 3693   | 11.5         | 70          | usa    | buick skylark 320         |
| 2   | 2   | 18.0 | 8         | 318.0        | 150        | 3436   | 11.0         | 70          | usa    | plymouth satellite        |
| 3   | 3   | 16.0 | 8         | 304.0        | 150        | 3433   | 12.0         | 70          | usa    | amc rebel sst             |
| 4   | 4   | 17.0 | 8         | 302.0        | 140        | 3449   | 10.5         | 70          | usa    | ford torino               |
| ... | ... | ...  | ...       | ...          | ...        | ...    | ...          | ...         | ...    | ...                       |
| 393 | 393 | 27.0 | 4         | 140.0        | 86         | 2790   | 15.6         | 82          | usa    | ford mustang gl           |
| 394 | 394 | 44.0 | 4         | 97.0         | 52         | 2130   | 24.6         | 82          | europe | vw pickup                 |
| 395 | 395 | 32.0 | 4         | 135.0        | 84         | 2295   | 11.6         | 82          | usa    | dodge rampage             |
| 396 | 396 | 28.0 | 4         | 120.0        | 79         | 2625   | 18.6         | 82          | usa    | ford ranger               |
| 397 | 397 | 31.0 | 4         | 119.0        | 82         | 2720   | 19.4         | 82          | usa    | chevy s-10                |

398 rows × 10 columns

Now that we've cleaned our data, we're going to one-hot encode our categorical fields — `model_year` and `origin` — using the `pd.dummies` function:

```python
@aq.op
def one_hot_encode(mpg_df):
    """
    This function one-hot encodes the model_year and the origin columns
    in our MPG data. While model years might present as continuous variables,
    they are actually categorical; it's not clear that a car from 1972 has 'more'
    value in some way than a car from 1970.

    Note that we convert the model_year column from numerical values to string
    values to avoid Pandas serialization issues.
    """

    # We convert to `str` type here before one-hot encoding our
    # model_year because Pandas serialization often has issues
    # with numerically typed columns
    year_one_hot = pd.get_dummies(mpg_df["model_year"].astype(str))
    origin_one_hot = pd.get_dummies(mpg_df["origin"])

    return mpg_df.join(year_one_hot).join(origin_one_hot)
```

Again, we can test the results of our function by claling `one_hot_encode.local`:

```python
# Again, calling `.local()` here allows us to execute one_hot_encode
# for testing purposes without defining a stage in our workflow.
mpg_df = one_hot_encode.local(mpg_df)
mpg_df
```

**Output**

|     | id  | mpg  | cylinders | displacement | horsepower | weight | acceleration | model\_year | origin | name                      | ... | 76  | 77  | 78  | 79  | 80  | 81  | 82  | europe | japan | usa |
| --- | --- | ---- | --------- | ------------ | ---------- | ------ | ------------ | ----------- | ------ | ------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ------ | ----- | --- |
| 0   | 0   | 18.0 | 8         | 307.0        | 130        | 3504   | 12.0         | 70          | usa    | chevrolet chevelle malibu | ... | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0      | 0     | 1   |
| 1   | 1   | 15.0 | 8         | 350.0        | 165        | 3693   | 11.5         | 70          | usa    | buick skylark 320         | ... | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0      | 0     | 1   |
| 2   | 2   | 18.0 | 8         | 318.0        | 150        | 3436   | 11.0         | 70          | usa    | plymouth satellite        | ... | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0      | 0     | 1   |
| 3   | 3   | 16.0 | 8         | 304.0        | 150        | 3433   | 12.0         | 70          | usa    | amc rebel sst             | ... | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0      | 0     | 1   |
| 4   | 4   | 17.0 | 8         | 302.0        | 140        | 3449   | 10.5         | 70          | usa    | ford torino               | ... | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0      | 0     | 1   |
| ... | ... | ...  | ...       | ...          | ...        | ...    | ...          | ...         | ...    | ...                       | ... | ... | ... | ... | ... | ... | ... | ... | ...    | ...   | ... |
| 393 | 393 | 27.0 | 4         | 140.0        | 86         | 2790   | 15.6         | 82          | usa    | ford mustang gl           | ... | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0      | 0     | 1   |
| 394 | 394 | 44.0 | 4         | 97.0         | 52         | 2130   | 24.6         | 82          | europe | vw pickup                 | ... | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 1      | 0     | 0   |
| 395 | 395 | 32.0 | 4         | 135.0        | 84         | 2295   | 11.6         | 82          | usa    | dodge rampage             | ... | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0      | 0     | 1   |
| 396 | 396 | 28.0 | 4         | 120.0        | 79         | 2625   | 18.6         | 82          | usa    | ford ranger               | ... | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0      | 0     | 1   |
| 397 | 397 | 31.0 | 4         | 119.0        | 82         | 2720   | 19.4         | 82          | usa    | chevy s-10                | ... | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0      | 0     | 1   |

398 rows × 26 columns

Finally, we're going to log-normalize all of our numerical fields and test the results again:

```python
@aq.op
def log_norm(mpg_df):
    """
    This function log-normalizes the numerical columns in our MPG data -- cylinders,
    displacement, horsepower, weight, and acceleration -- using numpy np.log function.
    It adds new columns with the log-normalized values rather than replacing the existing
    columns.
    """
    log_feature_columns = ["cylinders", "displacement", "horsepower", "weight", "acceleration"]
    log_features = mpg_df.copy()
    for col in log_feature_columns:
        log_features["log_" + col] = np.log(log_features[col] + 1.0)
    return log_features
```

```python
mpg_df = log_norm.local(mpg_df)
mpg_df
```

**Output**

|     | id  | mpg  | cylinders | displacement | horsepower | weight | acceleration | model\_year | origin | name                      | ... | 81  | 82  | europe | japan | usa | log\_cylinders | log\_displacement | log\_horsepower | log\_weight | log\_acceleration |
| --- | --- | ---- | --------- | ------------ | ---------- | ------ | ------------ | ----------- | ------ | ------------------------- | --- | --- | --- | ------ | ----- | --- | -------------- | ----------------- | --------------- | ----------- | ----------------- |
| 0   | 0   | 18.0 | 8         | 307.0        | 130        | 3504   | 12.0         | 70          | usa    | chevrolet chevelle malibu | ... | 0   | 0   | 0      | 0     | 1   | 2.197225       | 5.730100          | 4.875197        | 8.161946    | 2.564949          |
| 1   | 1   | 15.0 | 8         | 350.0        | 165        | 3693   | 11.5         | 70          | usa    | buick skylark 320         | ... | 0   | 0   | 0      | 0     | 1   | 2.197225       | 5.860786          | 5.111988        | 8.214465    | 2.525729          |
| 2   | 2   | 18.0 | 8         | 318.0        | 150        | 3436   | 11.0         | 70          | usa    | plymouth satellite        | ... | 0   | 0   | 0      | 0     | 1   | 2.197225       | 5.765191          | 5.017280        | 8.142354    | 2.484907          |
| 3   | 3   | 16.0 | 8         | 304.0        | 150        | 3433   | 12.0         | 70          | usa    | amc rebel sst             | ... | 0   | 0   | 0      | 0     | 1   | 2.197225       | 5.720312          | 5.017280        | 8.141481    | 2.564949          |
| 4   | 4   | 17.0 | 8         | 302.0        | 140        | 3449   | 10.5         | 70          | usa    | ford torino               | ... | 0   | 0   | 0      | 0     | 1   | 2.197225       | 5.713733          | 4.948760        | 8.146130    | 2.442347          |
| ... | ... | ...  | ...       | ...          | ...        | ...    | ...          | ...         | ...    | ...                       | ... | ... | ... | ...    | ...   | ... | ...            | ...               | ...             | ...         | ...               |
| 393 | 393 | 27.0 | 4         | 140.0        | 86         | 2790   | 15.6         | 82          | usa    | ford mustang gl           | ... | 0   | 1   | 0      | 0     | 1   | 1.609438       | 4.948760          | 4.465908        | 7.934155    | 2.809403          |
| 394 | 394 | 44.0 | 4         | 97.0         | 52         | 2130   | 24.6         | 82          | europe | vw pickup                 | ... | 0   | 1   | 1      | 0     | 0   | 1.609438       | 4.584967          | 3.970292        | 7.664347    | 3.242592          |
| 395 | 395 | 32.0 | 4         | 135.0        | 84         | 2295   | 11.6         | 82          | usa    | dodge rampage             | ... | 0   | 1   | 0      | 0     | 1   | 1.609438       | 4.912655          | 4.442651        | 7.738924    | 2.533697          |
| 396 | 396 | 28.0 | 4         | 120.0        | 79         | 2625   | 18.6         | 82          | usa    | ford ranger               | ... | 0   | 1   | 0      | 0     | 1   | 1.609438       | 4.795791          | 4.382027        | 7.873217    | 2.975530          |
| 397 | 397 | 31.0 | 4         | 119.0        | 82         | 2720   | 19.4         | 82          | usa    | chevy s-10                | ... | 0   | 1   | 0      | 0     | 1   | 1.609438       | 4.787492          | 4.418841        | 7.908755    | 3.015535          |

398 rows × 31 columns

Now that we've cleaned, one-hot encoded, and regularized our data, we're finally ready to train our data. We'll first define a list of our feature columns and then we'll use `sklearn.linear_model.LinearRegression` to fit a model that predicts our MPG:

```python
# Create a list of all our feature columns by selecting the log-normalized versions of our numerical
# columns and then adding the categorical columns we created in one_hot_encode, which are the
# one-hot columns for region of origin (europe, usa, japan) as well as the year the car was manufactured.
log_feature_columns = ["cylinders", "displacement", "horsepower", "weight", "acceleration"]
feature_columns = list(map(lambda x: "log_" + x, log_feature_columns))
feature_columns += map(lambda x: str(x), mpg_df["model_year"].unique().tolist())
feature_columns += mpg_df["origin"].unique().tolist()
feature_columns
```

**Output:**

```
['log_cylinders',
 'log_displacement',
 'log_horsepower',
 'log_weight',
 'log_acceleration',
 '70',
 '71',
 '72',
 '73',
 '74',
 '75',
 '76',
 '77',
 '78',
 '79',
 '80',
 '81',
 '82',
 'usa',
 'japan',
 'europe']
```

```python
model = sklearn.linear_model.LinearRegression()
model.fit(mpg_df[feature_columns], mpg_df["mpg"])
```

**Output**

```
LinearRegression()
```

LinearRegression

```
LinearRegression()
```

To see how effective our model is, we'll run our predictions against our training data and compare the results to the actual measurements. You can see that we're fairly accurate overall, but we're not particularly good at large outlier values:

```python
mpg_df["predicted_mpg"] = model.predict(mpg_df[feature_columns])
mpg_df[["mpg", "predicted_mpg"]]
```

**Output**

|     | mpg  | predicted\_mpg |
| --- | ---- | -------------- |
| 0   | 18.0 | 17.216879      |
| 1   | 15.0 | 15.021293      |
| 2   | 18.0 | 16.854917      |
| 3   | 16.0 | 16.675744      |
| 4   | 17.0 | 17.456320      |
| ... | ...  | ...            |
| 393 | 27.0 | 28.886285      |
| 394 | 44.0 | 36.001914      |
| 395 | 32.0 | 32.624318      |
| 396 | 28.0 | 29.900378      |
| 397 | 31.0 | 29.094823      |

398 rows × 2 columns

In reality, we'd probably want to spend more time featurizing our data and improving our model, but for this example, we'll declare victory here. We've built a reasonably effective model. The last thing we'll need to do is write a function that takes in the results of our featurization and generates predictions. We'll define this as another Aqueduct `@op` operator:

```python
@aq.op
def predict_mpg(normalized_data):
    """
    This function takes in the result of normalized and featurized MPG data and uses
    our sklearn LinearRegression model predict the MPG of the models based on some
    numerical features (listed below) and some categorical features (model year, region
    of origin).

    The resulting predictions are stored in a column called `predicted_mpg` on the resulting
    DataFrame.
    """
    log_feature_columns = ["cylinders", "displacement", "horsepower", "weight", "acceleration"]
    feature_columns = list(map(lambda x: "log_" + x, log_feature_columns))
    feature_columns += map(lambda x: str(x), mpg_df["model_year"].unique().tolist())
    feature_columns += mpg_df["origin"].unique().tolist()

    normalized_data["predicted_mpg"] = model.predict(normalized_data[feature_columns])
    return normalized_data
```

Now that we have all of our functions, we can define our workflow. It looks exactly like what we've been doing thus far, except we've removed the `.local()` on our function calls. As mentioned above, this now tells Aqueduct how to construct our MPG prediction workflows. Ideally, we'd set this workflow to run on data that we didn't use to train the model, but unfortunately our MPG dataset is quite limited.

```python
# We now use all of our @op-annotated functions to define our
# workflow in a few lines of Python.
hp_clean = clean_horsepower(mpg_data)
one_hot_encoded = one_hot_encode(hp_clean)
normalized = log_norm(one_hot_encoded)

predicted_mpg = predict_mpg(normalized)

# This tells Aqueduct to save the results in churn_table
# back to the demo DB we configured earlier.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published below.
demodb.save(predicted_mpg, table_name="predicted_mpg", update_mode="replace")
```

The last line above calls `.save()` on the `predicted_mpg` table. This tells Aqueduct that the results of `predicted_mpg` should be written to a database (in this case the `aqueduct_demo` DB we accessed earlier) into a table called `predicted_mpg`.

Now that we've defined our pipeline, we can call `.get()` on `predicted_mpg` to ensure that the pipeline executed successfully. Here, we can verify that our `predicted_mpg` matches what we computed locally:

```python
predicted_mpg.get()[["mpg", "predicted_mpg"]]
```

**Output**

|     | mpg  | predicted\_mpg |
| --- | ---- | -------------- |
| 0   | 18.0 | 17.216879      |
| 1   | 15.0 | 15.021293      |
| 2   | 18.0 | 16.854917      |
| 3   | 16.0 | 16.675744      |
| 4   | 17.0 | 17.456320      |
| ... | ...  | ...            |
| 393 | 27.0 | 28.886285      |
| 394 | 44.0 | 36.001914      |
| 395 | 32.0 | 32.624318      |
| 396 | 28.0 | 29.900378      |
| 397 | 31.0 | 29.094823      |

398 rows × 2 columns

The last thing we're going to do before publishing our new workflow is add some light validation. Here, we're going to calculate the RMSE of our predictions. We have a simple function to calculate RMSE below, but rather than tagging it with `@aq.op`, we've decorated this function with `@aq.metric`. A `metric` in Aqueduct is a measurement of your prediction workflow — a `metric` function takes in some data and returns a numerical value in return (here, it is the RMSE):

```python
# The @metric dectorator is similar to the @op decorator from
# above. The only difference is that a metric returns a numerical
# value that is tracked over and visualized over time.
@aq.metric
def get_rmse(predicted_mpg):
    """
    This metric function takes in a DataFrame and assumes it has two columns,
    `mpg` and `predicted_mpg`. It uses numpy to calculate the root mean squared
    error of the predicted quality column. It ignores any rows for which the quality
    column does not have a valid value.
    """
    import numpy as np

    residuals = (predicted_mpg["mpg"] - predicted_mpg["predicted_mpg"]).dropna()
    return np.sqrt((residuals * residuals).mean())
```

Once we have a `metric` function, we can call it on our data like any other function. We can also add a `bound` to an, which is a threshold that if crossed, will either raise a warning or an error. In this case, we add a hard upper bound of 3.0 to our RMSE:

```python
rmse = get_rmse(predicted_mpg)
# Enforce an upper bound of 3.0 on our RMSE; if the RMSE is
# above 3.0, then we will automatically fail the workflow.
rmse.bound(upper=3.0, severity="error")

rmse.get()
```

**Output:**

```
2.7015843391418457
```

And we're done! We can now call `publish_flow`, give our workflow a name, and tell Aqueduct which artifacts to publish as a part of it — in this case our `predicted_mpg` and the `rmse`. Aqueduct will automatically detect everything that is required to publish those artifacts and create a workflow out of the code we've written in this notebook. If you navigate to the link provided in the response, you will see an interactive graph of your workflow that allows you to see the code that ran, the data it generated, and any logs or error messages.

```python
# This publishes all of the logic needed to create predicted_mpg
# and rmse to Aqueduct. The URL below will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
client.publish_flow(name="MPG Predictor", artifacts=[predicted_mpg, rmse])
```

**Output:**

```
<aqueduct.flow.Flow at 0x11c5d7280>
```
