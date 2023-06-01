# Creating Parameters

Parameters are identified by their name, and _must_ have a default value. They are fed into operators just like any other artifact, and every artifact type is supported. Parameters can be created either implicitly or explicitly.
It is easier to implicitly create parameters, but an explicit parameter can be used by multiple operators, as well as use data from your local filesystem.

Explicitly created parameters 

### Creating a Parameter Implicitly

If you feed any regular python object `v` into any operator, we will implicitly convert the input into a parameter for you, with the default value `v`.
The name of the new parameter is the name of its corresponding argument on the function signature.

```python
from aqueduct import op, Client
import pandas as pd

client = Client() 
db = client.resource("aqueduct_demo")
reviews = db.sql("Select * from customers")

@op
def predict_churn(reviews: pd.DataFrame, country: str):
    ...

# This creates a new string parameter named `country`, with default value `Venezuela`.
churn = predict_churn(reviews, "Venezuela")

# This returns the result of `predict_churn()` with default "Venezuela" as the country input.
churn.get()

# This returns the result of `predict_churn()` with "United Kingdom" as the country input.
churn.get(parameters={"country": "United Kingdom"})
```

{% hint style="warning" %}
We currently disallow implicit parameter creation if an explicit parameter with the same name already exists.
{% endhint %}

### Creating a Parameter Explicitly

A parameter can be defined explicitly with `client.create_param()`, and the resulting variable can be used just like any other artifact.
That means it can be fed into multiple operators if needed. If the parameter value is customized at runtime
the input to each of those operators will change accordingly.

```python
from aqueduct import op, Client
import pandas as pd

client = Client() 
db = client.resource("aqueduct_demo")
reviews = db.sql("Select * from customers")

@op
def predict_churn(reviews: pd.DataFrame, country: str):
    ...

country_param = client.create_param("country", default="Venezuela") 
churn = predict_churn(reviews, country_param)

# This will return the result of `predict_churn()` with default "Venezuela" as the country input.
churn.get()

# This will return the result of `predict_churn()` with "United Kingdom" as the country input.
churn.get(parameters={"country": "United Kingdom"})
```

### Creating a Parameter Explicitly using Local Data

The create a parameter from local data on your filesystem, pass in the path as `default` and set `use_local` to `True`. 
You will also need to provide the expected Artifact type of the local data. The provided path is relative to the current
working directory.

Here's an example of how we can create a local data parameter that has the content of a local csv file:

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
