# Connecting to Databricks

Connecting Aqueduct to Databricks requires the following:
* The machine running Aqueduct has `pyspark` installed.
* `AWS S3` as your metadata storage 
* A Databricks Workspace/User that has permissionsn to exeucte Databricks Jobs
* ARN for a IAM role with S3 permissions.
*  The datasources that Aqueduct on Spark currently supports are the following (with more to come soon!):
    - `Snowflake`
    - `AWS S3`

For documentation on how to create a IAM role for Databricks, [refer to the following](https://docs.databricks.com/aws/iam/instance-profile-tutorial.html).

On the Aqueduct UI, click on the Databricks logo and enter the following information:

* A unique name for your Databricks connection.
* The Databricks Workspace URL.
* API key for access to Databricks.
* The ARN for the IAM role that grants Databricks access to S3.



You should be good to go! Let us know if you run into any issues with your Databricsk integrations on our community [Slack](https://slack.aqueducthq.com) or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).
