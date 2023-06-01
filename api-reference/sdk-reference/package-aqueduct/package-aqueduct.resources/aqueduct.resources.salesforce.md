# Table of Contents

* [aqueduct.resources.salesforce](#aqueduct.resources.salesforce)
  * [SalesforceResource](#aqueduct.resources.salesforce.SalesforceResource)
    * [search](#aqueduct.resources.salesforce.SalesforceResource.search)
    * [query](#aqueduct.resources.salesforce.SalesforceResource.query)
    * [save](#aqueduct.resources.salesforce.SalesforceResource.save)
    * [describe](#aqueduct.resources.salesforce.SalesforceResource.describe)

<a id="aqueduct.resources.salesforce"></a>

# aqueduct.resources.salesforce

<a id="aqueduct.resources.salesforce.SalesforceResource"></a>

## SalesforceResource Objects

```python
class SalesforceResource(BaseResource)
```

Class for Salesforce resource.

<a id="aqueduct.resources.salesforce.SalesforceResource.search"></a>

#### search

```python
@validate_is_connected()
def search(search_query: str,
           name: Optional[str] = None,
           output: Optional[str] = None,
           description: str = "") -> TableArtifact
```

Runs a search against the Salesforce resource.

**Arguments**:

  search_query:
  The search query to run.
  name:
  Name of the query.
  output:
  Name to assign the output artifact. If not set, the default naming scheme will be used.
  description:
  Description of the query.
  

**Returns**:

  TableArtifact representing result of the SQL query.

<a id="aqueduct.resources.salesforce.SalesforceResource.query"></a>

#### query

```python
@validate_is_connected()
def query(query: str,
          name: Optional[str] = None,
          output: Optional[str] = None,
          description: str = "") -> TableArtifact
```

Runs a query against the Salesforce resource.

**Arguments**:

  query:
  The query to run.
  name:
  Name of the query.
  output:
  Name to assign the output artifact. If not set, the default naming scheme will be used.
  description:
  Description of the query.
  

**Returns**:

  TableArtifact representing result of the SQL query.

<a id="aqueduct.resources.salesforce.SalesforceResource.save"></a>

#### save

```python
@validate_is_connected()
def save(artifact: BaseArtifact, object: str) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into Salesforce.
  object:
  The name of the Salesforce object to save to.

<a id="aqueduct.resources.salesforce.SalesforceResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the Salesforce resource.

