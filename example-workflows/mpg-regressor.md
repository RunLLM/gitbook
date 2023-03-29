


<!-- ------------- New Cell ------------ -->


# Predicting MPG

If you've ever taken a data science course, you've undoubtedly come across the classic MPG dataset, with information about the engine specs, make, and model year of cars from the 1970s and early 1980s. 

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/mpg-regressor/Predicting%20MPG.ipynb).**

As a simple introduction to how Aqueduct works, we're going to build a simple linear regression model in scikit-learn that takes the data we have about these cars and predicts their MPG. It's likely that many of you have built this exact model in the past, and we're not going to do anything innovative here, but we'd like to take the chance to show you how Aqueduct works.

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


First, we're going to import the libraries we'll use — `numpy`, `pandas`, and `sklearn` in addition to Aqueduct — and create an Aqueduct client. In the cell below, we use `aq.get_apikey()` to get our Aqueduct API key, but **note that this only works if you're running your notebook on the same machine as the Aqueduct server**. If you're running your notebook elsewhere, make sure to grab your API key and insert it as the first argument below and to modify the address of the Aqueduct server as well.




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


Once we have our client, the first thing we'll do is load our data. Aqueduct has the ability to most common databases and storage systems (check out the Integrations page on the Aqueduct UI). Here, we'll load a connection to the default `aqueduct_demo` database, which comes preloaded with a number of [common datasets](https://docs.aqueducthq.com/example-workflows/demo-data-warehouse). 

Once we have a connection to the demo DB, we can run a SQL query to retrieve our base data.




<!-- ------------- New Cell ------------ -->


```python
demodb = client.integration("aqueduct_demo")

# mpg_data is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
mpg_data = demodb.sql("SELECT * FROM mpg;")
```




<!-- ------------- New Cell ------------ -->


When we get a handle to a dataset in Aqueduct, we can call `.get()` to materialize the underlying object (in this case, a Pandas `DataFrame`).




<!-- ------------- New Cell ------------ -->


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






<!-- ------------- New Cell ------------ -->


You might've noticed that while most of the data here is numerical, our `DataFrame` has type `object` for horsepower, which we'd normally expect to be numerical as well. 

This likely means there's some invalid data buried in our horsepower column, so let's look for any non-numerical values that pop up in the horsepower column:




<!-- ------------- New Cell ------------ -->


```python
# Pick out all the rows which have a horsepower column that does not match
# a regex that finds multiple integers.
mpg_df[mpg_df["horsepower"].str.match(r"^\d+$").notna()]
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>mpg</th>
      <th>cylinders</th>
      <th>displacement</th>
      <th>horsepower</th>
      <th>weight</th>
      <th>acceleration</th>
      <th>model_year</th>
      <th>origin</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>32</th>
      <td>32</td>
      <td>25.0</td>
      <td>4</td>
      <td>98.0</td>
      <td>\N</td>
      <td>2046</td>
      <td>19.0</td>
      <td>71</td>
      <td>usa</td>
      <td>ford pinto</td>
    </tr>
    <tr>
      <th>126</th>
      <td>126</td>
      <td>21.0</td>
      <td>6</td>
      <td>200.0</td>
      <td>\N</td>
      <td>2875</td>
      <td>17.0</td>
      <td>74</td>
      <td>usa</td>
      <td>ford maverick</td>
    </tr>
    <tr>
      <th>330</th>
      <td>330</td>
      <td>40.9</td>
      <td>4</td>
      <td>85.0</td>
      <td>\N</td>
      <td>1835</td>
      <td>17.3</td>
      <td>80</td>
      <td>europe</td>
      <td>renault lecar deluxe</td>
    </tr>
    <tr>
      <th>336</th>
      <td>336</td>
      <td>23.6</td>
      <td>4</td>
      <td>140.0</td>
      <td>\N</td>
      <td>2905</td>
      <td>14.3</td>
      <td>80</td>
      <td>usa</td>
      <td>ford mustang cobra</td>
    </tr>
    <tr>
      <th>354</th>
      <td>354</td>
      <td>34.5</td>
      <td>4</td>
      <td>100.0</td>
      <td>\N</td>
      <td>2320</td>
      <td>15.8</td>
      <td>81</td>
      <td>europe</td>
      <td>renault 18i</td>
    </tr>
    <tr>
      <th>374</th>
      <td>374</td>
      <td>23.0</td>
      <td>4</td>
      <td>151.0</td>
      <td>\N</td>
      <td>3035</td>
      <td>20.5</td>
      <td>82</td>
      <td>usa</td>
      <td>amc concord dl</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


There seem to be 6 rows with `\N` as the value for `horsepower`. We'll assume that means that we didn't have valid measurements for those fields, so we'll replace the `horsepower` field with the average of all the valid horsepower values we have. Ideally, we'd encode some information about the car itself here, but we'll keep things simple for now.

Let's go ahead and write a function called `clean_horsepower` that takes the average of all the valid `horsepower` values and replaces the invalid values with the average:




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


You might've noticed that we added a decorator above `clean_horsepower` called `@aq.op`. What this decorator does is tell Aqueduct that we're going to use this function as a part of a workflow at some point. When we call `clean_horsepower` Aqueduct will note down that this function will process some data.

However, for testing purposes, we can run `clean_horsepower` locally (not as a part of a workflow) by calling `clean_horsepower.local`:




<!-- ------------- New Cell ------------ -->


```python
# Calling `.local()` on an @op-annotated function allows us to execute the
# function locally for testing purposes. When a function is called with
# `.local()`, Aqueduct does not capture the function execution as a part of
# the definition of a workflow.
mpg_df = clean_horsepower.local(mpg_df)
mpg_df
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>mpg</th>
      <th>cylinders</th>
      <th>displacement</th>
      <th>horsepower</th>
      <th>weight</th>
      <th>acceleration</th>
      <th>model_year</th>
      <th>origin</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>18.0</td>
      <td>8</td>
      <td>307.0</td>
      <td>130</td>
      <td>3504</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>chevrolet chevelle malibu</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>15.0</td>
      <td>8</td>
      <td>350.0</td>
      <td>165</td>
      <td>3693</td>
      <td>11.5</td>
      <td>70</td>
      <td>usa</td>
      <td>buick skylark 320</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>18.0</td>
      <td>8</td>
      <td>318.0</td>
      <td>150</td>
      <td>3436</td>
      <td>11.0</td>
      <td>70</td>
      <td>usa</td>
      <td>plymouth satellite</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>16.0</td>
      <td>8</td>
      <td>304.0</td>
      <td>150</td>
      <td>3433</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>amc rebel sst</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>17.0</td>
      <td>8</td>
      <td>302.0</td>
      <td>140</td>
      <td>3449</td>
      <td>10.5</td>
      <td>70</td>
      <td>usa</td>
      <td>ford torino</td>
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
    </tr>
    <tr>
      <th>393</th>
      <td>393</td>
      <td>27.0</td>
      <td>4</td>
      <td>140.0</td>
      <td>86</td>
      <td>2790</td>
      <td>15.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford mustang gl</td>
    </tr>
    <tr>
      <th>394</th>
      <td>394</td>
      <td>44.0</td>
      <td>4</td>
      <td>97.0</td>
      <td>52</td>
      <td>2130</td>
      <td>24.6</td>
      <td>82</td>
      <td>europe</td>
      <td>vw pickup</td>
    </tr>
    <tr>
      <th>395</th>
      <td>395</td>
      <td>32.0</td>
      <td>4</td>
      <td>135.0</td>
      <td>84</td>
      <td>2295</td>
      <td>11.6</td>
      <td>82</td>
      <td>usa</td>
      <td>dodge rampage</td>
    </tr>
    <tr>
      <th>396</th>
      <td>396</td>
      <td>28.0</td>
      <td>4</td>
      <td>120.0</td>
      <td>79</td>
      <td>2625</td>
      <td>18.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford ranger</td>
    </tr>
    <tr>
      <th>397</th>
      <td>397</td>
      <td>31.0</td>
      <td>4</td>
      <td>119.0</td>
      <td>82</td>
      <td>2720</td>
      <td>19.4</td>
      <td>82</td>
      <td>usa</td>
      <td>chevy s-10</td>
    </tr>
  </tbody>
</table>
<p>398 rows × 10 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


Now that we've cleaned our data, we're going to one-hot encode our categorical fields — `model_year` and `origin` — using the `pd.dummies` function:




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


Again, we can test the results of our function by claling `one_hot_encode.local`:




<!-- ------------- New Cell ------------ -->


```python
# Again, calling `.local()` here allows us to execute one_hot_encode
# for testing purposes without defining a stage in our workflow.
mpg_df = one_hot_encode.local(mpg_df)
mpg_df
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>mpg</th>
      <th>cylinders</th>
      <th>displacement</th>
      <th>horsepower</th>
      <th>weight</th>
      <th>acceleration</th>
      <th>model_year</th>
      <th>origin</th>
      <th>name</th>
      <th>...</th>
      <th>76</th>
      <th>77</th>
      <th>78</th>
      <th>79</th>
      <th>80</th>
      <th>81</th>
      <th>82</th>
      <th>europe</th>
      <th>japan</th>
      <th>usa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>18.0</td>
      <td>8</td>
      <td>307.0</td>
      <td>130</td>
      <td>3504</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>chevrolet chevelle malibu</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>15.0</td>
      <td>8</td>
      <td>350.0</td>
      <td>165</td>
      <td>3693</td>
      <td>11.5</td>
      <td>70</td>
      <td>usa</td>
      <td>buick skylark 320</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>18.0</td>
      <td>8</td>
      <td>318.0</td>
      <td>150</td>
      <td>3436</td>
      <td>11.0</td>
      <td>70</td>
      <td>usa</td>
      <td>plymouth satellite</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>16.0</td>
      <td>8</td>
      <td>304.0</td>
      <td>150</td>
      <td>3433</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>amc rebel sst</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>17.0</td>
      <td>8</td>
      <td>302.0</td>
      <td>140</td>
      <td>3449</td>
      <td>10.5</td>
      <td>70</td>
      <td>usa</td>
      <td>ford torino</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
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
      <th>393</th>
      <td>393</td>
      <td>27.0</td>
      <td>4</td>
      <td>140.0</td>
      <td>86</td>
      <td>2790</td>
      <td>15.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford mustang gl</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>394</th>
      <td>394</td>
      <td>44.0</td>
      <td>4</td>
      <td>97.0</td>
      <td>52</td>
      <td>2130</td>
      <td>24.6</td>
      <td>82</td>
      <td>europe</td>
      <td>vw pickup</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>395</th>
      <td>395</td>
      <td>32.0</td>
      <td>4</td>
      <td>135.0</td>
      <td>84</td>
      <td>2295</td>
      <td>11.6</td>
      <td>82</td>
      <td>usa</td>
      <td>dodge rampage</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>396</th>
      <td>396</td>
      <td>28.0</td>
      <td>4</td>
      <td>120.0</td>
      <td>79</td>
      <td>2625</td>
      <td>18.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford ranger</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>397</th>
      <td>397</td>
      <td>31.0</td>
      <td>4</td>
      <td>119.0</td>
      <td>82</td>
      <td>2720</td>
      <td>19.4</td>
      <td>82</td>
      <td>usa</td>
      <td>chevy s-10</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>398 rows × 26 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


Finally, we're going to log-normalize all of our numerical fields and test the results again:




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


```python
mpg_df = log_norm.local(mpg_df)
mpg_df
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>mpg</th>
      <th>cylinders</th>
      <th>displacement</th>
      <th>horsepower</th>
      <th>weight</th>
      <th>acceleration</th>
      <th>model_year</th>
      <th>origin</th>
      <th>name</th>
      <th>...</th>
      <th>81</th>
      <th>82</th>
      <th>europe</th>
      <th>japan</th>
      <th>usa</th>
      <th>log_cylinders</th>
      <th>log_displacement</th>
      <th>log_horsepower</th>
      <th>log_weight</th>
      <th>log_acceleration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>18.0</td>
      <td>8</td>
      <td>307.0</td>
      <td>130</td>
      <td>3504</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>chevrolet chevelle malibu</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2.197225</td>
      <td>5.730100</td>
      <td>4.875197</td>
      <td>8.161946</td>
      <td>2.564949</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>15.0</td>
      <td>8</td>
      <td>350.0</td>
      <td>165</td>
      <td>3693</td>
      <td>11.5</td>
      <td>70</td>
      <td>usa</td>
      <td>buick skylark 320</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2.197225</td>
      <td>5.860786</td>
      <td>5.111988</td>
      <td>8.214465</td>
      <td>2.525729</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>18.0</td>
      <td>8</td>
      <td>318.0</td>
      <td>150</td>
      <td>3436</td>
      <td>11.0</td>
      <td>70</td>
      <td>usa</td>
      <td>plymouth satellite</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2.197225</td>
      <td>5.765191</td>
      <td>5.017280</td>
      <td>8.142354</td>
      <td>2.484907</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>16.0</td>
      <td>8</td>
      <td>304.0</td>
      <td>150</td>
      <td>3433</td>
      <td>12.0</td>
      <td>70</td>
      <td>usa</td>
      <td>amc rebel sst</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2.197225</td>
      <td>5.720312</td>
      <td>5.017280</td>
      <td>8.141481</td>
      <td>2.564949</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>17.0</td>
      <td>8</td>
      <td>302.0</td>
      <td>140</td>
      <td>3449</td>
      <td>10.5</td>
      <td>70</td>
      <td>usa</td>
      <td>ford torino</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2.197225</td>
      <td>5.713733</td>
      <td>4.948760</td>
      <td>8.146130</td>
      <td>2.442347</td>
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
      <th>393</th>
      <td>393</td>
      <td>27.0</td>
      <td>4</td>
      <td>140.0</td>
      <td>86</td>
      <td>2790</td>
      <td>15.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford mustang gl</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1.609438</td>
      <td>4.948760</td>
      <td>4.465908</td>
      <td>7.934155</td>
      <td>2.809403</td>
    </tr>
    <tr>
      <th>394</th>
      <td>394</td>
      <td>44.0</td>
      <td>4</td>
      <td>97.0</td>
      <td>52</td>
      <td>2130</td>
      <td>24.6</td>
      <td>82</td>
      <td>europe</td>
      <td>vw pickup</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1.609438</td>
      <td>4.584967</td>
      <td>3.970292</td>
      <td>7.664347</td>
      <td>3.242592</td>
    </tr>
    <tr>
      <th>395</th>
      <td>395</td>
      <td>32.0</td>
      <td>4</td>
      <td>135.0</td>
      <td>84</td>
      <td>2295</td>
      <td>11.6</td>
      <td>82</td>
      <td>usa</td>
      <td>dodge rampage</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1.609438</td>
      <td>4.912655</td>
      <td>4.442651</td>
      <td>7.738924</td>
      <td>2.533697</td>
    </tr>
    <tr>
      <th>396</th>
      <td>396</td>
      <td>28.0</td>
      <td>4</td>
      <td>120.0</td>
      <td>79</td>
      <td>2625</td>
      <td>18.6</td>
      <td>82</td>
      <td>usa</td>
      <td>ford ranger</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1.609438</td>
      <td>4.795791</td>
      <td>4.382027</td>
      <td>7.873217</td>
      <td>2.975530</td>
    </tr>
    <tr>
      <th>397</th>
      <td>397</td>
      <td>31.0</td>
      <td>4</td>
      <td>119.0</td>
      <td>82</td>
      <td>2720</td>
      <td>19.4</td>
      <td>82</td>
      <td>usa</td>
      <td>chevy s-10</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1.609438</td>
      <td>4.787492</td>
      <td>4.418841</td>
      <td>7.908755</td>
      <td>3.015535</td>
    </tr>
  </tbody>
</table>
<p>398 rows × 31 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


Now that we've cleaned, one-hot encoded, and regularized our data, we're finally ready to train our data. We'll first define a list of our feature columns and then we'll use `sklearn.linear_model.LinearRegression` to fit a model that predicts our MPG:




<!-- ------------- New Cell ------------ -->


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






<!-- ------------- New Cell ------------ -->


```python
model = sklearn.linear_model.LinearRegression()
model.fit(mpg_df[feature_columns], mpg_df["mpg"])
```
**Output**
<div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>LinearRegression()</pre></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div>




<!-- ------------- New Cell ------------ -->


To see how effective our model is, we'll run our predictions against our training data and compare the results to the actual measurements. You can see that we're fairly accurate overall, but we're not particularly good at large outlier values:




<!-- ------------- New Cell ------------ -->


```python
mpg_df["predicted_mpg"] = model.predict(mpg_df[feature_columns])
mpg_df[["mpg", "predicted_mpg"]]
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mpg</th>
      <th>predicted_mpg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>18.0</td>
      <td>17.216879</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15.0</td>
      <td>15.021293</td>
    </tr>
    <tr>
      <th>2</th>
      <td>18.0</td>
      <td>16.854917</td>
    </tr>
    <tr>
      <th>3</th>
      <td>16.0</td>
      <td>16.675744</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17.0</td>
      <td>17.456320</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>393</th>
      <td>27.0</td>
      <td>28.886285</td>
    </tr>
    <tr>
      <th>394</th>
      <td>44.0</td>
      <td>36.001914</td>
    </tr>
    <tr>
      <th>395</th>
      <td>32.0</td>
      <td>32.624318</td>
    </tr>
    <tr>
      <th>396</th>
      <td>28.0</td>
      <td>29.900378</td>
    </tr>
    <tr>
      <th>397</th>
      <td>31.0</td>
      <td>29.094823</td>
    </tr>
  </tbody>
</table>
<p>398 rows × 2 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


In reality, we'd probably want to spend more time featurizing our data and improving our model, but for this example, we'll declare victory here. We've built a reasonably effective model. The last thing we'll need to do is write a function that takes in the results of our featurization and generates predictions. We'll define this as another Aqueduct `@op` operator:




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


Now that we have all of our functions, we can define our workflow. It looks exactly like what we've been doing thus far, except we've removed the `.local()` on our function calls. As mentioned above, this now tells Aqueduct how to construct our MPG prediction workflows. Ideally, we'd set this workflow to run on data that we didn't use to train the model, but unfortunately our MPG dataset is quite limited.




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


The last line above calls `.save()` on the `predicted_mpg` table. This tells Aqueduct that the results of `predicted_mpg` should be written to a database (in this case the `aqueduct_demo` DB we accessed earlier) into a table called `predicted_mpg`.

Now that we've defined our pipeline, we can call `.get()` on `predicted_mpg` to ensure that the pipeline executed successfully. Here, we can verify that our `predicted_mpg` matches what we computed locally:




<!-- ------------- New Cell ------------ -->


```python
predicted_mpg.get()[["mpg", "predicted_mpg"]]
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mpg</th>
      <th>predicted_mpg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>18.0</td>
      <td>17.216879</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15.0</td>
      <td>15.021293</td>
    </tr>
    <tr>
      <th>2</th>
      <td>18.0</td>
      <td>16.854917</td>
    </tr>
    <tr>
      <th>3</th>
      <td>16.0</td>
      <td>16.675744</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17.0</td>
      <td>17.456320</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>393</th>
      <td>27.0</td>
      <td>28.886285</td>
    </tr>
    <tr>
      <th>394</th>
      <td>44.0</td>
      <td>36.001914</td>
    </tr>
    <tr>
      <th>395</th>
      <td>32.0</td>
      <td>32.624318</td>
    </tr>
    <tr>
      <th>396</th>
      <td>28.0</td>
      <td>29.900378</td>
    </tr>
    <tr>
      <th>397</th>
      <td>31.0</td>
      <td>29.094823</td>
    </tr>
  </tbody>
</table>
<p>398 rows × 2 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


The last thing we're going to do before publishing our new workflow is add some light validation. Here, we're going to calculate the RMSE of our predictions. We have a simple function to calculate RMSE below, but rather than tagging it with `@aq.op`, we've decorated this function with `@aq.metric`. A `metric` in Aqueduct is a measurement of your prediction workflow — a `metric` function takes in some data and returns a numerical value in return (here, it is the RMSE):




<!-- ------------- New Cell ------------ -->


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




<!-- ------------- New Cell ------------ -->


Once we have a `metric` function, we can call it on our data like any other function. We can also add a `bound` to an, which is a threshold that if crossed, will either raise a warning or an error. In this case, we add a hard upper bound of 3.0 to our RMSE:




<!-- ------------- New Cell ------------ -->


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






<!-- ------------- New Cell ------------ -->


And we're done! We can now call `publish_flow`, give our workflow a name, and tell Aqueduct which artifacts to publish as a part of it — in this case our `predicted_mpg` and the `rmse`. Aqueduct will automatically detect everything that is required to publish those artifacts and create a workflow out of the code we've written in this notebook. If you navigate to the link provided in the response, you will see an interactive graph of your workflow that allows you to see the code that ran, the data it generated, and any logs or error messages. 




<!-- ------------- New Cell ------------ -->


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



