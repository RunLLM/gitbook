# Table of Contents

* [aqueduct.models.resource](#aqueduct.models.resource)
  * [ResourceInfo](#aqueduct.models.resource.ResourceInfo)
    * [describe](#aqueduct.models.resource.ResourceInfo.describe)
    * [is\_relational](#aqueduct.models.resource.ResourceInfo.is_relational)
  * [BaseResource](#aqueduct.models.resource.BaseResource)
    * [\_\_hash\_\_](#aqueduct.models.resource.BaseResource.__hash__)
    * [\_\_eq\_\_](#aqueduct.models.resource.BaseResource.__eq__)

<a id="aqueduct.models.resource"></a>

# aqueduct.models.resource

<a id="aqueduct.models.resource.ResourceInfo"></a>

## ResourceInfo Objects

```python
class ResourceInfo(BaseModel)
```

<a id="aqueduct.models.resource.ResourceInfo.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the resource.

<a id="aqueduct.models.resource.ResourceInfo.is_relational"></a>

#### is\_relational

```python
def is_relational() -> bool
```

Returns whether the resource connects to a relational data store.

<a id="aqueduct.models.resource.BaseResource"></a>

## BaseResource Objects

```python
class BaseResource(ABC)
```

Base class for the various resources Aqueduct interacts with.

<a id="aqueduct.models.resource.BaseResource.__hash__"></a>

#### \_\_hash\_\_

```python
def __hash__() -> int
```

An resource is uniquely identified by its name.
Ref: https://docs.python.org/3.5/reference/datamodel.html#object.__hash__

<a id="aqueduct.models.resource.BaseResource.__eq__"></a>

#### \_\_eq\_\_

```python
def __eq__(other: Any) -> bool
```

The string and resource object representation are equivalent allowing
the user to access a dictionary keyed by the BaseResource object with the
resource name as a string and vice versa

