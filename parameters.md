# Parameters

**A Parameter is simply an argument that is provided to a workflow at runtime.**

See this [**Parameters Tutorial**](example-workflows/parameters-tutorial.md) notebook for a full walkthrough of the API with concrete examples.

## Creating Parameters

Parameters are identified by their name, and _must_ have default values. They are fed into operators just like any other artifact, and every artifact type is supported. Parameters can be created explicitly or implicitly.

Here's an example of how we can create an explicit parameter and feed it into a prediction function:

```python
@op
def predict_churn(reviews, pd.DataFrame, country: str):
    ...

reviews = db.sql("Select * from customer_reviews")
country_param = client.create_param("country", default="Venezuela") 
churn = predict_churn(reviews, country_param)

# This will return the result of `predict_churn()` with default "Venezuela" as the country input.
churn.get()

# This will return the result of `predict_churn()` with "United Kingdom" as the country input.
churn.get(parameters={"country": "United Kingdom"})
```

To create a parameter implicitly, feed a regular python object `v` into any operator. We will implicitly convert the input into a parameter for you, with the default value `v`. The name of the new parameter is the name of its corresponding argument on the function signature.

```python
# This will create a new parameter directly, named `country`, with default value `Venezuela`.
churn = predict_churn(reviews, "Venezuela")
```

{% hint style="warning" %}
We currently disallow implicit parameter creation if an explicit parameter with the same name already exists.
{% endhint %}

To create a parameter from local data, pass in the path to data as `default` and set `use_local` to `True`. You will also need to provide the expected Artifact type of the local data.

Here's an example of how we can create a local data parameter that has the content of a csv data from a local file:

```python
from aqueduct.constants.enums import ArtifactType
local_data = client.create_param(
                    name ="table_data", 
                    default="path/to/data.csv",
                    use_local=True,
                    as_type=ArtifactType.TABLE,
                    format="csv",
                    ) 
```

Note that, we need to specify the `format` of the table if we want to create a Table Artifact Parameter.

To publish a workflow that depends on any local data parameter:

```python
client.publish_flow(
                    name = "local_data_workflow",
                    artifacts = [local_data, ...],
                    use_local = True,
                    ) 
```

## Customizing Parameters

As shown in the examples above, you can immediately materialize artifacts locally with different parameters using `.get(parameters={...})`. This allows you to play around with and observe the effects of different parameters in your local environment.

However, you can also publish parameterized flows, and trigger new runs of that flow with different parameters!

```python
churn = predict_churn(reviews, country_param)
flow = client.publish_flow("Churn Prediction", artifacts=[churn])
...

# This will trigger a new run of the already published flow "Churn Prediction",
# but with the country parameter value set to "China".
client.trigger(flow.id(), parameters={"country": "China"})
```

{% hint style="info" %}
Triggering a workflow with non-default parameters is a one-off operation. It does not change the default parameters for future runs of the workflow. That is to say, if a workflow is running on a schedule, every scheduled run will continue to use the same default parameters. You can always change the default value of a published parameter by re-running the workflow with the new default value.
{% endhint %}

{% hint style="info" %}
Types are also enforced on parameters. You cannot set a new value that is of a different type as the default value.
{% endhint %}

## Parameters in SQL Queries

SQL queries are parameterized in a special way, using this double-bracket syntax:

```python
# A parameter used in a SQL query *must* be a string type.
_ = client.create_param("country", default="Venezuela")

# The value of `country_param` is interpolated into the double-bracketed placeholder.
db = client.integration("aqueduct_demo")
output_table = db.sql("select * from locations where country_name='{{country}}'")

# This returns the result of `select * from locations where country_name='Venezuela';`
output_table.get()

# This returns the result of `select * from locations where country_name='Argentina';`
output_table.get(parameters={"country": "Argentina"})
```

{% hint style="info" %}
We perform a direct substitute of the parameter string into the double-bracketed placeholder token, so it is still up to you to construct your SQL query appropriately. For example, we do not automatically add quotes around column names for you.
{% endhint %}

We provide the following built-in SQL parameters:

* `today`: expands to a string date in the format `'%Y-%m-%d'`. Eg. `SELECT * from hotel_reviews WHERE review_date={{today}};`
