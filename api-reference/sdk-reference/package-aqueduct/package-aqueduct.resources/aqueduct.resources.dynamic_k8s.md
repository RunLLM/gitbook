# Table of Contents

* [aqueduct.resources.dynamic\_k8s](#aqueduct.resources.dynamic_k8s)
  * [DynamicK8sResource](#aqueduct.resources.dynamic_k8s.DynamicK8sResource)
    * [status](#aqueduct.resources.dynamic_k8s.DynamicK8sResource.status)
    * [create](#aqueduct.resources.dynamic_k8s.DynamicK8sResource.create)
    * [update](#aqueduct.resources.dynamic_k8s.DynamicK8sResource.update)
    * [delete](#aqueduct.resources.dynamic_k8s.DynamicK8sResource.delete)
    * [describe](#aqueduct.resources.dynamic_k8s.DynamicK8sResource.describe)

<a id="aqueduct.resources.dynamic_k8s"></a>

# aqueduct.resources.dynamic\_k8s

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource"></a>

## DynamicK8sResource Objects

```python
class DynamicK8sResource(BaseResource)
```

Class for Dynamic K8s resource.

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource.status"></a>

#### status

```python
@validate_is_connected()
def status() -> str
```

Get the current status of the dynamic Kubernetes cluster.

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource.create"></a>

#### create

```python
@validate_is_connected()
def create(
    config_delta: Union[Dict[str, Union[int, str]], DynamicK8sConfig] = {}
) -> None
```

Creates the dynamic Kubernetes cluster, if it is not currently running.

**Arguments**:

  config_delta (optional):
  This field contains new config values to be used in creating the cluster.
  These new values will overwrite existing ones from that point on. Any config values
  that are identical to the current ones do not need to be included in config_delta.
  

**Raises**:

  InvalidResourceException:
  An error occurred when the dynamic engine doesn't exist.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource.update"></a>

#### update

```python
@validate_is_connected()
def update(
        config_delta: Union[Dict[str, Union[int, str]],
                            DynamicK8sConfig]) -> None
```

Update the dynamic Kubernetes cluster. This can only be done when the cluster is in
Active status.

**Arguments**:

  config_delta:
  This field contains new config values to be used in creating the cluster.
  These new values will overwrite existing ones from that point on. Any config values
  that are identical to the current ones do not need to be included in config_delta.
  

**Raises**:

  InvalidResourceException:
  An error occurred when the dynamic engine doesn't exist.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource.delete"></a>

#### delete

```python
@validate_is_connected()
def delete(force: bool = False) -> None
```

Deletes the dynamic Kubernetes cluster if it is running, ignoring the keepalive period.

**Arguments**:

  force:
  By default, if there are any pods in the "Running" or "ContainerCreating" status,
  the deletion process will fail. However, if the flag is set to "True", this check
  will be skipped, allowing the cluster to be deleted despite the presence of such pods.
  

**Raises**:

  InvalidResourceException:
  An error occurred when the dynamic engine doesn't exist.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.resources.dynamic_k8s.DynamicK8sResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the K8s resource.

