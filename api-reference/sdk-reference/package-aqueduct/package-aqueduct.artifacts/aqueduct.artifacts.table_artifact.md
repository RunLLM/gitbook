# Table of Contents

* [aqueduct.artifacts.table\_artifact](#aqueduct.artifacts.table_artifact)
  * [TableArtifact](#aqueduct.artifacts.table_artifact.TableArtifact)
    * [get](#aqueduct.artifacts.table_artifact.TableArtifact.get)
    * [head](#aqueduct.artifacts.table_artifact.TableArtifact.head)
    * [save](#aqueduct.artifacts.table_artifact.TableArtifact.save)
    * [list\_preset\_metrics](#aqueduct.artifacts.table_artifact.TableArtifact.list_preset_metrics)
    * [list\_system\_metrics](#aqueduct.artifacts.table_artifact.TableArtifact.list_system_metrics)
    * [validate\_with\_expectation](#aqueduct.artifacts.table_artifact.TableArtifact.validate_with_expectation)
    * [number\_of\_missing\_values](#aqueduct.artifacts.table_artifact.TableArtifact.number_of_missing_values)
    * [number\_of\_rows](#aqueduct.artifacts.table_artifact.TableArtifact.number_of_rows)
    * [max](#aqueduct.artifacts.table_artifact.TableArtifact.max)
    * [min](#aqueduct.artifacts.table_artifact.TableArtifact.min)
    * [mean](#aqueduct.artifacts.table_artifact.TableArtifact.mean)
    * [std](#aqueduct.artifacts.table_artifact.TableArtifact.std)
    * [system\_metric](#aqueduct.artifacts.table_artifact.TableArtifact.system_metric)
    * [\_\_str\_\_](#aqueduct.artifacts.table_artifact.TableArtifact.__str__)
    * [describe](#aqueduct.artifacts.table_artifact.TableArtifact.describe)
    * [remove\_check](#aqueduct.artifacts.table_artifact.TableArtifact.remove_check)

<a id="aqueduct.artifacts.table_artifact"></a>

# aqueduct.artifacts.table\_artifact

<a id="aqueduct.artifacts.table_artifact.TableArtifact"></a>

## TableArtifact Objects

```python
class TableArtifact(BaseArtifact)
```

This class represents a computed table within the flow's DAG.

Any `@op`-annotated python function that returns a dataframe will
return this class when that function is called called.

**Examples**:

  >>> @op
  >>> def predict(df):
  >>>     return predictions
  >>>
  >>> output_artifact = predict(input_artifact)
  
  The contents of these artifacts can be manifested locally or written to an
  integration:
  
  >>> df = output_artifact.get()
  >>> print(df.head())
  >>> output_artifact.save(warehouse.config(table_name="output_table"))

<a id="aqueduct.artifacts.table_artifact.TableArtifact.get"></a>

#### get

```python
def get(parameters: Optional[Dict[str, Any]] = None) -> pd.DataFrame
```

Materializes TableArtifact into an actual dataframe.

**Arguments**:

  parameters:
  A map from parameter name to its custom value, to be used when evaluating
  this artifact.
  

**Returns**:

  A dataframe containing the table contents of this artifact.
  

**Raises**:

  InvalidRequestError:
  An error occurred because of an issue with the user's code or inputs.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.head"></a>

#### head

```python
def head(n: int = 5,
         parameters: Optional[Dict[str, Any]] = None) -> pd.DataFrame
```

Returns a preview of the table artifact.

>>> db = client.integration(name="demo/")
>>> customer_data = db.sql("SELECT * from customers")
>>> churn_predictions = predict_churn(customer_data)
>>> churn_predictions.head()

**Arguments**:

  n:
  the number of row previewed. Default to 5.

**Returns**:

  A dataframe containing the table contents of this artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.save"></a>

#### save

```python
def save(config: SaveConfig) -> None
```

Configure this artifact to be written to a specific integration after its computed.

>>> db = client.integration(name="demo/")
>>> customer_data = db.sql("SELECT * from customers")
>>> churn_predictions = predict_churn(customer_data)
>>> churn_predictions.save(config=db.config(table="churn_predictions"))

**Arguments**:

  config:
  SaveConfig object generated from integration using
  the <integration>.config(...) method.

**Raises**:

  InvalidIntegrationException:
  An error occurred because the requested integration could not be
  found.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.list_preset_metrics"></a>

#### list\_preset\_metrics

```python
def list_preset_metrics() -> List[str]
```

Returns a list of all preset metrics available on the table artifact.
These preset metrics can be set via the invoking the preset method on a artifact.

**Returns**:

  A list of available preset metrics on a table

<a id="aqueduct.artifacts.table_artifact.TableArtifact.list_system_metrics"></a>

#### list\_system\_metrics

```python
def list_system_metrics() -> List[str]
```

Returns a list of all system metrics available on the table artifact.
These system metrics can be set via the invoking the system_metric() method the table.

**Returns**:

  A list of available system metrics on a table

<a id="aqueduct.artifacts.table_artifact.TableArtifact.validate_with_expectation"></a>

#### validate\_with\_expectation

```python
def validate_with_expectation(
    expectation_name: str,
    expectation_args: Optional[Dict[str, Any]] = None,
    severity: CheckSeverity = CheckSeverity.WARNING
) -> bool_artifact.BoolArtifact
```

Creates a check that validates with the table with great_expectations and its set of internal expectations.
The expectations supported can be found here:
https://great-expectations.readthedocs.io/en/latest/reference/glossary_of_expectations.html
The expectations supported are only those that can support Pandas on the great_expectations backend.
E.g. Use a expectation to check all column values are unique.
ge_check = table.validate_with_expectation("expect_column_values_to_be_unique", {"column": "fixed_acidity"})
ge_check.get() // True or False based on expectation passing

**Arguments**:

  expectation_name:
  Name of built-in expectation to run with great_expectations
  expectation_args:
  Dictionary of args to pass into the expectation suite for the expectation being run.
  severity:
  Optional severity associated with the check created with this expectations
  

**Returns**:

  A bool artifact that represent the validation result of running the expectation provided on the table.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.number_of_missing_values"></a>

#### number\_of\_missing\_values

```python
def number_of_missing_values(
        column_id: Any = None,
        row_id: Any = None) -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the number of missing values over a given column or row.

Note: takes a scalar column_id/row_id and uses pandas.DataFrame.isnull() to compute value.

**Arguments**:

  column_id:
  column identifier to find missing values for
  row_id:
  row identifier to find missing values for
  

**Returns**:

  A numeric artifact that represents the number of missing values for the row/column on the applied table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.number_of_rows"></a>

#### number\_of\_rows

```python
def number_of_rows() -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the number of rows of this table

Note: uses len() to determine row count over the pandas.DataFrame.

**Returns**:

  A numeric artifact that represents the number of rows on this table.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.max"></a>

#### max

```python
def max(column_id: Any) -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the maximum value over the given column

Note: takes a scalar column_id and uses pandas.DataFrame.max to compute value.

**Arguments**:

  column_id:
  column identifier to find max of
  

**Returns**:

  A numeric artifact that represents the max for the given column on the applied table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.min"></a>

#### min

```python
def min(column_id: Any) -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the minimum value over the given column

Note: takes a scalar column_id and uses pandas.DataFrame.min to compute value.

**Arguments**:

  column_id:
  column identifier to find min of
  

**Returns**:

  A numeric artifact that represents the min for the given column on the applied table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.mean"></a>

#### mean

```python
def mean(column_id: Any) -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the mean value over the given column

Note: takes a scalar column_id and uses pandas.DataFrame.mean to compute value.

**Arguments**:

  column_id:
  column identifier to compute mean of
  

**Returns**:

  A numeric artifact that represents the mean for the given column on the applied table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.std"></a>

#### std

```python
def std(column_id: Any) -> numeric_artifact.NumericArtifact
```

Creates a metric that represents the standard deviation value over the given column

takes a scalar column_id and uses pandas.DataFrame.std to compute value

**Arguments**:

  column_id:
  column identifier to compute standard deviation of
  

**Returns**:

  A numeric artifact that represents the standard deviation for the given column on the applied table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.system_metric"></a>

#### system\_metric

```python
def system_metric(metric_name: str) -> numeric_artifact.NumericArtifact
```

Creates a system metric that represents the given system information from the previous @op that ran on the table.

**Arguments**:

  metric_name:
  name of system metric to retrieve for the table.
  valid metrics are:
- `runtime` - runtime of previous @op func in seconds
- `max_memory` - maximum memory usage of previous @op func in Mb
  

**Returns**:

  A numeric artifact that represents the requested system metric

<a id="aqueduct.artifacts.table_artifact.TableArtifact.__str__"></a>

#### \_\_str\_\_

```python
def __str__() -> str
```

Prints out a human-readable description of the table artifact.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.describe"></a>

#### describe

```python
def describe() -> None
```

Prints the stringified description of the table artifact to stdout.

<a id="aqueduct.artifacts.table_artifact.TableArtifact.remove_check"></a>

#### remove\_check

```python
def remove_check(name: str) -> None
```

Remove a check on this artifact by name.

**Raises**:

- `InvalidUserActionException` - if a matching check operator could not be found.

