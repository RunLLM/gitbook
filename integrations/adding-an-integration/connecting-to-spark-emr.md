# Connecting to Spark EMR

Connecting Aqueduct to Spark EMR requires the following:
* A standing Spark EMR cluster configured with a [Livy](http://livy.incubator.apache.org/get-started/) endpoint. We use Livy to submit applications to the Spark cluster.
* [`AWS S3` as your metadata storage ](./connecting-to-aws-s3.md)
*  The datasources that Aqueduct on Spark currently supports are the following (with more to come soon!):
    - `Snowflake`
    - `AWS S3`


On the Aqueduct UI, click on the Spark logo and enter the following information:

* A unique name for your Spark connection.
* The endpoint for the Livy URL.


You should be good to go! Let us know if you run into any issues with your Databricks integrations on our community [Slack](https://slack.aqueducthq.com) or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).
