# aqueduct.metric\_artifact

## Table of Contents

* [aqueduct.metric\_artifact](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact)
  * [MetricArtifact](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact.MetricArtifact)
    * [get](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact.MetricArtifact.get)
    * [list\_preset\_checks](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact.MetricArtifact.list\_preset\_checks)
    * [bound](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact.MetricArtifact.bound)
    * [describe](aqueduct.metric\_artifact.md#aqueduct.metric\_artifact.MetricArtifact.describe)

## aqueduct.metric\_artifact

### MetricArtifact Objects

```python
class MetricArtifact(Artifact)
```

This class represents a computed metric within the flow's DAG.

Any `@metric`-annotated python function that returns a float will return this class when that function is called.

**Examples**:

> > > @metric def compute\_metric(df): return metric metric\_artifact = compute\_metric(input\_artifact)

The contents of this artifact can be manifested locally.

> > > val = metric\_artifact.get()

**get**

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> float
```

Materializes a MetricArtifact into its immediate float value.

**Returns**:

The evaluated metric as a float.

**Raises**:

InvalidRequestError: An error occurred because of an issue with the user's code or inputs. InternalServerError: An unexpected error occurred within the Aqueduct cluster.

**list\_preset\_checks**

```python
def list_preset_checks() -> List[str]
```

Returns a list of all preset checks available on the metric artifact. These preset checks can be set via the bound() method on a artifact.

**Returns**:

A list of available preset checks on a metric

**bound**

```python
def bound(upper: Optional[float] = None,
          lower: Optional[float] = None,
          equal: Optional[float] = None,
          notequal: Optional[float] = None,
          severity: CheckSeverity = CheckSeverity.WARNING) -> CheckArtifact
```

Computes a bounds check on this metric with the specified boundary condition.

Only one of `upper` and `lower` can be set.

> > > metric\_artifact.bound(upper = 0.9, severity = CheckSeverity.Error)

If the metric ever exceeds 0.9, the flow will fail.

**Arguments**:

upper: Sets an upper bound on the value of the metric. lower: Sets a lower bound on the value of the metric. severity: If specified, will set the severity of this check as specified. Defaults to CheckSeverity.WARNING

**Returns**:

A check artifact bound to this metric.

**describe**

```python
def describe() -> None
```

Prints out a human-readable description of the metric artifact.
