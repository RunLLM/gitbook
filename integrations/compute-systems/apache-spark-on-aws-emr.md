# Apache Spark on AWS EMR

## Connecting to Apache Spark&#x20;

{% hint style="info" %}
Before proceeding, please ensure the following:

1. Aqueduct relies on [Apache Livy](https://livy.incubator.apache.org/get-started/) to submit workloads to your Spark cluster. Please ensure you have a Livy endpoint running.&#x20;
2. Before you can connect Aqueduct to Spark, you must be using AWS S3 as your artifact store. See [configuring-aqueduct.md](../../installation-and-configuration/configuring-aqueduct.md "mention")
3. Aqueduct's Spark integration currently supports data living in Snowflake and AWS S3. If you need access to other [data-systems](../data-systems/ "mention"), please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!&#x20;
4. Docker is installed ([instructions](https://docs.docker.com/engine/install/)). Aqueduct uses Docker to package your python requirements to ship to Spark.
{% endhint %}

To connect Aqueduct to your Spark cluster, you will need the following information:&#x20;

* **Name**: A unique name for your connection. This is totally up to you!
* **Livy Server URL**: The URL of the [Apache Livy](https://livy.incubator.apache.org/get-started/) server connected to your Spark cluster.

### UI

To connect Aqueduct to Spark, navigate to the Aqueduct integrations page and click on the Spark icon. Enter the information above.

### SDK

Aqueduct does not currently support connecting an Spark integration via the SDK. If you need this capability, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!

## Using Apache Spark

The recommended way to use the Spark integration is to set the integration in the `global_config` (see [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention")) and to use lazy execution mode:&#x20;

```python
aq.global_config({'engine': 'my_spark_integration', 'lazy': True})
```

{% hint style="info" %}
Aqueduct's Spark integration currently only supports [Spark DataFrames](https://spark.apache.org/docs/latest/sql-programming-guide.html) and does not support Pandas DataFrames. Please see the Spark documentation linked above for more details on how to use Spark DataFrames.
{% endhint %}

Once your execution engine is set, you can write a Python function that accepts and returns a Spark DataFrame. For example:&#x20;

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

When this is executed by either eagerly executing the operator or by publishing a workflow, Aqueduct will use Livy endpoint to submit your function to be executed on your Spark cluster.&#x20;

{% hint style="info" %}
The nature of Aqueduct's Spark integration requires us to execute a full workflow on the Jobs Cluster once it's created. That means that Aqueduct currently only supports running a whole workflow — rather than a single operator — on Spark. As such, you cannot publish a workflow that executes only a subset of its operators on Spark. &#x20;
{% endhint %}
