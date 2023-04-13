# Parameters

**A Parameter is simply an argument that is provided to a workflow at runtime.**

See this [**Parameters Tutorial**](example-workflows/parameters-tutorial.md) notebook for a full walkthrough of the API with concrete examples.

## Creating Parameters

Parameters are identified by their name, and _must_ have default values. They are fed into operators just like any other artifact, and every artifact type is supported. Parameters can be created explicitly or implicitly.

Here's an example of how we can create an explicit parameter and feed it into a prediction function:

```python
from aqueduct import op, Client
import pandas as pd

client = Client() 
db = client.integration("aqueduct_demo")

@op
def predict_churn(reviews: pd.DataFrame, country: str):
    ...

reviews = db.sql("Select * from customers")
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

## Customizing Parameters

As shown in the examples above, you can immediately materialize artifacts locally with different parameters using `.get(parameters={...})`. This allows you to play around with and observe the effects of different parameters in your local environment.

However, you can also publish parameterized flows, and trigger new runs of that flow with different parameters!

```python
churn = predict_churn(reviews, country_param)
flow = client.publish_flow("Churn Prediction", artifacts=[churn])

...

import time
# Sleep for 10 seconds to wait for flow to finish.
time.sleep(10)

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

SQL queries are parameterized using the Postgres-inspired $1, $2, etc. placeholder syntax. The number after the dollar sign indicates which parameter to replace the placeholder with. This allows a parameter to be reused multiple times within the same query and is similar to how traditional RDBMS interfaces work.

```python
# A parameter used in a SQL query *must* be a string type.
table_name_param = client.create_param("table_name", default="hotel_reviews")

# TThe value of `table_name_param` is substituted into $1.
db = client.integration("aqueduct_demo")
table_artifact = db.sql(query="select * from $1", parameters=[table_name_param])

# This returns the result of `select * from hotel_reviews;`
table_artifact.get()

# This returns the result of `select * from tips;`
table_artifact.get(parameters={"table_name": "tips"})
```

{% hint style="info" %}
We perform a direct substitute of the parameter string into the $-placeholder token, so it is still up to you to construct your SQL query appropriately. For example, we do not automatically add quotes around column names for you.
{% endhint %}

We also provide the following built-in SQL parameters:

* `today`: expands to a string date in the format `'%Y-%m-%d'`. Eg. `SELECT * from hotel_reviews WHERE review_date={{today}};`

In order to use the built-in parameters, you must use the `{{ }}` syntax. For example:

```python
reviews_before_today = db.sql("select * from hotel_reviews where review_date < {{ today }}")
reviews_before_today.get()
```
