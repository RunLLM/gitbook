# Running A Workflow On Airflow
This guide will walk you through how to deploy a workflow on your existing Airflow cluster.

## Prerequisites
- The Aqueduct metadata storage must be changed to an AWS S3 bucket. See [here]() for how to do so. The AWS credentials file for this
bucket must be copied over to your Airflow cluster.
- Modify your Airflow cluster's config to use `Basic Authentication`. See the instructions [here](https://airflow.apache.org/docs/apache-airflow/stable/security/api.html#basic-authentication) for how to do so. For now, Aqueduct only supports basic authentication
using a username and password. We are working on adding support for other authentication methods.
- Check that Airflow cluster is accessible from the machine running Aqueduct.

## Adding An Airflow Integration
Once the Airflow cluster has been properly configured and is running, you will need to connect it to Aqueduct via an integration.
The integration dialog requires the following fields:
- Name
- Host
- Username
- Password
- S3 Credentials Path: This is the path to the AWS credentials file on the Airflow server. These credentials must have access to the
S3 bucket used for Aqueduct metadata storage.
- S3 Credentials Profile: The AWS profile to use for the credentials above.

![](<../.gitbook/assets/connect_airflow.png>)

## Deploying a Workflow
The Python SDK can be used as is for constructing workflows. The only difference comes when issuing an API call to publish the workflow.

First, you need fetch the Airflow integration.
```python
airflow_integration = client.integration("<YOUR_INTEGRATION_NAME>")
```

Next, you will set the workflow engine to the above Airflow integration when publishing the workflow.
```python
from aqueduct import FlowConfig

flow = client.publish_flow(
    name = WORKFLOW_NAME, 
    artifacts = ARTIFACTS,
    config = FlowConfig(engine=airflow_integration),
)
```

At this point the SDK should return an Airflow DAG file and print out the location of the downloaded file. You must copy this file
to your Airflow cluster. The file should be placed in your DAGs folder. At this point the workflow has been registered with Airflow,
and it will execute at the schedule you have specified.

## Viewing Workflow Results
Workflow results can be viewed from the UI the same way results for workflows running natively on Aqueduct can be viewed. Aqueduct continuously fetches the latest DAG runs from Airflow to ensure that you see the latest workflow results.

## Updating a Workflow
The Python SDK can be used to update an Airflow workflow just like a regular workflow. The only difference is that since the DAG structure
may have changed, the SDK will return an updated Airflow DAG file. This updated file should be copied over to your Airflow cluster and
replace the original DAG file. Until this file is copied over, subsequent DAG runs on Airflow will not be synced over to Aqueduct.