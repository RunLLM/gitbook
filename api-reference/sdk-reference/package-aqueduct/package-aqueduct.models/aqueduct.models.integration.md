# Table of Contents

* [aqueduct.models.integration](#aqueduct.models.integration)
  * [IntegrationInfo](#aqueduct.models.integration.IntegrationInfo)
    * [describe](#aqueduct.models.integration.IntegrationInfo.describe)
    * [is\_relational](#aqueduct.models.integration.IntegrationInfo.is_relational)
  * [Integration](#aqueduct.models.integration.Integration)
    * [\_\_hash\_\_](#aqueduct.models.integration.Integration.__hash__)
    * [\_\_eq\_\_](#aqueduct.models.integration.Integration.__eq__)

<a id="aqueduct.models.integration"></a>

# aqueduct.models.integration

<a id="aqueduct.models.integration.IntegrationInfo"></a>

## IntegrationInfo Objects

```python
class IntegrationInfo(BaseModel)
```

<a id="aqueduct.models.integration.IntegrationInfo.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the integration.

<a id="aqueduct.models.integration.IntegrationInfo.is_relational"></a>

#### is\_relational

```python
def is_relational() -> bool
```

Returns whether the integration connects to a relational data store.

<a id="aqueduct.models.integration.Integration"></a>

## Integration Objects

```python
class Integration(ABC)
```

Base class for the various integrations Aqueduct interacts with.

<a id="aqueduct.models.integration.Integration.__hash__"></a>

#### \_\_hash\_\_

```python
def __hash__() -> int
```

An integration is uniquely identified by its name.
Ref: https://docs.python.org/3.5/reference/datamodel.html#object.__hash__

<a id="aqueduct.models.integration.Integration.__eq__"></a>

#### \_\_eq\_\_

```python
def __eq__(other: Any) -> bool
```

The string and Integration object representation are equivalent allowing
the user to access a dictionary keyed by the Integration object with the
integration name as a string and vice versa

