# Table of Contents

* [aqueduct.constants.enums](#aqueduct.constants.enums)
  * [MetaEnum](#aqueduct.constants.enums.MetaEnum)
  * [CheckSeverity](#aqueduct.constants.enums.CheckSeverity)
  * [RelationalDBServices](#aqueduct.constants.enums.RelationalDBServices)
  * [GithubRepoConfigContentType](#aqueduct.constants.enums.GithubRepoConfigContentType)
  * [ArtifactType](#aqueduct.constants.enums.ArtifactType)
    * [IMAGE](#aqueduct.constants.enums.ArtifactType.IMAGE)

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

<a id="aqueduct.constants.enums.RelationalDBServices"></a>

## RelationalDBServices Objects

```python
class RelationalDBServices(str, Enum, metaclass=MetaEnum)
```

Must match the corresponding entries in `ServiceType` exactly.

<a id="aqueduct.constants.enums.GithubRepoConfigContentType"></a>

## GithubRepoConfigContentType Objects

```python
class GithubRepoConfigContentType(str, Enum, metaclass=MetaEnum)
```

Github repo config (.aqconfig) content type.

<a id="aqueduct.constants.enums.ArtifactType"></a>

## ArtifactType Objects

```python
class ArtifactType(str, Enum, metaclass=MetaEnum)
```

<a id="aqueduct.constants.enums.ArtifactType.IMAGE"></a>

#### IMAGE

corresponds to PIL.Image.Image type

