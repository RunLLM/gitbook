# Table of Contents

* [aqueduct.integrations.sql\_integration](#aqueduct.integrations.sql_integration)
  * [find\_parameter\_artifacts](#aqueduct.integrations.sql_integration.find_parameter_artifacts)
  * [RelationalDBIntegration](#aqueduct.integrations.sql_integration.RelationalDBIntegration)
    * [list\_tables](#aqueduct.integrations.sql_integration.RelationalDBIntegration.list_tables)
    * [table](#aqueduct.integrations.sql_integration.RelationalDBIntegration.table)
    * [sql](#aqueduct.integrations.sql_integration.RelationalDBIntegration.sql)
    * [config](#aqueduct.integrations.sql_integration.RelationalDBIntegration.config)
    * [save](#aqueduct.integrations.sql_integration.RelationalDBIntegration.save)
    * [describe](#aqueduct.integrations.sql_integration.RelationalDBIntegration.describe)

<a id="aqueduct.integrations.sql_integration"></a>

# aqueduct.integrations.sql\_integration

<a id="aqueduct.integrations.sql_integration.find_parameter_artifacts"></a>

#### find\_parameter\_artifacts

```python
def find_parameter_artifacts(dag: DAG,
                             names: List[str]) -> List[ArtifactMetadata]
```

`find_parameter_artifacts` finds all parameter artifacts corresponding to given `names`.
parameters:
    names: the list of names, repeating names are allowed.
returns:
    a list of unique parameter artifacts for these names. Built-in names are omitted.

raises: InvalidUserArgumentException if there's no parameter for the provided name.

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration"></a>

## RelationalDBIntegration Objects

```python
class RelationalDBIntegration(Integration)
```

Class for RealtionalDB integrations.

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.list_tables"></a>

#### list\_tables

```python
def list_tables() -> pd.DataFrame
```

Lists the tables available in the RealtionalDB integration.

**Returns**:

  pd.DataFrame of available tables.

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.table"></a>

#### table

```python
def table(name: str) -> pd.DataFrame
```

Retrieves a table from a RealtionalDB integration.

**Arguments**:

  name:
  The name of the table to retrieve.
  

**Returns**:

  pd.DataFrame of the table to retrieve.

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.sql"></a>

#### sql

```python
def sql(query: Union[str, List[str], RelationalDBExtractParams],
        name: Optional[str] = None,
        description: str = "",
        lazy: bool = False) -> TableArtifact
```

Runs a SQL query against the RelationalDB integration.

**Arguments**:

  query:
  The query to run. When a list is provided, we run the list
  in a chain and return the result of the final query.
  name:
  Name of the query.
  description:
  Description of the query.
  lazy:
  Whether to run this operator lazily. See https://docs.aqueducthq.com/operators/lazy-vs.-eager-execution .
  

**Returns**:

  TableArtifact representing result of the SQL query.

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.config"></a>

#### config

```python
def config(table: str, update_mode: LoadUpdateMode) -> SaveConfig
```

TODO(ENG-2035): Deprecated and will be removed.
Configuration for saving to RelationalDB Integration.

**Arguments**:

  table:
  Table to save to.
  update_mode:
  The update mode to use when saving the artifact as a relational table.
  Possible values are: APPEND, REPLACE, or FAIL.

**Returns**:

  SaveConfig object to use in TableArtifact.save()

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.save"></a>

#### save

```python
def save(artifact: BaseArtifact, table_name: str,
         update_mode: LoadUpdateMode) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into this sql integration.
  table_name:
  The table to save the artifact to.
  update_mode:
  Defines the semantics of the save if a table already exists.
  Options are "replace", "append" (row-wise), or "fail" (if table already exists).

<a id="aqueduct.integrations.sql_integration.RelationalDBIntegration.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the SQL integration.

