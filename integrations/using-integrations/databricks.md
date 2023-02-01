# Databricks

{% hint style="info" %}
Before starting, please make sure you've added a Databricsk integration to Aqueduct (see [adding-an-integration](../adding-an-integration/connecting-to-databricks.md "mention")).&#x20;
{% endhint %}

### Running a Workflow

The recommended way to use the Databricks integration is to [set the global config](./compute-integrations.md "mention") to use the Databricks integration and lazy execution mode. 

```python
aq.global_config({'engine': '<databricks_integration name>', 'lazy': True})
```

We spin up an on-demand cluster for each workflow; pre-setting lazy execution allows us to avoid unnecessary spin-up of clusters as we build our workflow. See [lazy-vs.-eager-execution.md](../../operators/lazy-vs.-eager-execution.md "mention") for more information about execution mode.

After setting this mode, we are ready to launch workflows on Databricks!

### Writing an Operator for Databricks

For operators that run on Databricks, inputs that are Table types are expected to be Spark DataFrames. In the example below, both the input `df` and the output are Spark DataFrames.

```python
@op
def log_featurize(df):
    import numpy as np
    import pyspark.sql.functions as F
    from pyspark.sql.types import FloatType
    """
    log_featurize takes in customer data from the Aqueduct customers table
    and log normalizes the numerical columns using the numpy.log function.
    It skips the cust_id, using_deep_learning, and using_dbt columns because
    these are not numerical columns that require regularization.

    log_featurize adds all the log-normalized values into new columns, and
    maintains the original values as-is. In addition to the original company_size
    column, log_featurize will add a log_company_size column.
    """
    

    def udf_np_log(a):
        # actual function
        return float(np.log(a+1.0))

    np_log = F.udf(udf_np_log, FloatType())
    features = df.alias('features')
    skip_cols = ["cust_id", "using_deep_learning", "using_dbt"]

    for col in [c for c in features.schema.names if c not in skip_cols]:
        features = features.withColumn("LOG_" + col, np_log(features[col]))

    return features.drop("cust_id")
```

