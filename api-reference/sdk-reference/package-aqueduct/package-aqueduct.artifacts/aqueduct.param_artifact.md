# aqueduct.param\_artifact

## Table of Contents

* [aqueduct.param\_artifact](aqueduct.param\_artifact.md#aqueduct.param\_artifact)
  * [ParamArtifact](aqueduct.param\_artifact.md#aqueduct.param\_artifact.ParamArtifact)
    * [\_\_init\_\_](aqueduct.param\_artifact.md#aqueduct.param\_artifact.ParamArtifact.\_\_init\_\_)

## aqueduct.param\_artifact

### ParamArtifact Objects

```python
class ParamArtifact(Artifact)
```

**\_\_init\_\_**

```python
def __init__(dag: DAG, artifact_id: uuid.UUID, from_flow_run: bool = False)
```

The APIClient is only included because decorated functions operators acting on this parameter will need a handle to an API client.
