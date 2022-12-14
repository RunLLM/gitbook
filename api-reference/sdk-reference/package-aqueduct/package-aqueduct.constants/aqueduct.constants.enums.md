# Table of Contents

* [aqueduct.constants.enums](#aqueduct.constants.enums)
  * [MetaEnum](#aqueduct.constants.enums.MetaEnum)
  * [CheckSeverity](#aqueduct.constants.enums.CheckSeverity)
  * [GithubRepoConfigContentType](#aqueduct.constants.enums.GithubRepoConfigContentType)

<a id="aqueduct.constants.enums"></a>

# aqueduct.constants.enums

<a id="aqueduct.constants.enums.MetaEnum"></a>

## MetaEnum Objects

```python
class MetaEnum(EnumMeta)
```

Allows to very easily check if strings are present in the enum, without a helper.

Eg.
    if "Postgres" in ServiceType:
        ...

<a id="aqueduct.constants.enums.CheckSeverity"></a>

## CheckSeverity Objects

```python
class CheckSeverity(str, Enum, metaclass=MetaEnum)
```

An ERROR severity will fail the flow.

<a id="aqueduct.constants.enums.GithubRepoConfigContentType"></a>

## GithubRepoConfigContentType Objects

```python
class GithubRepoConfigContentType(str, Enum, metaclass=MetaEnum)
```

Github repo config (.aqconfig) content type.

