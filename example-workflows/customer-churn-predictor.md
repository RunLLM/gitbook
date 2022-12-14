


<!-- ------------- New Cell ------------ -->


# Customer Churn Prediction

This is a tutorial that will walk you through creating your first useful workflow with Aqueduct. You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/churn_prediction/Build%20and%20Deploy%20Churn%20Ensemble.ipynb).

The philosophy behind the Aqueduct SDK is that you should be able to connect to your data systems, transform your data and generate predictions, and publish your results once you’re happy with them. This guide will walk you through installing your SDK, setting up a client, transforming some data, and publishing a workflow to the cloud. 

**Outline:**
1. We will use standard Python libraries to transform our data and build a simple ensemble of churn models.
2. We will use the Aqueduct Python API to deploy our churn prediction pipeline
3. We will visit the Aqueduct web interface to view our deployed prediction pipeline

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


--- 


## Step 1: Building the Churn Model

Before we do anything with Aqueduct, let’s first build some machine learning models. Aqueduct doesn’t have any opinions about where or how you do this.  In this example notebook, we will build a basic model churn prediction model using a customer churn dataset. 

Here is our training data:




<!-- ------------- New Cell ------------ -->


```python
import pandas as pd
import numpy as np
import aqueduct as aq

# Read some customer data from the Aqueduct repo.
customers_table = pd.read_csv(
    "https://raw.githubusercontent.com/aqueducthq/aqueduct/main/examples/churn_prediction/data/customers.csv"
)
churn_table = pd.read_csv(
    "https://raw.githubusercontent.com/aqueducthq/aqueduct/main/examples/churn_prediction/data/churn_data.csv"
)
pd.merge(customers_table, churn_table, on="cust_id").head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cust_id</th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
      <th>churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>False</td>
      <td>1.0</td>
      <td>True</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


### Feature Transformations

Most real-world machine learning relies on some form of feature transformation.  Because we have counts in our data we apply a log transformation to those counts.




<!-- ------------- New Cell ------------ -->


```python
# The @op decorator here allows Aqueduct to run this function as
# a part of an Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@aq.op
def log_featurize(cust: pd.DataFrame) -> pd.DataFrame:
    """
    log_featurize takes in customer data from the Aqueduct customers table
    and log normalizes the numerical columns using the numpy.log function.
    It skips the cust_id, using_deep_learning, and using_dbt columns because
    these are not numerical columns that require regularization.

    log_featurize adds all the log-normalized values into new columns, and
    maintains the original values as-is. In addition to the original company_size
    column, log_featurize will add a log_company_size column.
    """
    features = cust.copy()
    skip_cols = ["cust_id", "using_deep_learning", "using_dbt"]

    for col in features.columns.difference(skip_cols):
        features["log_" + col] = np.log(features[col] + 1.0)

    return features.drop(columns="cust_id")
```




<!-- ------------- New Cell ------------ -->


```python
# Calling `.local()` on an @op-annotated function allows us to execute the
# function locally for testing purposes. When a function is called with
# `.local()`, Aqueduct does not capture the function execution as a part of
# the definition of a workflow.
features_table = log_featurize.local(customers_table)
features_table.head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
      <th>log_company_size</th>
      <th>log_duration_months</th>
      <th>log_n_data_eng</th>
      <th>log_n_integrations</th>
      <th>log_n_rows</th>
      <th>log_n_support_tickets</th>
      <th>log_n_users</th>
      <th>log_n_workflows</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>3.401197</td>
      <td>0.693147</td>
      <td>1.098612</td>
      <td>1.791759</td>
      <td>7.604894</td>
      <td>1.386294</td>
      <td>1.098612</td>
      <td>1.609438</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>3.465736</td>
      <td>0.693147</td>
      <td>1.386294</td>
      <td>1.609438</td>
      <td>9.052399</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.386294</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>False</td>
      <td>1.0</td>
      <td>True</td>
      <td>3.401197</td>
      <td>1.386294</td>
      <td>0.693147</td>
      <td>1.386294</td>
      <td>8.929170</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.609438</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>3.526361</td>
      <td>1.609438</td>
      <td>1.386294</td>
      <td>1.609438</td>
      <td>8.363342</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.386294</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>3.367296</td>
      <td>0.693147</td>
      <td>1.098612</td>
      <td>1.386294</td>
      <td>7.667158</td>
      <td>0.000000</td>
      <td>0.693147</td>
      <td>1.098612</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


