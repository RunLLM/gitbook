# Creating and Using Parameters

See this [**Workflow Parameters Tutorial**](../example-workflows/using-parameters.md) notebook for a full walkthrough of the API with concrete examples.

### Creating Parameters

Parameters are identified by their name, and *must* have default values. They are fed into operators just like any other artifact, and 
every artifact type is supported. Parameters can be created explicitly or implicitly.

To create a parameter explicitly, use:

```python
param_artifact = client.create_param("Param Name", default=124) 
```

They can also be referenced within SQL queries using the double-bracket syntax!
```python

# A parameter used in a SQL query *must* be a string type.
client.create_param("country", default="'Argentina'")

# The value of `country_param` is interpolated into the double-bracketed placeholder.
db = client.integration("aqueduct_demo")
output = db.sql("select * from locations where country_name={{country}}")

# This returns the result of `select * from locations where country_name='Argentina';`
output.get()
```

{% hint style="info" %}
We perform a direct substitute of the parameter string into the double-bracketed placeholder token,
so it is still up to you to construct your SQL query appropriately. For example, we do not automatically
add quotes around column names for you.
{% endhint %}

We provide the following built-in SQL parameters:
- `today`: expands to a string date in the format `'%Y-%m-%d'`. Eg. `SELECT * from hotel_reviews WHERE review_date={{today}};`

To create a parameter implicitly, feed a regular python object `v` into any operator. We will implicitly convert the input into a parameter for you, with the default value `v`. 
The name of the new parameter is the name of its corresponding argument on the function signature.

```python
@op
def predict_churn(reviews, country):
    ...

reviews = db.sql("Select * from customer_reviews")

# This will create a new parameter named `country`, with default value `Venezuela`.
churn = predict_churn(reviews, "Venezuela")
```

{% hint style="warning" %}
We currently disallow implicit parameter creation if an explicit parameter with the same name already exists.
{% endhint %}


### Customizing Parameters
For any artifact, you can materialize it with custom parameter values fed into `.get()`, keyed by parameter name. For example:

```python
# Runs the same SQL query as the example above, but filters the results to a different country.
df = output.get(parameters={"country": "'United Kingdom'"})
```

{% hint style="info" %}
Types are also enforced for parameters. You cannot set a new value that is of a different type as the default value.
{% endhint %}

Similarly, for any flow that has been published with parameters, you can always trigger a new flow run with custom parameters:
```python
flow = client.flow("ce77bcf3-1fa7-4d29-b733-c3b44f952dfe")
client.trigger(flow.id(), parameters={"country": "'China'"})
```

{% hint style="info" %}
Triggering a workflow with non-default parameters is a one-off operation. It does not change the default parameters for future runs of the workflow.
That is to say, if a workflow is running on a schedule, every scheduled run will continue to use the same default parameters. 
You can always change the default value  of a published parameter by re-running the workflow with the new default value.
{% endhint %}
