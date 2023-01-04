# aqueduct.enums

## Table of Contents

* [aqueduct.enums](aqueduct.enums.md#aqueduct.enums)
  * [MetaEnum](aqueduct.enums.md#aqueduct.enums.MetaEnum)
  * [CheckSeverity](aqueduct.enums.md#aqueduct.enums.CheckSeverity)
  * [GithubRepoConfigContentType](aqueduct.enums.md#aqueduct.enums.GithubRepoConfigContentType)

## aqueduct.enums

### MetaEnum Objects

```python
class MetaEnum(EnumMeta)
```

Allows to very easily check if strings are present in the enum, without a helper.

Eg. if "Postgres" in ServiceType: ...

### CheckSeverity Objects

```python
class CheckSeverity(str, Enum, metaclass=MetaEnum)
```

An ERROR severity will fail the flow.

### GithubRepoConfigContentType Objects

```python
class GithubRepoConfigContentType(str, Enum, metaclass=MetaEnum)
```

Github repo config (.aqconfig) content type.
