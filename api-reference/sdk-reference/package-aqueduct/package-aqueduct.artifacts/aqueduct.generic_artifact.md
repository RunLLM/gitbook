# aqueduct.generic\_artifact

## Table of Contents

* [aqueduct.generic\_artifact](aqueduct.generic\_artifact.md#aqueduct.generic\_artifact)
  * [Artifact](aqueduct.generic\_artifact.md#aqueduct.generic\_artifact.Artifact)
    * [id](aqueduct.generic\_artifact.md#aqueduct.generic\_artifact.Artifact.id)
    * [name](aqueduct.generic\_artifact.md#aqueduct.generic\_artifact.Artifact.name)

## aqueduct.generic\_artifact

### Artifact Objects

```python
class Artifact(ABC)
```

**id**

```python
def id() -> uuid.UUID
```

Fetch the id associated with this artifact.

This id will not exist in the system if the artifact has not yet been published.

**name**

```python
def name() -> str
```

Fetch the name of this artifact.
