# aqueduct.check\_artifact

## Table of Contents

* [aqueduct.check\_artifact](aqueduct.check\_artifact.md#aqueduct.check\_artifact)
  * [CheckArtifact](aqueduct.check\_artifact.md#aqueduct.check\_artifact.CheckArtifact)
    * [get](aqueduct.check\_artifact.md#aqueduct.check\_artifact.CheckArtifact.get)
    * [describe](aqueduct.check\_artifact.md#aqueduct.check\_artifact.CheckArtifact.describe)

## aqueduct.check\_artifact

### CheckArtifact Objects

```python
class CheckArtifact(Artifact)
```

This class represents a check within the flow's DAG.

Any `@check`-annotated python function that returns a boolean will return this class when that function is called called. This also is returned from pre-defined functions like metric.bound(...).

**Examples**:

> > > @check def check\_something(df1, metric) -> bool: return check\_result
> > >
> > > check\_artifact = check\_something(table\_artifact, metric\_artifact)

The contents of the check artifact can be manifested locally:

> > > assert check\_artifact.get()

**get**

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> bool
```

Materializes a CheckArtifact into a boolean.

**Returns**:

A boolean representing whether the check passed or not.

**Raises**:

InvalidRequestError: An error occurred because of an issue with the user's code or inputs. InternalServerError: An unexpected error occurred in the server.

**describe**

```python
def describe() -> None
```

Prints out a human-readable description of the check artifact.
