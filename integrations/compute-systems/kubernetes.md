---
description: Run your ML pipelines on Kubernetes.
---

# Kubernetes

{% embed url="https://www.youtube.com/watch?v=2SUGEIfN-7w" %}

If you don't already have a Kubernetes cluster running, please see:&#x20;

* Aqueduct's support for [on-demand-aws-eks-clusters.md](on-demand-resources/on-demand-aws-eks-clusters.md "mention")
* Creating a cluster on AWS' [Elastic Kubernetes Service](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
* Creating a cluster on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster)

If your operator requires large Python packages such as `torch` and `transformers`, we recommend
allocating at least 32 GB of disk space to your Kubernetes node.

## Connecting to Kubernetes

{% hint style="info" %}
Before you can connect Aqueduct to Kubernetes, you must be using AWS S3 or Google Cloud Storage as your artifact store. See [configuring-aqueduct.md](../../installation-and-configuration/configuring-aqueduct.md "mention") for more on how to update your artifact storage layer.
{% endhint %}

To connect Aqueduct to Kubernetes, you will need the following information:&#x20;

* **Name**: A unique name for your connection. This is totally up to you!
* **Kubernetes Config Path**: The path on your Aqueduct server to the config file for the Kubernetes cluster you are connecting to. Note that this file must be on the same machine as the Aqueduct server. By default, your `kubeconfig` file will be `$HOME/.kube/config`.
* **Cluster Name**: The name of the Kubernetes cluster to which you're connecting.&#x20;

{% hint style="info" %}
Note that Aqueduct executors require roughly 4 vCPUs and 8GB of RAM. If your Kubernetes cluster has smaller machines than this, you may see significant performance degradation.
{% endhint %}

### UI

To connect Aqueduct to Kubernetes, navigate to the Aqueduct integrations page and click on the Kubernetes icon. Enter the information above.

### SDK

Aqueduct does not currently support connecting an Kubernetes integration via the SDK. If you need this capability, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!

## Using Kubernetes

You can create workflows that use Kubernetes in one of two ways.

**First**, you can set the Aqueduct `global_config` to use your Kubernetes integration:

```python
import aqueduct as aq
aq.global_config({ "engine": "my_k8s_integration" })
```

As discussed in [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention"), this will execute all code — both interactive function calls and workflows published from this environment — using `my_k8s_integration`.

**Second**, you can set the `engine` field when calling `publish_flow`:

```python
flow = client.publish_flow(
    name="Kubernetes Example",
    artifacts=output_artifact,
    engine="my_k8s_integration",
)
```

In both cases, each function invocation will cause Aqueduct to create a containerized Python environment with the appropriate library dependencies, and trigger a pod execution on Kubernetes.&#x20;

### Resource Requests

Aqueduct's Kubernetes integration allows you to declaratively request resources. You can specify the resources that are made available to a function by using the `resources` argument to the `@op` decorator:&#x20;

```python
@op(
    engine="my_k8s_integration",
    resources={
        'num_cpus': 4,
        'memory': '16GB',
        'gpu_resource_name': 'nvidia.com/gpu',
        'cuda_version': '11.8.0'
    },
)
def my_fn():
    return True
```

{% hint style="info" %}
Aqueduct currently will only trigger the execution of your function if the appropriate resources are available. Otherwise, the request will stay in a `Pending` execution state until the appropriate resources are made available. Please ensure that you have the right resources available before triggering a Kubernetes request. We're working on improving the experience.
{% endhint %}

The `resources` map supports 4 keys:

* `num_cpus`: specifies the number vCPUs that will be made available to your function - the default value is 2.
* `memory`: specifies how much RAM will be made available to your function - the default value is 4GB.
* `gpu_resource_name`: specifies the name of the custom resource type that signifies that your Kubernetes cluster uses to signify GPU resources.
* `cuda_version`: the specific version of CUDA that should be installed in your container; this is only valid if you have specified `gpu_resource_name` — the default value is `11.4.1`.

{% hint style="info" %}
Note that Aqueduct currently only supports Nvidia GPUs. Containers will automatically have CUDA 11.4.1 installed unless otherwise specified.
{% endhint %}
