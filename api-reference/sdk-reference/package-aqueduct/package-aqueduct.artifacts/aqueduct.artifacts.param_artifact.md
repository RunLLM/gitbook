# Table of Contents

* [aqueduct.artifacts.param\_artifact](#aqueduct.artifacts.param_artifact)
  * [ParamArtifact](#aqueduct.artifacts.param_artifact.ParamArtifact)
    * [\_\_init\_\_](#aqueduct.artifacts.param_artifact.ParamArtifact.__init__)

<a id="aqueduct.artifacts.param_artifact"></a>

# aqueduct.artifacts.param\_artifact

<a id="aqueduct.artifacts.param_artifact.ParamArtifact"></a>

## ParamArtifact Objects

```python
class ParamArtifact(BaseArtifact)
```

<a id="aqueduct.artifacts.param_artifact.ParamArtifact.__init__"></a>

#### \_\_init\_\_

```python
def __init__(dag: DAG, artifact_id: uuid.UUID, from_flow_run: bool = False)
```

The APIClient is only included because decorated functions operators acting on this parameter
will need a handle to an API client.

