


<!-- ------------- New Cell ------------ -->


# Classifying Diabetes Risk

In this example, we'll use a classic dataset with the health characteristics of several hundred patients and whether or not they have diabetes to build a K-nearest neighbors classifier for diabetes risk. 

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/diabetes-classifier/Classifying%20Diabetes%20Risk.ipynb).**

This notebook is adapted from the [most upvoted Kaggle notebook](https://www.kaggle.com/code/shrutimechlearn/step-by-step-diabetes-classification-knn-detailed) that does data cleaning and build a KNN classifier on the Pima diabetes dataset.

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


```python
# First, we'll load our dataset in from a catalog of common datasets on GitHub.
# The CSV we're loading from doesn't have column headers, so we provide the names
# of the columns to Pandas when reading the CSV
import pandas as pd

df = pd.read_csv(
    "https://raw.githubusercontent.com/jbrownlee/Datasets/master/pima-indians-diabetes.csv",
    names=[
        "pregnancies",
        "glucose",
        "diastolic_bp",
        "skin_thickness",
        "2_hr_insulin",
        "bmi",
        "pedigree_fn",
        "age",
        "has_diabetes",
    ],
    header=None,
)
print(df.dtypes)
df
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pregnancies</th>
      <th>glucose</th>
      <th>diastolic_bp</th>
      <th>skin_thickness</th>
      <th>2_hr_insulin</th>
      <th>bmi</th>
      <th>pedigree_fn</th>
      <th>age</th>
      <th>has_diabetes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6</td>
      <td>148</td>
      <td>72</td>
      <td>35</td>
      <td>0</td>
      <td>33.6</td>
      <td>0.627</td>
      <td>50</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>85</td>
      <td>66</td>
      <td>29</td>
      <td>0</td>
      <td>26.6</td>
      <td>0.351</td>
      <td>31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8</td>
      <td>183</td>
      <td>64</td>
      <td>0</td>
      <td>0</td>
      <td>23.3</td>
      <td>0.672</td>
      <td>32</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>89</td>
      <td>66</td>
      <td>23</td>
      <td>94</td>
      <td>28.1</td>
      <td>0.167</td>
      <td>21</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>137</td>
      <td>40</td>
      <td>35</td>
      <td>168</td>
      <td>43.1</td>
      <td>2.288</td>
      <td>33</td>
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
    </tr>
    <tr>
      <th>763</th>
      <td>10</td>
      <td>101</td>
      <td>76</td>
      <td>48</td>
      <td>180</td>
      <td>32.9</td>
      <td>0.171</td>
      <td>63</td>
      <td>0</td>
    </tr>
    <tr>
      <th>764</th>
      <td>2</td>
      <td>122</td>
      <td>70</td>
      <td>27</td>
      <td>0</td>
      <td>36.8</td>
      <td>0.340</td>
      <td>27</td>
      <td>0</td>
    </tr>
    <tr>
      <th>765</th>
      <td>5</td>
      <td>121</td>
      <td>72</td>
      <td>23</td>
      <td>112</td>
      <td>26.2</td>
      <td>0.245</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>766</th>
      <td>1</td>
      <td>126</td>
      <td>60</td>
      <td>0</td>
      <td>0</td>
      <td>30.1</td>
      <td>0.349</td>
      <td>47</td>
      <td>1</td>
    </tr>
    <tr>
      <th>767</th>
      <td>1</td>
      <td>93</td>
      <td>70</td>
      <td>31</td>
      <td>0</td>
      <td>30.4</td>
      <td>0.315</td>
      <td>23</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>768 rows × 9 columns</p>
</div>




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


We know from the [dataset definition](https://github.com/jbrownlee/Datasets/blob/master/pima-indians-diabetes.names) that there are going to be missing values, but when we look at the types of the columns above, we only have integers. We know that oftentimes, missing values are encoded as 0s in datasets, so we'll see how many values in each column are 0. 

Notice that for some columns (`pregnancies`), 0 is a valid value, so we'll ignore that. The `has_diabetes` column also uses 0 to indicate that the user doesn't have diabetes, so we can ignore that count as well.




<!-- ------------- New Cell ------------ -->


```python
(df == 0).sum()
```
**Output:**


```
pregnancies       111
glucose             5
diastolic_bp       35
skin_thickness    227
2_hr_insulin      374
bmi                11
pedigree_fn         0
age                 0
has_diabetes      500
dtype: int64
```






<!-- ------------- New Cell ------------ -->


With so much missing data, especially in the insulin and skin thickness measurements, it will be tough to build a model. Normally, we'd start visualizing our data to see what the distribution of the data across various splits is, but here, we'll cheat a little bit. Many of our friends over at Kaggle have built great visualizations for this dataset. In this example, we're using [this](https://www.kaggle.com/code/shrutimechlearn/step-by-step-diabetes-classification-knn-detailed) notebook, which shows that we have normal distributions for `glucose` and `diastolic_bp` and skewed distributions for `skin_thickness`, `2_hr_insulin`, and `bmi`. 

We'll write an interpolation function that replaces the 0 values in normal distributions with the mean of the dataset and replaces the 0 values in the skewed distributions with the median of the dataset. 




<!-- ------------- New Cell ------------ -->


```python
# The @op decorator here allows Aqueduct to run this function as
# a part of an Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@aq.op
def interpolate_missing_values(diabetes_df):
    """
    This function interpolates missing values for our diabetes dataset. For the glucose
    and diastolic_bp columns, this function assumes a normal distribution, calculates the
    mean of the non-zero values, and replaces all the 0 values with the mean.

    For the skin_thickness, bmi, and 2_hr_insulin column, this function assumes a skewed
    distribution and instead replaces the 0 values with the median of the non-0 values.
    """

    result = diabetes_df.copy()

    # As per our Kaggle guide, the glucose and diastolic BP values are normally distributed,
    # so we plug in the mean value of those columns for the missing values.
    result["glucose"].replace(
        0, int(diabetes_df[diabetes_df["glucose"] != 0]["glucose"].mean()), inplace=True
    )
    result["diastolic_bp"].replace(
        0, int(diabetes_df[diabetes_df["diastolic_bp"] != 0]["diastolic_bp"].mean()), inplace=True
    )

    # skin_thickness, 2_hr_insulin, and bmi are skewed distribution, so we take the median
    # values instead.
    result["skin_thickness"].replace(
        0,
        int(diabetes_df[diabetes_df["skin_thickness"] != 0]["skin_thickness"].median()),
        inplace=True,
    )
    result["2_hr_insulin"].replace(
        0, int(diabetes_df[diabetes_df["2_hr_insulin"] != 0]["2_hr_insulin"].median()), inplace=True
    )
    result["bmi"].replace(
        0, int(diabetes_df[diabetes_df["bmi"] != 0]["bmi"].median()), inplace=True
    )

    return result
```




<!-- ------------- New Cell ------------ -->


Now that we've defined our interpolation function, we can test it locally. When you call an `@op`-decorated function, Aqueduct assumes that it's a part of a larger workflow. For testing purposes, you can call a function with `.local()` that will tell Aqueduct to run this function once but to avoid building a larger workflow around it (yet).




<!-- ------------- New Cell ------------ -->


```python
# Calling `.local()` on an @op-annotated function allows us to execute the
# function locally for testing purposes. When a function is called with
# `.local()`, Aqueduct does not capture the function execution as a part of
# the definition of a workflow.
interpolated = interpolate_missing_values.local(df)
interpolated
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pregnancies</th>
      <th>glucose</th>
      <th>diastolic_bp</th>
      <th>skin_thickness</th>
      <th>2_hr_insulin</th>
      <th>bmi</th>
      <th>pedigree_fn</th>
      <th>age</th>
      <th>has_diabetes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6</td>
      <td>148</td>
      <td>72</td>
      <td>35</td>
      <td>125</td>
      <td>33.6</td>
      <td>0.627</td>
      <td>50</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>85</td>
      <td>66</td>
      <td>29</td>
      <td>125</td>
      <td>26.6</td>
      <td>0.351</td>
      <td>31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8</td>
      <td>183</td>
      <td>64</td>
      <td>29</td>
      <td>125</td>
      <td>23.3</td>
      <td>0.672</td>
      <td>32</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>89</td>
      <td>66</td>
      <td>23</td>
      <td>94</td>
      <td>28.1</td>
      <td>0.167</td>
      <td>21</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>137</td>
      <td>40</td>
      <td>35</td>
      <td>168</td>
      <td>43.1</td>
      <td>2.288</td>
      <td>33</td>
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
    </tr>
    <tr>
      <th>763</th>
      <td>10</td>
      <td>101</td>
      <td>76</td>
      <td>48</td>
      <td>180</td>
      <td>32.9</td>
      <td>0.171</td>
      <td>63</td>
      <td>0</td>
    </tr>
    <tr>
      <th>764</th>
      <td>2</td>
      <td>122</td>
      <td>70</td>
      <td>27</td>
      <td>125</td>
      <td>36.8</td>
      <td>0.340</td>
      <td>27</td>
      <td>0</td>
    </tr>
    <tr>
      <th>765</th>
      <td>5</td>
      <td>121</td>
      <td>72</td>
      <td>23</td>
      <td>112</td>
      <td>26.2</td>
      <td>0.245</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>766</th>
      <td>1</td>
      <td>126</td>
      <td>60</td>
      <td>29</td>
      <td>125</td>
      <td>30.1</td>
      <td>0.349</td>
      <td>47</td>
      <td>1</td>
    </tr>
    <tr>
      <th>767</th>
      <td>1</td>
      <td>93</td>
      <td>70</td>
      <td>31</td>
      <td>125</td>
      <td>30.4</td>
      <td>0.315</td>
      <td>23</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>768 rows × 9 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


Now that we have cleaned data that we can use, the next natural step when building is to rescale our data. Because KNN is a distance-based algorithm, we don't want the relative magnitudes of the different features to affect the correctness of our predictions. Here, we'll use scikit-learn's `StandardScaler` to rescale our numerical features.




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def rescale_data(interpolated_df):
    """
    This function takes in our diabetes dataset after 0-values have been
    filled in, and it uses sklearn's StandardScaler to rescale the data.
    It rescales all numerical columns, except for the has_diabetes column
    which is binary and is the output vairable we are trying to predict.
    """
    from sklearn.preprocessing import StandardScaler

    scaler = StandardScaler()

    all_columns = list(interpolated_df.columns)
    all_columns.remove("has_diabetes")  # Remove the outcome column.
    scaled = scaler.fit_transform(interpolated_df[all_columns])

    # Convert the result of our StandardScaler back to a DataFrame
    results = pd.DataFrame(scaled, columns=all_columns)
    results["has_diabetes"] = interpolated_df["has_diabetes"]

    return results
```




<!-- ------------- New Cell ------------ -->


```python
# Again, calling `.local()` here allows us to execute rescale_data
# for testing purposes without defining a stage in our workflow.
scaled = rescale_data.local(interpolated)
scaled
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pregnancies</th>
      <th>glucose</th>
      <th>diastolic_bp</th>
      <th>skin_thickness</th>
      <th>2_hr_insulin</th>
      <th>bmi</th>
      <th>pedigree_fn</th>
      <th>age</th>
      <th>has_diabetes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.639947</td>
      <td>0.865461</td>
      <td>-0.020645</td>
      <td>0.831114</td>
      <td>-0.609776</td>
      <td>0.167240</td>
      <td>0.468492</td>
      <td>1.425995</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.844885</td>
      <td>-1.205788</td>
      <td>-0.516132</td>
      <td>0.180566</td>
      <td>-0.609776</td>
      <td>-0.851551</td>
      <td>-0.365061</td>
      <td>-0.190672</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.233880</td>
      <td>2.016154</td>
      <td>-0.681294</td>
      <td>-0.469981</td>
      <td>-0.609776</td>
      <td>-1.331838</td>
      <td>0.604397</td>
      <td>-0.105584</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.844885</td>
      <td>-1.074281</td>
      <td>-0.516132</td>
      <td>-0.469981</td>
      <td>-0.003871</td>
      <td>-0.633239</td>
      <td>-0.920763</td>
      <td>-1.041549</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-1.141852</td>
      <td>0.503814</td>
      <td>-2.663240</td>
      <td>0.831114</td>
      <td>0.696707</td>
      <td>1.549885</td>
      <td>5.484909</td>
      <td>-0.020496</td>
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
    </tr>
    <tr>
      <th>763</th>
      <td>1.827813</td>
      <td>-0.679757</td>
      <td>0.309679</td>
      <td>2.240633</td>
      <td>0.810314</td>
      <td>0.065361</td>
      <td>-0.908682</td>
      <td>2.532136</td>
      <td>0</td>
    </tr>
    <tr>
      <th>764</th>
      <td>-0.547919</td>
      <td>0.010659</td>
      <td>-0.185807</td>
      <td>-0.036283</td>
      <td>-0.609776</td>
      <td>0.632973</td>
      <td>-0.398282</td>
      <td>-0.531023</td>
      <td>0</td>
    </tr>
    <tr>
      <th>765</th>
      <td>0.342981</td>
      <td>-0.022218</td>
      <td>-0.020645</td>
      <td>-0.469981</td>
      <td>0.166540</td>
      <td>-0.909768</td>
      <td>-0.685193</td>
      <td>-0.275760</td>
      <td>0</td>
    </tr>
    <tr>
      <th>766</th>
      <td>-0.844885</td>
      <td>0.142167</td>
      <td>-1.011618</td>
      <td>-0.469981</td>
      <td>-0.609776</td>
      <td>-0.342155</td>
      <td>-0.371101</td>
      <td>1.170732</td>
      <td>1</td>
    </tr>
    <tr>
      <th>767</th>
      <td>-0.844885</td>
      <td>-0.942773</td>
      <td>-0.185807</td>
      <td>0.397415</td>
      <td>-0.609776</td>
      <td>-0.298493</td>
      <td>-0.473785</td>
      <td>-0.871374</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>768 rows × 9 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


Finally, we're ready to train our model. We set out feature columns to be everything in the dataset except for the `has_diabetes` column and use `sklearn.neighbors.KNeighborsClassifier` to fit a new model.




<!-- ------------- New Cell ------------ -->


```python
from sklearn.neighbors import KNeighborsClassifier

feature_columns = list(scaled.columns)
feature_columns.remove("has_diabetes")

knn = KNeighborsClassifier(11)
knn.fit(scaled[feature_columns], scaled["has_diabetes"])
```
**Output**
<div id="sk-container-id-10" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>KNeighborsClassifier(n_neighbors=11)</pre></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-10" type="checkbox" checked><label for="sk-estimator-id-10" class="sk-toggleable__label sk-toggleable__label-arrow">KNeighborsClassifier</label><div class="sk-toggleable__content"><pre>KNeighborsClassifier(n_neighbors=11)</pre></div></div></div></div></div>




<!-- ------------- New Cell ------------ -->


Finally, we'll use our new KNN model to define a predict function, which will select all of our feature columns and return a new DataFrame with a column called `pred_has_diabetes`:




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def predict_diabetes(scaled):
    """
    This function accepts a rescaled diabetes dataset and uses our
    KNN model for diabetes prediction to predict whether the patient
    does or does not have diabetes.

    This function assumes that the columns in the scaled dataset are
    pregnancies, glucose, diastolic_bp, skin_thickness, 2_hr_insulin, bmi,
    pedigree_fn, and age. The model will not work correctly otherwise.

    The results are returned in a new column called pred_has_diabetes.
    """
    feature_columns = list(scaled.columns)
    feature_columns.remove("has_diabetes")

    scaled["pred_has_diabetes"] = knn.predict(scaled[feature_columns])
    return scaled
```




<!-- ------------- New Cell ------------ -->


Now that we've defined all the operators in our workflow, we can go ahead and define our workflow itself. The [Aqueduct demo DB](https://docs.aqueducthq.com/example-workflows/demo-data-warehouse) comes with the diabetes dataset prepackaged, so we can retrieve it directly and construct our workflow on top of it.




<!-- ------------- New Cell ------------ -->


```python
demodb = client.resource("aqueduct_demo")

# mpg_data is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
diabetes_data = demodb.sql("SELECT * FROM diabetes;")
diabetes_data.get()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pregnancies</th>
      <th>glucose</th>
      <th>diastolic_bp</th>
      <th>skin_thickness</th>
      <th>2_hr_insulin</th>
      <th>bmi</th>
      <th>pedigree_fn</th>
      <th>age</th>
      <th>has_diabetes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6</td>
      <td>148</td>
      <td>72</td>
      <td>35</td>
      <td>0</td>
      <td>33.6</td>
      <td>0.627</td>
      <td>50</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>85</td>
      <td>66</td>
      <td>29</td>
      <td>0</td>
      <td>26.6</td>
      <td>0.351</td>
      <td>31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8</td>
      <td>183</td>
      <td>64</td>
      <td>0</td>
      <td>0</td>
      <td>23.3</td>
      <td>0.672</td>
      <td>32</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>89</td>
      <td>66</td>
      <td>23</td>
      <td>94</td>
      <td>28.1</td>
      <td>0.167</td>
      <td>21</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>137</td>
      <td>40</td>
      <td>35</td>
      <td>168</td>
      <td>43.1</td>
      <td>2.288</td>
      <td>33</td>
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
    </tr>
    <tr>
      <th>763</th>
      <td>10</td>
      <td>101</td>
      <td>76</td>
      <td>48</td>
      <td>180</td>
      <td>32.9</td>
      <td>0.171</td>
      <td>63</td>
      <td>0</td>
    </tr>
    <tr>
      <th>764</th>
      <td>2</td>
      <td>122</td>
      <td>70</td>
      <td>27</td>
      <td>0</td>
      <td>36.8</td>
      <td>0.340</td>
      <td>27</td>
      <td>0</td>
    </tr>
    <tr>
      <th>765</th>
      <td>5</td>
      <td>121</td>
      <td>72</td>
      <td>23</td>
      <td>112</td>
      <td>26.2</td>
      <td>0.245</td>
      <td>30</td>
      <td>0</td>
    </tr>
    <tr>
      <th>766</th>
      <td>1</td>
      <td>126</td>
      <td>60</td>
      <td>0</td>
      <td>0</td>
      <td>30.1</td>
      <td>0.349</td>
      <td>47</td>
      <td>1</td>
    </tr>
    <tr>
      <th>767</th>
      <td>1</td>
      <td>93</td>
      <td>70</td>
      <td>31</td>
      <td>0</td>
      <td>30.4</td>
      <td>0.315</td>
      <td>23</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>768 rows × 9 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


```python
# We now use all of our @op-annotated functions to define our
# workflow in a few lines of Python.
interpolated = interpolate_missing_values(diabetes_data)
scaled = rescale_data(interpolated)
diabetes_preds = predict_diabetes(scaled)

# This tells Aqueduct to save the results in diabetes_preds
# back to the demo DB we configured earlier.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published below.
demodb.save(diabetes_preds, table_name="predicted_diabetes", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


Finally, before publishing our workflow, we're going to add a check to our workflow just to make sure our predictor works as expected. Here, we're going to write a simple check that ensures that all of our diabetes predictions return a binary class (0 for no diabetes and 1 for diabetes):




<!-- ------------- New Cell ------------ -->


```python
# The @check dectorator is similar to the @op decorator from
# above. The only difference is that a check returns a boolean
# value that is used to ensure the correctness of the workflow
@aq.check
def ensure_correct_classes(diabetes_preds):
    """
    This function ensures that the diabetes_preds DataFrame has the correct
    number and names of classes in the DataFrame. It ensures that there are
    two classes named 0 and 1, and it will fail otherwise.
    """
    classes = diabetes_preds["pred_has_diabetes"].value_counts()
    if len(classes) > 2:
        return False

    class_names = list(classes.keys())
    if 0 not in class_names or 1 not in class_names:
        return False

    return True
```




<!-- ------------- New Cell ------------ -->


```python
# Similar to a regular operator, a check can be called directly on
# an Aqueduct TableArtifact, and calling `.get()` on the result will
# give you the underlying computation.
has_correct_classes = ensure_correct_classes(diabetes_preds)
has_correct_classes.get()
```
**Output:**


```
True
```






<!-- ------------- New Cell ------------ -->


We're ready to publish our predictions! We can use the `publish_flow` API call to create a new workflow called "Diabetes Classifier" that is going to encapsulate all of the code that we've defined in this notebook, package it up as a rerunnable workflow, and deploy it onto the Aqueduct server. We don't do this below, but we could optionally set this workflow to run on a schedule (hourly, daily, weekly, etc.).




<!-- ------------- New Cell ------------ -->


```python
# This publishes all of the logic needed to create diabetes_preds
# and rmse to Aqueduct. The URL below will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
client.publish_flow(name="Diabetes Classifier", artifacts=[diabetes_preds])
```
**Output:**


```
<aqueduct.flow.Flow at 0x165f82040>
```






<!-- ------------- New Cell ------------ -->




