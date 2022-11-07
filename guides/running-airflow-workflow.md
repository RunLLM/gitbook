# Running a Workflow on Airflow

This guide will walk you through how to deploy a workflow on your existing Airflow cluster.

### Prerequisites

* The Aqueduct metadata storage must be changed to an AWS S3 bucket. See here for how to do so. The AWS credentials file for this bucket must be copied over to your Airflow cluster.
* Modify your Airflow cluster's config to use `Basic Authentication`. See the instructions [here](https://airflow.apache.org/docs/apache-airflow/stable/security/api.html#basic-authentication) for how to do so. For now, Aqueduct only supports basic authentication using a username and password. We are working on adding support for other authentication methods.

### Adding An Airflow Integration

Once the Airflow cluster has been properly configured and is running, you will need to connect it to Aqueduct via an integration.

The S3 credentials file must already be on the Airflow cluster. _Aqueduct cannot do this for you._ The credentials must have access to the same S3 bucket used as the Aqueduct metadata store.

### Deploying a Workflow

The Python SDK can be used as is for constructing workflows. All you have to do is set the workflow engine to the appropriate Airflow integration when publishing:

```python
from aqueduct import FlowConfig

flow = client.publish_flow(
    name = "<WORKFLOW_NAME>",
    artifacts = "<ARTIFACTS>",
    engine="<YOUR_AIRFLOW_INTEGRATION_NAME>",
)
```

At this point the SDK should return an Airflow DAG file and print out the location of the downloaded file. **You must copy this file to your Airflow cluster.** The file should be placed in your DAGs folder. At this point the workflow has been registered with Airflow, and it will execute at the schedule you have specified.

### Viewing Workflow Results

Workflow results can be viewed from the UI the same way results for workflows running natively on Aqueduct can be viewed. Aqueduct continuously fetches the latest DAG runs from Airflow to ensure that you see the latest workflow results.

### Updating a Workflow

The Python SDK can be used to update an Airflow workflow just like a regular workflow. The only difference is that since the DAG structure may have changed, the SDK will return an updated Airflow DAG file. This updated file should be copied over to your Airflow cluster and replace the original DAG file. **Until this file is copied over, subsequent DAG runs on Airflow will not be synced over to Aqueduct.**
