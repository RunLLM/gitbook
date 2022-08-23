# Table of Contents

* [aqueduct.artifacts.generic\_artifact](#aqueduct.artifacts.generic_artifact)
  * [GenericArtifact](#aqueduct.artifacts.generic_artifact.GenericArtifact)
    * [get](#aqueduct.artifacts.generic_artifact.GenericArtifact.get)
    * [describe](#aqueduct.artifacts.generic_artifact.GenericArtifact.describe)

<a id="aqueduct.artifacts.generic_artifact"></a>

# aqueduct.artifacts.generic\_artifact

<a id="aqueduct.artifacts.generic_artifact.GenericArtifact"></a>

## GenericArtifact Objects

```python
class GenericArtifact(BaseArtifact)
```

This class represents a generic artifact within the flow's DAG.
Currently, a generic artifact can be any artifact other than table, numeric, bool, or parameter.

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

