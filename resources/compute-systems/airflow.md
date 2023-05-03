---
description: Run your ML pipelines on Airflow.
---

# Airflow

## Connecting to Airflow

{% hint style="info" %}
Before you can connect Aqueduct to Airflow, you must be using AWS S3 as your artifact store. See [configuring-aqueduct.md](../../installation-and-configuration/configuring-aqueduct.md "mention") for more on how to update your artifact storage layer.

Once you've set an S3 bucket as your artifact store, you must ensure that the AWS credentials files on the machine running your Airflow server has access to this bucket.&#x20;
{% endhint %}

To connect Aqueduct to Airflow, you will need the following information:&#x20;

* **Name**: A unique name for your connection. This is totally up to you!
* **Host**: The hostname or IP address of your Airflow server.
* **Username**: The username of the account via which you're connecting to Airflow.
* **Password**: The password corresponding to the above username.
* **S3 Credentials Path**: The path on the Airflow server in which your AWS credentials are stored. _These credentials must have access to the S3 bucket you use for artifact storage._
* **S3 Credentials Profile (optional)**: If your AWS credentials file has multiple profiles, this specifies which profile should be used. If none is specified, the `default` profile will be used.

{% hint style="info" %}
Aqueduct currently only supports Basic Authentication mode for Airflow connections. If you need to use a different authentication mode, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!
{% endhint %}

### UI

To connect Aqueduct to Airflow, navigate to the Aqueduct resources page and click on the Airflow icon. Enter the information above.

### SDK

Aqueduct does not currently support connecting an Airflow resource via the SDK. If you need this capability, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!

## Using Airflow

To define a workflow that runs on Airflow, you must have a workflow spec stored on the Airflow server. Aqueduct automatically generates this workflow spec for you.

Your Python functions can be defined in the regular Aqueduct operator syntax. When you call `publish_flow`, you can specify the name of your Airflow resource in the `engine` field:&#x20;

```python
flow = client.publish_flow(
    name="Airflow Example",
    artifacts=output_artifact,
    engine="my_airflow_cluster",
)
```

When you call `publish_flow`, you will see a pointer to a downloaded file with the Airflow spec for your workflow. **You will need to copy that spec file to your Airflow server in order for Airflow to execute it correctly**.

{% hint style="warning" %}
For a workflow that you're updating, please ensure you copy the updated workflow spec to your Airflow server ASAP. If there is a delay, workflow runs with older specs might be triggered.
{% endhint %}

Once your workflow is registered, you will be able to see the workflow spec on the Aqueduct UI. Because of the way Airflow's API is designed, Aqueduct cannot eagerly keep workflow metadata in sync. Aqueduct will sync workflow metadata when you load the details for a workflow, but you might see a slight delay in the metadata.

{% hint style="info" %}
You cannot use Airflow as the engine in your `global_config`. Because of the nature of Airflow's execution engine, it does not support interactive execution mechanisms. See [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention") for more details on setting your `global_config`.
{% endhint %}
