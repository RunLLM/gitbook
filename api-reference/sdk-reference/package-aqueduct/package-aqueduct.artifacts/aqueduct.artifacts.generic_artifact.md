# Table of Contents

* [aqueduct.artifacts.generic\_artifact](#aqueduct.artifacts.generic_artifact)
  * [GenericArtifact](#aqueduct.artifacts.generic_artifact.GenericArtifact)
    * [get](#aqueduct.artifacts.generic_artifact.GenericArtifact.get)
    * [describe](#aqueduct.artifacts.generic_artifact.GenericArtifact.describe)
    * [system\_metric](#aqueduct.artifacts.generic_artifact.GenericArtifact.system_metric)

<a id="aqueduct.artifacts.generic_artifact"></a>

# aqueduct.artifacts.generic\_artifact

<a id="aqueduct.artifacts.generic_artifact.GenericArtifact"></a>

## GenericArtifact Objects

```python
class GenericArtifact(BaseArtifact, system_metric.SystemMetricMixin)
```

This class represents a generic artifact within the flow's DAG.

Currently, a generic artifact can be any artifact other than table, numeric, bool, or parameter
generated from eager execution, or an artifact of unknown type generated from lazy execution.

<a id="aqueduct.artifacts.generic_artifact.GenericArtifact.get"></a>

#### get

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> Any
```

Materializes the artifact.

**Returns**:

  The materialized value.
  

**Raises**:

  InvalidRequestError:
  An error occurred because of an issue with the user's code or inputs.
  InternalServerError:
  An unexpected error occurred in the server.

<a id="aqueduct.artifacts.generic_artifact.GenericArtifact.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the bool artifact.

<a id="aqueduct.artifacts.generic_artifact.GenericArtifact.system_metric"></a>

#### system\_metric

```python
def system_metric(metric_name: str,
                  lazy: bool = False) -> numeric_artifact.NumericArtifact
```

Creates a system metric that represents the given system information from the previous @op that ran on the table.

**Arguments**:

  metric_name:
  Name of system metric to retrieve for the table.
  Valid metrics are:
- `runtime` - runtime of previous @op func in seconds
- `max_memory` - maximum memory usage of previous @op func in Mb
  

**Returns**:

  A numeric artifact that represents the requested system metric

