# Configuring Resource Constraints for an Operator

{% hint style="warning" %}
All this is currently only supported for operators executing on Kubernetes integrations.
{% endhint %}

The @op decorator accepts a `resources` dictionary that allows you to request a certain
amount of resources for an operator to run against. The exact amount of requested resource
is provided; no more, no less. If you ask for 4GB of memory, that is what you will get. If
the runtime does not have the capacity of allocate the requested resource available, the 
operator will fail immediately with a helpful error message.


### GPU Access
```python
import pytorch
@op(resources={"gpu_resource_name": "nvidia.com/gpu"})
def has_gpu_access():
    return pytorch.cuda.is_available() # returns true
```

Allows the operator to access GPU resource in a Kubernetes cluster. Currently we limit an operator
to access 1 GPU per invocation. If the Kubernetes cluster does not have an available GPU or if the
`gpu_resource_name` is provided incorrectly, the operator will fail immediately with a 
helpful error message.

To find the gpu_resource_name, run `kubectl describe node <name of node with GPU>` and look under
the `Allocatable` section.


### Number of CPUs

```python
@op(resources={"num_cpus": 4})
```
Configures the number of CPUs available to the operator.


### Available Memory

```python
@op(resources={"memory": "4GB"})
```

Configures the amount of memory available to the operator. This value can be a string or an integer.
Integers are in interpreted in units of MBs. Strings support both "MB" and "GB" suffixes, case-insensitive.

{% hint style="warning" %}
If you are having issues previewing such operators, make sure your execution engine is configured.

```python
import aqueduct as aq
aq.global_config({"engine": "my_kubernetes_integration"})
```
{% endhint %