# Databricks

{% hint style="info" %}
Before starting, please make sure you've added a Databricks integration to Aqueduct (see [connecting-to-databricks.md](../adding-an-integration/connecting-to-databricks.md "mention")).
{% endhint %}

### Running a Workflow

The recommended way to use the Databricks integration is to set the integration in the global [compute-integrations.md](compute-integrations.md "mention") and to use lazy execution mode.

```python
aq.global_config({'engine': '<databricks_integration name>', 'lazy': True})
```

We spin up an [on-demand cluster](https://docs.databricks.com/workflows/jobs/jobs.html#create-a-job) for each workflow; pre-setting lazy execution allows us to avoid unnecessary spin-up of clusters as we build our workflow. See [lazy-vs.-eager-execution.md](../../operators/lazy-vs.-eager-execution.md "mention") for more information about execution mode.

After setting this mode, we are ready to launch workflows on Databricks!

### Writing an Operator for Databricks

For operators that run on Databricks, inputs that are Table types are expected to be [Spark DataFrames](https://api-docs.databricks.com/python/pyspark/latest/pyspark.sql/dataframe.html?\_ga=2.9671094.26953074.1675193222-707915355.1675193222). In the example below, both the input `df` and the output are Spark DataFrames.

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

When this is executed by either eagerly executing the operator or by publishing a workflow, Aqueduct will use the Databricks Jobs API to launch a Job Cluster and execute the operators of the specified workflow. After execution, the cluster will be torn down.