### Training the Model

In this example, we will train and ensemble two basic classifiers.  In practice, would probably do something more interesting but this will help illustrate post-processing logic (the ensemble function).




<!-- ------------- New Cell ------------ -->


```python
from sklearn.linear_model import LogisticRegression

linear_model = LogisticRegression(max_iter=10000)
linear_model.fit(features_table, churn_table["churn"])
```
**Output**
<div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>LogisticRegression(max_iter=10000)</pre></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label sk-toggleable__label-arrow">LogisticRegression</label><div class="sk-toggleable__content"><pre>LogisticRegression(max_iter=10000)</pre></div></div></div></div></div>




<!-- ------------- New Cell ------------ -->


```python
from sklearn.tree import DecisionTreeClassifier

decision_tree_model = DecisionTreeClassifier(max_depth=10, min_samples_split=3)
decision_tree_model.fit(features_table, churn_table["churn"])
```
**Output**
<div id="sk-container-id-2" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>DecisionTreeClassifier(max_depth=10, min_samples_split=3)</pre></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-2" type="checkbox" checked><label for="sk-estimator-id-2" class="sk-toggleable__label sk-toggleable__label-arrow">DecisionTreeClassifier</label><div class="sk-toggleable__content"><pre>DecisionTreeClassifier(max_depth=10, min_samples_split=3)</pre></div></div></div></div></div>




<!-- ------------- New Cell ------------ -->


### Making Predictions Locally 

Putting all the above pieces together we can define the **prediction workflow** as the following:




<!-- ------------- New Cell ------------ -->


```python
@aq.op
def predict_linear(features_table):
    """
    Generates predictions using the logistic regression model and
    returns a new DataFrame with a column called linear that has
    the likelihood of the customer churning.
    """
    return pd.DataFrame({"linear": linear_model.predict_proba(features_table)[:, 1]})

@aq.op
def predict_tree(features_table):
    """
    Generates predictions using the decision tree model and
    returns a new DataFrame with a column called tree that has
    the likelihood of the customer churning.
    """
    return pd.DataFrame({"tree": decision_tree_model.predict_proba(features_table)[:, 1]})

@aq.op
def predict_ensemble(customers_table, linear_pred_table, tree_pred_table):
    """
    predict_ensemble combines the results from our logistic regression
    and decision tree models by taking the average of the two models'
    probabilities that a user might churn. The resulting average is
    then assigned into the `prob_churn` column on the customers_table.
    """
    return customers_table.assign(prob_churn=linear_pred_table.join(tree_pred_table).mean(axis=1))
```




<!-- ------------- New Cell ------------ -->


```python
features_table = log_featurize.local(customers_table)
linear_pred_table = predict_linear.local(features_table)
tree_pred_table = predict_tree.local(features_table)
churn_table = predict_ensemble.local(customers_table, linear_pred_table, tree_pred_table)
```




<!-- ------------- New Cell ------------ -->


If we look at the output table we see that the `prob_churn` column contains the churn predictions.




<!-- ------------- New Cell ------------ -->


```python
churn_table.head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cust_id</th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
      <th>prob_churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>0.081036</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>0.146564</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>False</td>
      <td>1.0</td>
      <td>True</td>
      <td>0.155584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>False</td>
      <td>3.0</td>
      <td>True</td>
      <td>0.145398</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>True</td>
      <td>0.161406</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


Using standard python libraries we have trained two models and developed a simple workflow that combines feature transformations with models and post-prediction logic to estimate churn.  In the remainder of this notebook we will deploy the above prediction workflow to the cloud, integrate it with production data sources, and publish predictions to multiple destinations.  




<!-- ------------- New Cell ------------ -->


---

## Step 2: Deploying a Prediction Workflow

We will now deploy the above prediction workflow to the cloud.  You will need the API key for your Aqueduct server, which you can find on the `/account` page of your UI.  




<!-- ------------- New Cell ------------ -->


```python
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


### Connecting to Your Data

