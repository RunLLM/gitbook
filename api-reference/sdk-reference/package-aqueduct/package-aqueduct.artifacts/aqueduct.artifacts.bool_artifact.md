# Table of Contents

* [aqueduct.artifacts.bool\_artifact](#aqueduct.artifacts.bool_artifact)
  * [BoolArtifact](#aqueduct.artifacts.bool_artifact.BoolArtifact)
    * [get](#aqueduct.artifacts.bool_artifact.BoolArtifact.get)
    * [describe](#aqueduct.artifacts.bool_artifact.BoolArtifact.describe)

<a id="aqueduct.artifacts.bool_artifact"></a>

# aqueduct.artifacts.bool\_artifact

<a id="aqueduct.artifacts.bool_artifact.BoolArtifact"></a>

## BoolArtifact Objects

```python
class BoolArtifact(BaseArtifact)
```

This class represents a bool within the flow's DAG.

Any `@check`-annotated python function that returns a boolean will
return this class when that function is called. This class can also
be returned from pre-defined functions like metric.bound(...).

Any `@op`-annotated python function that returns a boolean
will return this class when that function is called in non-lazy mode.

**Examples**:

  >>> @check
  >>> def check_something(df1, metric) -> bool:
  >>>     return check_result
  >>>
  >>> check_artifact = check_something(table_artifact, metric_artifact)
  
  The contents of the bool artifact can be manifested locally:
  
  >>> assert check_artifact.get()

<a id="aqueduct.artifacts.bool_artifact.BoolArtifact.get"></a>

#### get

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> Union[bool, np.bool_]
```

Materializes a BoolArtifact into a boolean.

**Returns**:

  A boolean representing whether the check passed or not.
  

**Raises**:

  InvalidRequestError:
  An error occurred because of an issue with the user's code or inputs.
  InternalServerError:
  An unexpected error occurred in the server.

<a id="aqueduct.artifacts.bool_artifact.BoolArtifact.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the bool artifact.

