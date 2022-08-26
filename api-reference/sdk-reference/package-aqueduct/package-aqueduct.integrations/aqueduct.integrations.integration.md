# Table of Contents

* [aqueduct.integrations.integration](#aqueduct.integrations.integration)
  * [IntegrationInfo](#aqueduct.integrations.integration.IntegrationInfo)
    * [describe](#aqueduct.integrations.integration.IntegrationInfo.describe)
  * [Integration](#aqueduct.integrations.integration.Integration)
    * [\_\_hash\_\_](#aqueduct.integrations.integration.Integration.__hash__)
    * [\_\_eq\_\_](#aqueduct.integrations.integration.Integration.__eq__)

<a id="aqueduct.integrations.integration"></a>

# aqueduct.integrations.integration

<a id="aqueduct.integrations.integration.IntegrationInfo"></a>

## IntegrationInfo Objects

```python
class IntegrationInfo(BaseModel)
```

<a id="aqueduct.integrations.integration.IntegrationInfo.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the integration.

<a id="aqueduct.integrations.integration.Integration"></a>

## Integration Objects

```python
class Integration(ABC)
```

Abstract class for the various integrations Aqueduct interacts with.

<a id="aqueduct.integrations.integration.Integration.__hash__"></a>

#### \_\_hash\_\_

```python
def __hash__() -> int
```

An integration is uniquely identified by its name.
Ref: https://docs.python.org/3.5/reference/datamodel.html#object.__hash__

<a id="aqueduct.integrations.integration.Integration.__eq__"></a>

#### \_\_eq\_\_

```python
def __eq__(other: Any) -> bool
```

The string and Integration object representation are equivalent allowing
the user to access a dictionary keyed by the Integration object with the
integration name as a string and vice versa

