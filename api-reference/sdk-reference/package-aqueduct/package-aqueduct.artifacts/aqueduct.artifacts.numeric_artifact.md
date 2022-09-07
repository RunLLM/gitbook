# Table of Contents

* [aqueduct.artifacts.numeric\_artifact](#aqueduct.artifacts.numeric_artifact)
  * [NumericArtifact](#aqueduct.artifacts.numeric_artifact.NumericArtifact)
    * [get](#aqueduct.artifacts.numeric_artifact.NumericArtifact.get)
    * [list\_preset\_checks](#aqueduct.artifacts.numeric_artifact.NumericArtifact.list_preset_checks)
    * [bound](#aqueduct.artifacts.numeric_artifact.NumericArtifact.bound)
    * [describe](#aqueduct.artifacts.numeric_artifact.NumericArtifact.describe)

<a id="aqueduct.artifacts.numeric_artifact"></a>

# aqueduct.artifacts.numeric\_artifact

<a id="aqueduct.artifacts.numeric_artifact.NumericArtifact"></a>

## NumericArtifact Objects

```python
class NumericArtifact(BaseArtifact)
```

This class represents a computed number within the flow's DAG.

Any `@metric`-annotated python function that returns a number
will return this class when that function is called.

Any `@op`-annotated python function that returns a number
will return this class when that function is called in non-lazy mode.

**Examples**:

  >>> @metric
  >>> def compute_metric(df):
  >>>     return metric
  >>> metric_artifact = compute_metric(input_artifact)
  
  The contents of this artifact can be manifested locally.
  
  >>> val = metric_artifact.get()

<a id="aqueduct.artifacts.numeric_artifact.NumericArtifact.get"></a>

#### get

```python
def get(
    parameters: Optional[Dict[str,
                              Any]] = None) -> Union[int, float, np.number]
```

Materializes a NumericArtifact into its immediate float value.

**Returns**:

  The evaluated metric as a number.
  

**Raises**:

  InvalidRequestError:
  An error occurred because of an issue with the user's code or inputs.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.artifacts.numeric_artifact.NumericArtifact.list_preset_checks"></a>

#### list\_preset\_checks

```python
def list_preset_checks() -> List[str]
```

Returns a list of all preset checks available on the numeric artifact.
These preset checks can be set via the bound() method on a artifact.

**Returns**:

  A list of available preset checks on a metric

<a id="aqueduct.artifacts.numeric_artifact.NumericArtifact.bound"></a>

#### bound

```python
def bound(upper: Optional[float] = None,
          lower: Optional[float] = None,
          equal: Optional[float] = None,
          notequal: Optional[float] = None,
          severity: CheckSeverity = CheckSeverity.WARNING,
          lazy: bool = False) -> bool_artifact.BoolArtifact
```

Computes a bounds check on this metric with the specified boundary condition.

Only one of `upper` and `lower` can be set.

>>> metric_artifact.bound(upper = 0.9, severity = CheckSeverity.Error)

If the metric ever exceeds 0.9, the flow will fail.

**Arguments**:

  upper:
  Sets an upper bound on the value of the metric.
  lower:
  Sets a lower bound on the value of the metric.
  severity:
  If specified, will set the severity of this check as specified. Defaults to CheckSeverity.WARNING
  

**Returns**:

  A bool artifact bound to this metric.

<a id="aqueduct.artifacts.numeric_artifact.NumericArtifact.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the numeric artifact.

