# Table of Contents

* [aqueduct.models.integration](#aqueduct.models.integration)
  * [ResourceInfo](#aqueduct.models.integration.ResourceInfo)
    * [describe](#aqueduct.models.integration.ResourceInfo.describe)
    * [is\_relational](#aqueduct.models.integration.ResourceInfo.is_relational)
  * [BaseResource](#aqueduct.models.integration.BaseResource)
    * [\_\_hash\_\_](#aqueduct.models.integration.BaseResource.__hash__)
    * [\_\_eq\_\_](#aqueduct.models.integration.BaseResource.__eq__)

<a id="aqueduct.models.integration"></a>

# aqueduct.models.integration

<a id="aqueduct.models.integration.ResourceInfo"></a>

## ResourceInfo Objects

```python
class ResourceInfo(BaseModel)
```

<a id="aqueduct.models.integration.ResourceInfo.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the integration.

<a id="aqueduct.models.integration.ResourceInfo.is_relational"></a>

#### is\_relational

```python
def is_relational() -> bool
```

Returns whether the integration connects to a relational data store.

<a id="aqueduct.models.integration.BaseResource"></a>

## BaseResource Objects

```python
class BaseResource(ABC)
```

Base class for the various integrations Aqueduct interacts with.

<a id="aqueduct.models.integration.BaseResource.__hash__"></a>

#### \_\_hash\_\_

```python
def __hash__() -> int
```

An integration is uniquely identified by its name.
Ref: https://docs.python.org/3.5/reference/datamodel.html#object.__hash__

<a id="aqueduct.models.integration.BaseResource.__eq__"></a>

#### \_\_eq\_\_

```python
def __eq__(other: Any) -> bool
```

The string and resource object representation are equivalent allowing
the user to access a dictionary keyed by the BaseResource object with the
integration name as a string and vice versa

