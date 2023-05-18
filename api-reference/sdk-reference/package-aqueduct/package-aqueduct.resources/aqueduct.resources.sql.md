# Table of Contents

* [aqueduct.resources.sql](#aqueduct.resources.sql)
  * [RelationalDBResource](#aqueduct.resources.sql.RelationalDBResource)
    * [list\_tables](#aqueduct.resources.sql.RelationalDBResource.list_tables)
    * [table](#aqueduct.resources.sql.RelationalDBResource.table)
    * [sql](#aqueduct.resources.sql.RelationalDBResource.sql)
    * [save](#aqueduct.resources.sql.RelationalDBResource.save)
    * [describe](#aqueduct.resources.sql.RelationalDBResource.describe)

<a id="aqueduct.resources.sql"></a>

# aqueduct.resources.sql

<a id="aqueduct.resources.sql.RelationalDBResource"></a>

## RelationalDBResource Objects

```python
class RelationalDBResource(BaseResource)
```

Class for Relational integrations.

<a id="aqueduct.resources.sql.RelationalDBResource.list_tables"></a>

#### list\_tables

```python
@validate_is_connected()
def list_tables() -> pd.DataFrame
```

Lists the tables available in the RelationalDB integration.

**Returns**:

  pd.DataFrame of available tables.

<a id="aqueduct.resources.sql.RelationalDBResource.table"></a>

#### table

```python
@validate_is_connected()
def table(name: str) -> pd.DataFrame
```

Retrieves a table from a RelationalDB integration.

**Arguments**:

  name:
  The name of the table to retrieve.
  

**Returns**:

  pd.DataFrame of the table to retrieve.

<a id="aqueduct.resources.sql.RelationalDBResource.sql"></a>

#### sql

```python
@validate_is_connected()
def sql(query: Union[str, List[str], RelationalDBExtractParams],
        name: Optional[str] = None,
        output: Optional[str] = None,
        description: str = "",
        parameters: Optional[List[BaseArtifact]] = None,
        lazy: bool = False) -> TableArtifact
```

Runs a SQL query against the RelationalDB integration.

**Arguments**:

  query:
  The query to run. When a list is provided, we run the list
  in a chain and return the result of the final query.
  name:
  Name of the query.
  output:
  Name to assign the output artifact. If not set, the default naming scheme will be used.
  description:
  Description of the query.
  parameters:
  An optional list of string parameters to use in the query. We use the Postgres syntax of $1, $2 for placeholders.
  The number denotes which parameter in the list to use (one-indexed). These parameters feed into the
  sql query operator and will fill in the placeholders in the query with the actual values.
  
  For example, for the following query with parameters=[param1, param2]:
  SELECT * FROM my_table where age = $1 and name = $2
  Assuming default values of "18" and "John" respectively, the default query will expand into
  SELECT * FROM my_table where age = 18 and name = "John".
  
  If multiple of the same placeholders are used in the same query, the same value will be supplied for each.
  lazy:
  Whether to run this operator lazily. See https://docs.aqueducthq.com/operators/lazy-vs.-eager-execution .
  

**Returns**:

  TableArtifact representing result of the SQL query.

<a id="aqueduct.resources.sql.RelationalDBResource.save"></a>

#### save

```python
@validate_is_connected()
def save(artifact: BaseArtifact, table_name: Union[str, BaseArtifact],
         update_mode: LoadUpdateMode) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into this sql integration.
  table_name:
  The table to save the artifact to. You can also parameterize this field by passing
  a string parameter here. When this save is parameterized, the table name parameter
  will always be ordered before the artifact in the save operator's input list.
  update_mode:
  Defines the semantics of the save if a table already exists.
  Options are "replace", "append" (row-wise), or "fail" (if table already exists).

<a id="aqueduct.resources.sql.RelationalDBResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the SQL integration.

