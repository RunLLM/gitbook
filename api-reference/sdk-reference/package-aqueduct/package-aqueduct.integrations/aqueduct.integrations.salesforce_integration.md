# Table of Contents

* [aqueduct.integrations.salesforce\_integration](#aqueduct.integrations.salesforce_integration)
  * [SalesforceIntegration](#aqueduct.integrations.salesforce_integration.SalesforceIntegration)
    * [search](#aqueduct.integrations.salesforce_integration.SalesforceIntegration.search)
    * [query](#aqueduct.integrations.salesforce_integration.SalesforceIntegration.query)
    * [save](#aqueduct.integrations.salesforce_integration.SalesforceIntegration.save)
    * [describe](#aqueduct.integrations.salesforce_integration.SalesforceIntegration.describe)

<a id="aqueduct.integrations.salesforce_integration"></a>

# aqueduct.integrations.salesforce\_integration

<a id="aqueduct.integrations.salesforce_integration.SalesforceIntegration"></a>

## SalesforceIntegration Objects

```python
class SalesforceIntegration(Integration)
```

Class for Salesforce integration.

<a id="aqueduct.integrations.salesforce_integration.SalesforceIntegration.search"></a>

#### search

```python
def search(search_query: str,
           name: Optional[str] = None,
           output: Optional[str] = None,
           description: str = "") -> TableArtifact
```

Runs a search against the Salesforce integration.

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

<a id="aqueduct.integrations.salesforce_integration.SalesforceIntegration.query"></a>

#### query

```python
def query(query: str,
          name: Optional[str] = None,
          output: Optional[str] = None,
          description: str = "") -> TableArtifact
```

Runs a query against the Salesforce integration.

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

<a id="aqueduct.integrations.salesforce_integration.SalesforceIntegration.save"></a>

#### save

```python
def save(artifact: BaseArtifact, object: str) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into Salesforce.
  object:
  The name of the Salesforce object to save to.

<a id="aqueduct.integrations.salesforce_integration.SalesforceIntegration.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the Salesforce integration.

