# Spark EMR


Before starting, please make sure you've added a Spark integration to Aqueduct (see [connecting-to-spark-emr.md](../adding-an-integration/connecting-to-spark-emr.md "mention")).

### Running a Workflow

The recommended way to use the Spark integration is to set the integration in the global [compute-integrations.md](compute-integrations.md "mention") and to use lazy execution mode.

```python
aq.global_config({'engine': '<spark_integration name>', 'lazy': True})
```

After setting this mode, we are ready to launch workflows on Spark!

### Writing an Operator for Spark

For operators that run on Spark, inputs that are Table types are expected to be [Spark DataFrames](https://api-docs.databricks.com/python/pyspark/latest/pyspark.sql/dataframe.html?\_ga=2.9671094.26953074.1675193222-707915355.1675193222). In the example below, both the input `df` and the output are Spark DataFrames.

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
    maintains the original values as-is. In addition to the original company_size column, log_featurize will add a log_company_size column.
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

