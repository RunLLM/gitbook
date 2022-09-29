# Running a Workflow on a Kubernetes Cluster

This guide will walk you through how to deploy a workflow on your existing K8s cluster.

### Prerequisites

* The Aqueduct metadata storage must be changed to an `AWS S3` or `GCS` bucket.
* The `kubeconfig` file that allows access to the k8s cluster must be on the same machine running the Aqueduct server.

### Adding A K8s Integration

Once the Airflow cluster has been properly configured and is running, you will need to [connect it to Aqueduct via an integration](../integrations/adding-an-integration/connecting-to-k8s-cluster.md).


### Deploying a Workflow

The Python SDK can be used as is for constructing workflows. The only difference comes when issuing an API call to publish the workflow.

First, you need fetch the Airflow integration.

```python
k8s_integration = client.integration("<YOUR_INTEGRATION_NAME>")
```

Next, you will set the workflow engine to the above Airflow integration when publishing the workflow.

```python
from aqueduct import FlowConfig

flow = client.publish_flow(
    name = WORKFLOW_NAME,
    artifacts = ARTIFACTS,
    config = FlowConfig(engine=k8s_integration),
)
```

### Viewing Workflow Results

Workflow results can be viewed from the UI the same way results for workflows running natively on Aqueduct can be viewed. Aqueduct continuously fetches the latest DAG runs from Airflow to ensure that you see the latest workflow results.

You can also view progress via kubectl commands. All Aqueduct operators and resources will be under the aqueduct namespace. For example, you can check the pods created by Aqueudct by running `kubectl get pods -n aqueduct`. 

### Updating a Workflow

The Python SDK can be used to update an Airflow workflow just like a regular workflow.
