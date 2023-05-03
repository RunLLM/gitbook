# aqueduct.integrations.integration

## Table of Contents

* [aqueduct.integrations.integration](aqueduct.integrations.integration.md#aqueduct.integrations.integration)
  * [IntegrationInfo](aqueduct.integrations.integration.md#aqueduct.integrations.integration.IntegrationInfo)
    * [describe](aqueduct.integrations.integration.md#aqueduct.integrations.integration.IntegrationInfo.describe)
    * [is\_relational](aqueduct.integrations.integration.md#aqueduct.integrations.integration.IntegrationInfo.is\_relational)
  * [Integration](aqueduct.integrations.integration.md#aqueduct.integrations.integration.Integration)
    * [\_\_hash\_\_](aqueduct.integrations.integration.md#aqueduct.integrations.integration.Integration.\_\_hash\_\_)
    * [\_\_eq\_\_](aqueduct.integrations.integration.md#aqueduct.integrations.integration.Integration.\_\_eq\_\_)

## aqueduct.integrations.integration

### IntegrationInfo Objects

```python
class IntegrationInfo(BaseModel)
```

**describe**

```python
def describe() -> None
```

Prints out a human-readable description of the integration.

**is\_relational**

```python
def is_relational() -> bool
```

Returns whether the integration connects to a relational data store.

### Integration Objects

```python
class Integration(ABC)
```

Abstract class for the various integrations Aqueduct interacts with.

**\_\_hash\_\_**

```python
def __hash__() -> int
```

An integration is uniquely identified by its name. Ref: https://docs.python.org/3.5/reference/datamodel.html#object.**hash**

**\_\_eq\_\_**

```python
def __eq__(other: Any) -> bool
```

The string and Integration object representation are equivalent allowing the user to access a dictionary keyed by the Integration object with the integration name as a string and vice versa
