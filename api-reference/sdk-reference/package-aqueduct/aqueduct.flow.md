# Table of Contents

* [aqueduct.flow](#aqueduct.flow)
  * [Flow](#aqueduct.flow.Flow)
    * [id](#aqueduct.flow.Flow.id)
    * [name](#aqueduct.flow.Flow.name)
    * [list\_runs](#aqueduct.flow.Flow.list_runs)
    * [list\_saved\_objects](#aqueduct.flow.Flow.list_saved_objects)
    * [describe](#aqueduct.flow.Flow.describe)

<a id="aqueduct.flow"></a>

# aqueduct.flow

<a id="aqueduct.flow.Flow"></a>

## Flow Objects

```python
class Flow()
```

This class is a read-only handle to flow already in the system.

Flows can have at multiple corresponding runs, and must have at least one.

<a id="aqueduct.flow.Flow.id"></a>

#### id

```python
def id() -> uuid.UUID
```

Returns the id of the flow.

<a id="aqueduct.flow.Flow.name"></a>

#### name

```python
def name() -> str
```

Returns the latest name of the flow.

<a id="aqueduct.flow.Flow.list_runs"></a>

#### list\_runs

```python
def list_runs(limit: int = 10) -> List[Dict[str, str]]
```

Lists the historical runs associated with this flow, sorted chronologically from most to least recent.

**Arguments**:

  limit:
  If set, we return only a limit number of the latest runs. Defaults to 10.
  

**Returns**:

  A list of dictionaries, each of which corresponds to a single flow run.
  Each dictionary contains essential information about the run (eg. id, status, etc.).

<a id="aqueduct.flow.Flow.list_saved_objects"></a>

#### list\_saved\_objects

```python
def list_saved_objects() -> DefaultDict[str, List[SavedObjectUpdate]]
```

Get everything saved by the flow.

**Returns**:

  A dictionary mapping the resource id to the list of table names/storage path.

<a id="aqueduct.flow.Flow.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the flow.

