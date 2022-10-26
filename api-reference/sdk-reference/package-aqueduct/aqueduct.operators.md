# Table of Contents

* [aqueduct.operators](#aqueduct.operators)
  * [GithubMetadata](#aqueduct.operators.GithubMetadata)
  * [RelationalDBExtractParams](#aqueduct.operators.RelationalDBExtractParams)

<a id="aqueduct.operators"></a>

# aqueduct.operators

<a id="aqueduct.operators.GithubMetadata"></a>

## GithubMetadata Objects

```python
class GithubMetadata(BaseModel)
```

Specifies a destination in github integration.
There are two ways to specify the content:
-   by `path`, which points to a file or dir in the github repo.
-   from `repo_config_content_type` and `repo_config_content_name`, which points to
    information stored in the repo's `.aqconfig`.
If using `repo_config` content, backend will ignore `path` and overwrite it with
the `path` specified in `.aqconfig`.

<a id="aqueduct.operators.RelationalDBExtractParams"></a>

## RelationalDBExtractParams Objects

```python
class RelationalDBExtractParams(BaseModel)
```

Specifies the query to run when extracting from a relational DB.
Exactly one of the 3 fields should be set.

query: the string to run a single query.
queries: a list of strings to run a chain of queries.
github_metadata: Github information to run a query stored in github.