Workflows access and publish to data integrations that are configured on the Integrations Page.  In this demo we will connect to the demo data integration which is a Postgres database containing several standard [sample datasets](https://docs.aqueducthq.com/example-workflows/demo-data-warehouse) including some synthetic customer churn data.  Each kind of data integration may offer different functionality.  Here we are using a relational integration which support general SQL expressions.





<!-- ------------- New Cell ------------ -->


```python
warehouse = client.integration(name="aqueduct_demo")

# customers_table is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
customers_table = warehouse.sql(query="SELECT * FROM customers;")
print(type(customers_table))
```




<!-- ------------- New Cell ------------ -->


Here the `cust` table is a `TableArtifact` representing a virtual table living in the cloud.  For debugging purposes I can get the Pandas DataFrame corresponding to any `TableArtifact` by using the `get()` method:




<!-- ------------- New Cell ------------ -->


```python
# This gets the head of the underlying DataFrame. Note that you can't
# pass a DataFrame as an argument to a workflow; you must use the Aqueduct
# TableArtifact!
customers_table.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cust_id</th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0</td>
      <td>1.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


### Applying Operators to Tables

I can transform these `TableArtifacts` by applying *decorated* Python functions. Here, we can use the `log_featurize` function we defined above, and we call it like a regular Python function. Because we added the decorator above, Aqueduct will now run this function in the cloud as a part of a workflow.




<!-- ------------- New Cell ------------ -->


```python
features_table = log_featurize(customers_table)
print(type(features_table))
```




<!-- ------------- New Cell ------------ -->


As before, we can always view the value of the table by getting the corresponding data frame. 




<!-- ------------- New Cell ------------ -->


```python
features_table.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
      <th>log_company_size</th>
      <th>log_duration_months</th>
      <th>log_n_data_eng</th>
      <th>log_n_integrations</th>
      <th>log_n_rows</th>
      <th>log_n_support_tickets</th>
      <th>log_n_users</th>
      <th>log_n_workflows</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
      <td>3.401197</td>
      <td>0.693147</td>
      <td>1.098612</td>
      <td>1.791759</td>
      <td>7.604894</td>
      <td>1.386294</td>
      <td>1.098612</td>
      <td>1.609438</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
      <td>3.465736</td>
      <td>0.693147</td>
      <td>1.386294</td>
      <td>1.609438</td>
      <td>9.052399</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.386294</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0</td>
      <td>1.0</td>
      <td>1</td>
      <td>3.401197</td>
      <td>1.386294</td>
      <td>0.693147</td>
      <td>1.386294</td>
      <td>8.929170</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.609438</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
      <td>3.526361</td>
      <td>1.609438</td>
      <td>1.386294</td>
      <td>1.609438</td>
      <td>8.363342</td>
      <td>0.693147</td>
      <td>0.693147</td>
      <td>1.386294</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
      <td>3.367296</td>
      <td>0.693147</td>
      <td>1.098612</td>
      <td>1.386294</td>
      <td>7.667158</td>
      <td>0.000000</td>
      <td>0.693147</td>
      <td>1.098612</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


Next, we can apply the prediction functions we defined above, and Aqueduct will continue to build a workflow of our functions running in the cloud: 




<!-- ------------- New Cell ------------ -->


```python
linear_pred_table = predict_linear(features_table)
tree_pred_table = predict_tree(features_table)
churn_table = predict_ensemble(customers_table, linear_pred_table, tree_pred_table)
```




<!-- ------------- New Cell ------------ -->


Note that our ensemble function can take in 3 data tables. Aqueduct will make sure that all three are correctly provided to the function when it's executing.

Examining the final table we see our predictions in the `prob_churn` column.




<!-- ------------- New Cell ------------ -->


```python
churn_table.get().head()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cust_id</th>
      <th>n_workflows</th>
      <th>n_rows</th>
      <th>n_users</th>
      <th>company_size</th>
      <th>n_integrations</th>
      <th>n_support_tickets</th>
      <th>duration_months</th>
      <th>using_deep_learning</th>
      <th>n_data_eng</th>
      <th>using_dbt</th>
      <th>prob_churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>4</td>
      <td>2007</td>
      <td>2</td>
      <td>29</td>
      <td>5</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
      <td>0.081036</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3</td>
      <td>8538</td>
      <td>1</td>
      <td>31</td>
      <td>4</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
      <td>0.146564</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4</td>
      <td>7548</td>
      <td>1</td>
      <td>29</td>
      <td>3</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0</td>
      <td>1.0</td>
      <td>1</td>
      <td>0.155584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>3</td>
      <td>4286</td>
      <td>1</td>
      <td>33</td>
      <td>4</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>0</td>
      <td>3.0</td>
      <td>1</td>
      <td>0.145398</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2</td>
      <td>2136</td>
      <td>1</td>
      <td>28</td>
      <td>3</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>2.0</td>
      <td>1</td>
      <td>0.161406</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


### Checking Your Tables

Debugging or even knowing when a prediction pipeline is failing can be challenging!  Input data is constantly change, features can drift, and even the choice of model may no longer make sense as labels shift.  It is therefore critical that we check our data before publishing predictions.  This can be done using `checks`.

An Aqueduct `check` takes in a Pandas DataFrame and returns a Panda Series of booleans. The test passes if everything in the Series is true. 

Let's ensure that all our probabilities are valid probabilities between 0 and 1.




<!-- ------------- New Cell ------------ -->


```python
@aq.check(description="Ensuring valid probabilities.")
def valid_probabilities(df: pd.DataFrame):
    return (df["prob_churn"] >= 0) & (df["prob_churn"] <= 1)
```




<!-- ------------- New Cell ------------ -->


Executing `valid_probabilities` on `churn_table` will automatically attach the check to the table, but it will not run the test and until you call `get()` on the output.




<!-- ------------- New Cell ------------ -->


```python
check_result = valid_probabilities(churn_table)
```




<!-- ------------- New Cell ------------ -->


We can also create metrics and check the values of those metrics. A metric is simply a measurement applied to a whole table or a column in a table. Every artifact comes with a set of common, built-in metrics like `mean`, `min`, `max`, etc.




<!-- ------------- New Cell ------------ -->


```python
# Use Aqueduct's built-in mean metric to calculate the average value of `prob_churn`.
# Calling .get() on the metric will retrieve the current value.
avg_pred_churn_metric = churn_table.mean("prob_churn")
avg_pred_churn_metric.get()
```
**Output:**


```
0.28025028109550476
```






<!-- ------------- New Cell ------------ -->


We can also add bounds to metrics that enforce correctness constraints on the value of the metric. For example, we might expect the average churn metric to be between 0.1 and 0.3, and if `avg_pred_churn` ever goes above 0.4 we want to prevent predictions from being published.




<!-- ------------- New Cell ------------ -->


```python
# Bounds on metrics ensure that the metric stays within a valid range.
# In this case, we'd ideally like churn to be between .1 and .3, and we
# know something's gone wrong if it's above .4.
avg_pred_churn_metric.bound(lower=0.1)
avg_pred_churn_metric.bound(upper=0.3)
avg_pred_churn_metric.bound(upper=0.4, severity="error")
```
**Output:**


```
<aqueduct.check_artifact.CheckArtifact at 0x120d68970>
```






<!-- ------------- New Cell ------------ -->


### Saving Tables Artifacts to Integrations

So far we have defined a workflow to build the `churn_table` containing our churn predictions.  We now want to publish this table where others can use it.  We do this by `saving` the table to various integrations.  

First we save the table back to the data warehouse that contains the original customer data. 




<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to save the results in churn_table
# back to the demo DB we configured earlier.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published in the next cell.
warehouse.save(churn_table, table_name="pred_churn", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


## Scheduling the Workflow to Run Repeatedly

With all this hard work done, we're ready to deploy our workflow so that it runs repeatedly.  To do this, we define a workflow with a name and the artifacts that we want to compute and a cadence that we want to run the workflow.  The Aqueduct APIs will automatically capture and intermediate artifacts and code needed to produce the final artifacts.

When you call `publish_flow`, all of this will be shipped off to the cloud!




<!-- ------------- New Cell ------------ -->


```python
# This publishes all of the logic needed to create churn_table
# and avg_pred_churn_metric to Aqueduct and schedules the workflow
# to run on an hourly basis. The URL below will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
churn_flow = client.publish_flow(
    name="Demo Churn Ensemble",
    artifacts=[churn_table, avg_pred_churn_metric],
    schedule=aq.hourly(),
)
print(churn_flow.id())
```




<!-- ------------- New Cell ------------ -->


Clicking on the URL above will take you to the Aqueudct UI where you can see the workflow that we just created! On the Aqueduct UI, you'll be able to see the DAG of operators we just created, click into any of those operators, and see the data and metadata associated with each stage of the pipeline.

