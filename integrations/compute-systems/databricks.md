---
description: Run your ML pipelines on Databricks.
---

# Databricks

## Connecting to Databricks

{% hint style="info" %}
Please ensure the following before proceeding:

1. Aqueduct currently only supports Databricks clusters on AWS. If you need access to Databricks on Azure or GCP, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!&#x20;
2. Before you can connect Aqueduct to Databricks, you must be using AWS S3 as your artifact store. See [configuring-aqueduct.md](../../installation-and-configuration/configuring-aqueduct.md "mention") for more on how to update your artifact storage layer.
3. Aqueduct's Databricks integration currently works with data living in Snowflake and AWS S3. If you need access to other [data-systems](../data-systems/ "mention"), please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!
{% endhint %}

Before connecting Aqueduct to Databricks, please ensure that the machine running the Aqueduct server has `pyspark` installed. See the PySpark documentation for [installation instructions](https://spark.apache.org/docs/latest/api/python/getting\_started/install.html).

To connect Aqueduct to Databricks, you will need the following information:&#x20;

* **Name**: A unique name for your connection. This is totally up to you!
* **Workspace URL**: The Databricks-provided unique URL for your workspace.&#x20;
* **Access Token**: The Databricks-generated API key that you use to authenticate with your Databricks workspace.
* **S3 Instance Role ARN**: ARN for a IAM role with S3 permissions. See here for documentation on [how to create a IAM role for Databricks](https://docs.databricks.com/aws/iam/instance-profile-tutorial.html).

### UI

To connect Aqueduct to Databricks, navigate to the Aqueduct integrations page and click on the Databricks icon. Enter the information above.

### SDK

Aqueduct does not currently support connecting an Databricks integration via the SDK. If you need this capability, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!

## Using Databricks

The recommended way to use the Databricks integration is to set the integration in the `global_config` (see [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention")) and to use lazy execution mode:&#x20;

```python
aq.global_config({'engine': '<my_databricks_integration>', 'lazy': True})
```

For each invocation, we spin up an [on-demand Databricks cluster](https://docs.databricks.com/workflows/jobs/jobs.html#create-a-job); setting lazy execution allows us to avoid unnecessary spin-up of clusters as we build our workflow.

{% hint style="info" %}
Aqueduct's Databricks integration currently only supports [Spark DataFrames](https://spark.apache.org/docs/latest/sql-programming-guide.html) and does not support Pandas DataFrames. Please see the Spark documentation linked above for more details on how to use Spark DataFrames.
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

When this is executed by either eagerly executing the operator or by publishing a workflow, Aqueduct will use the [Databricks Jobs API](https://docs.databricks.com/workflows/jobs/jobs.html) to launch a Job Cluster and execute the specified code. After execution, the cluster will be torn down.

{% hint style="info" %}
The nature of the Databricks Jobs API requires Aqueduct to execute a full workflow on the Jobs Cluster once it's created. That means that Aqueduct currently only supports running a whole workflow — rather than a single operator — on Databricks. As such, you cannot publish a workflow that executes only a subset of its operators on Databricks. &#x20;
{% endhint %}
