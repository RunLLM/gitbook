# Compute Integrations

{% hint style="info" %}
Before starting, make sure you've added your Compute Integration to Aqueduct (see [adding-an-integration](../adding-an-integration/ "mention")).
{% endhint %}
Currently, Aqueduct supports Airflow, Kubernetes, and Lambda. First, we're going to get a connection to your compute integrations(we are using Kubernetes as an example) by calling `.integration()` on the Aqueduct Client.

```python
k8s_integration = client.integration('k8s_integration')

```

### Publishing Workflow from the Connected Engines
To publish a workflow executed by different execution engine to Aqueduct, we will add in the `config` parameter to `publish_flow` to specify which engine we intend to use:

```python
from aqueduct.config import FlowConfig
flow = client.publish_flow(name='average_acidity', 
                           artifacts=[acidity_by_group],
                           schedule=aqueduct.hourly(),
                           config=FlowConfig(engine=k8s_integration))
print(flow.id())
```

Here, the `config` parameter is set to a `FlowConfig` variable where we can specifies its `engine` parameter to the Kubernetes integration we have connected. After executing the command, the workflow will be published using the Kubernetes as the execution engine. 