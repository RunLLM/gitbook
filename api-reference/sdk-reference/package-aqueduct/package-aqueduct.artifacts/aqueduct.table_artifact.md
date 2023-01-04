# aqueduct.table\_artifact

## Table of Contents

* [aqueduct.table\_artifact](aqueduct.table\_artifact.md#aqueduct.table\_artifact)
  * [TableArtifact](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact)
    * [get](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.get)
    * [head](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.head)
    * [save](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.save)
    * [list\_preset\_metrics](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.list\_preset\_metrics)
    * [list\_system\_metrics](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.list\_system\_metrics)
    * [validate\_with\_expectation](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.validate\_with\_expectation)
    * [number\_of\_missing\_values](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.number\_of\_missing\_values)
    * [number\_of\_rows](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.number\_of\_rows)
    * [max](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.max)
    * [min](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.min)
    * [mean](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.mean)
    * [std](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.std)
    * [system\_metric](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.system\_metric)
    * [\_\_str\_\_](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.\_\_str\_\_)
    * [describe](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.describe)
    * [remove\_check](aqueduct.table\_artifact.md#aqueduct.table\_artifact.TableArtifact.remove\_check)

## aqueduct.table\_artifact

### TableArtifact Objects

```python
class TableArtifact(Artifact)
```

This class represents a computed table within the flow's DAG.

Any `@op`-annotated python function that returns a dataframe will return this class when that function is called called.

**Examples**:

> > > @op def predict(df): return predictions
> > >
> > > output\_artifact = predict(input\_artifact)

The contents of these artifacts can be manifested locally or written to an integration:

> > > df = output\_artifact.get() print(df.head()) output\_artifact.save(warehouse.config(table\_name="output\_table"))

**get**

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> pd.DataFrame
```

Materializes TableArtifact into an actual dataframe.

**Arguments**:

parameters: A map from parameter name to its custom value, to be used when evaluating this artifact.

**Returns**:

A dataframe containing the tabular contents of this artifact.

**Raises**:

InvalidRequestError: An error occurred because of an issue with the user's code or inputs. InternalServerError: An unexpected error occurred within the Aqueduct cluster.

**head**

```python
def head(n: int = 5,
         parameters: Optional[Dict[str, Any]] = None) -> pd.DataFrame
```

Returns a preview of the table artifact.

> > > db = client.integration(name="demo/") customer\_data = db.sql("SELECT \* from customers") churn\_predictions = predict\_churn(customer\_data) churn\_predictions.head()

**Arguments**:

n: the number of row previewed. Default to 5.

**Returns**:

A dataframe containing the tabular contents of this artifact.

**save**

```python
def save(config: SaveConfig) -> None
```

Configure this artifact to be written to a specific integration after its computed.

> > > db = client.integration(name="demo/") customer\_data = db.sql("SELECT \* from customers") churn\_predictions = predict\_churn(customer\_data) churn\_predictions.save(config=db.config(table="churn\_predictions"))

**Arguments**:

config: SaveConfig object generated from integration using the .config(...) method.

**Raises**:

InvalidIntegrationException: An error occurred because the requested integration could not be found.

**list\_preset\_metrics**

```python
def list_preset_metrics() -> List[str]
```

Returns a list of all preset metrics available on the table artifact. These preset metrics can be set via the invoking the preset method on a artifact.

**Returns**:

A list of available preset metrics on a table

**list\_system\_metrics**

```python
def list_system_metrics() -> List[str]
```

Returns a list of all system metrics available on the table artifact. These system metrics can be set via the invoking the system\_metric() method the table.

**Returns**:

A list of available system metrics on a table

**validate\_with\_expectation**

```python
def validate_with_expectation(
        expectation_name: str,
        expectation_args: Optional[Dict[str, Any]] = None,
        severity: CheckSeverity = CheckSeverity.WARNING) -> CheckArtifact
```

Creates a check that validates with the table with great\_expectations and its set of internal expectations. The expectations supported can be found here: https://great-expectations.readthedocs.io/en/latest/reference/glossary\_of\_expectations.html The expectations supported are only those that can support Pandas on the great\_expectations backend. E.g. Use a expectation to check all column values are unique. ge\_check = table.validate\_with\_expectation("expect\_column\_values\_to\_be\_unique", {"column": "fixed\_acidity"}) ge\_check.get() // True or False based on expectation passing

**Arguments**:

expectation\_name: Name of built-in expectation to run with great\_expectations expectation\_args: Dictionary of args to pass into the expectation suite for the expectation being run. severity: Optional severity associated with the check created with this expectations

**Returns**:

A check artifact that represent the validation result of running the expectation provided on the table.

**number\_of\_missing\_values**

```python
def number_of_missing_values(column_id: Any = None,
                             row_id: Any = None) -> MetricArtifact
```

Creates a metric that represents the number of missing values over a given column or row.

Note: takes a scalar column\_id/row\_id and uses pandas.DataFrame.isnull() to compute value.

**Arguments**:

column\_id: column identifier to find missing values for row\_id: row identifier to find missing values for

**Returns**:

A metric artifact that represents the number of missing values for the row/column on the applied table artifact.

**number\_of\_rows**

```python
def number_of_rows() -> MetricArtifact
```

Creates a metric that represents the number of rows of this table

Note: uses len() to determine row count over the pandas.DataFrame.

**Returns**:

A metric artifact that represents the number of rows on this table.

**max**

```python
def max(column_id: Any) -> MetricArtifact
```

Creates a metric that represents the maximum value over the given column

Note: takes a scalar column\_id and uses pandas.DataFrame.max to compute value.

**Arguments**:

column\_id: column identifier to find max of

**Returns**:

A metric artifact that represents the max for the given column on the applied table artifact.

**min**

```python
def min(column_id: Any) -> MetricArtifact
```

Creates a metric that represents the minimum value over the given column

Note: takes a scalar column\_id and uses pandas.DataFrame.min to compute value.

**Arguments**:

column\_id: column identifier to find min of

**Returns**:

A metric artifact that represents the min for the given column on the applied table artifact.

**mean**

```python
def mean(column_id: Any) -> MetricArtifact
```

Creates a metric that represents the mean value over the given column

Note: takes a scalar column\_id and uses pandas.DataFrame.mean to compute value.

**Arguments**:

column\_id: column identifier to compute mean of

**Returns**:

A metric artifact that represents the mean for the given column on the applied table artifact.

**std**

```python
def std(column_id: Any) -> MetricArtifact
```

Creates a metric that represents the standard deviation value over the given column

takes a scalar column\_id and uses pandas.DataFrame.std to compute value

**Arguments**:

column\_id: column identifier to compute standard deviation of

**Returns**:

A metric artifact that represents the standard deviation for the given column on the applied table artifact.

**system\_metric**

```python
def system_metric(metric_name: str) -> MetricArtifact
```

Creates a system metric that represents the given system information from the previous @op that ran on the table.

**Arguments**:

metric\_name: name of system metric to retrieve for the table. valid metrics are:

* `runtime` - runtime of previous @op func in seconds
* `max_memory` - maximum memory usage of previous @op func in Mb

**Returns**:

A metric artifact that represents the requested system metric

**\_\_str\_\_**

```python
def __str__() -> str
```

Prints out a human-readable description of the table artifact.

**describe**

```python
def describe() -> None
```

Prints the stringified description of the table artifact to stdout.

**remove\_check**

```python
def remove_check(name: str) -> None
```

Remove a check on this artifact by name.

**Raises**:

* `InvalidUserActionException` - if a matching check operator could not be found.
