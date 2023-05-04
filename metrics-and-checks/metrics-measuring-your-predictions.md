# Metrics: Measuring your Predictions

**Metrics** are numerical measurements and summaries of a dataset in Aqueduct. Writing a metric is very similar to writing an [Operator](../operators.md) -- the only difference is that rather than returning a data Artifact, a Metric function returns a floating point number.

Here's how you can create a Metric:

```python
from aqueduct import Client, metric
import pandas as pd

client = Client() 

db = client.resource('aqueduct_demo')
customers = db.sql('SELECT * FROM customers;')

@metric
def avg_customer_workflows(customers: pd.DataFrame) -> float:
    return customers['n_workflows'].mean() 
    
avg_num_wflows = avg_customer_workflows(customers)
avg_num_wflows.get()
```

Here, we created a function called `avg_customer_workflows` that returns the mean of the `num_workflows` column in the customers table. When we apply it to the `customers` table, we get a Metric in response -- as with all computation in Aqeuduct, the results are lazily computed when you call `.get()`.

Metrics can also have bounds on them, which enforce minimum or maximum values of the metric. If a metric exceeds one of its bounds, Aqueduct will automatically detect this and raise an error for you. Bounds are shorthand for [Checks](checks-ensuring-correctness.md), and we will cover creating bounds in that documentation.

### System Metrics

In addition to metrics measuring the data that your workflows generate, Aqueduct also allows you to measure lower-level metrics about the actual generation of your data, which we call "System Metrics." System metrics are attached to the result of a function, and they will measure the resource consumption of the function that was used to generate this data.

Aqueduct currently supports two system metrics:

* `runtime`: Measures the runtime for the operator that generated this table in seconds.
* `max_memory`: Measures the maximum memmmory that the function which generated this table used in MB.

Here's an example that creates a system metric:

```python
from aqueduct import op, Client
import pandas as pd

client = Client() 
db = client.resource("aqueduct_demo")

@op
def predict_churn(reviews: pd.DataFrame, country: str):
    ...

reviews = db.sql("Select * from customers")
country_param = client.create_param("country", default="Venezuela") 
churn = predict_churn(reviews, country_param)

# This measures the amount of time `predict_churn` took to compute `churn`.
runtime = churn.system_metric('runtime')
```

