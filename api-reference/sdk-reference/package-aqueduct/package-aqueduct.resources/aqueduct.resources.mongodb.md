# Table of Contents

* [aqueduct.resources.mongodb](#aqueduct.resources.mongodb)
  * [MongoDBCollectionResource](#aqueduct.resources.mongodb.MongoDBCollectionResource)
    * [find](#aqueduct.resources.mongodb.MongoDBCollectionResource.find)
    * [save](#aqueduct.resources.mongodb.MongoDBCollectionResource.save)
  * [MongoDBResource](#aqueduct.resources.mongodb.MongoDBResource)
    * [collection](#aqueduct.resources.mongodb.MongoDBResource.collection)
    * [describe](#aqueduct.resources.mongodb.MongoDBResource.describe)
    * [save](#aqueduct.resources.mongodb.MongoDBResource.save)

<a id="aqueduct.resources.mongodb"></a>

# aqueduct.resources.mongodb

<a id="aqueduct.resources.mongodb.MongoDBCollectionResource"></a>

## MongoDBCollectionResource Objects

```python
class MongoDBCollectionResource(BaseResource)
```

<a id="aqueduct.resources.mongodb.MongoDBCollectionResource.find"></a>

#### find

```python
@validate_is_connected()
def find(*args: List[Any],
         name: Optional[str] = None,
         output: Optional[str] = None,
         description: str = "",
         parameters: Optional[List[BaseArtifact]] = None,
         lazy: bool = False,
         **kwargs: Dict[str, Any]) -> BaseArtifact
```

`find` accepts almost exactly the same input signature as the `find` exposed by mongo:
https://www.mongodb.com/docs/manual/tutorial/query-documents/ .

Under the hood, we call mongo SDK's `find` API to extract from DB, using arguments you
provided to this function.

You can additionally provide the following keyword arguments:
name:
Name of the query.
description:
Description of the query.
output:
Name to assign the output artifact. If not set, the default naming scheme will be used.
parameters:
An optional list of string parameters to use in the query.  We use the Postgres syntax of $1, $2 for placeholders.
The number denotes which parameter in the list to use (one-indexed). These parameters feed into the
sql query operator and will fill in the placeholders in the query with the actual values.

**Example**:

  country1 = client.create_param("UK", default=" United Kingdom ")
  country2 = client.create_param("Thailand", default=" Thailand ")
  mongo_db_resource.collection("hotel_reviews").find(
  {
- `"reviewer_nationality"` - {
- `"$in"` - [$1, $2],
  }
  },
  parameters=[country1, country2],
  )
  
  The query will then be executed with:
- `"reviewer_nationality"` - {
- `"$in"` - [" United Kingdom ", " Thailand "],
  }
  
  
  lazy:
  Whether to run this operator lazily. See https://docs.aqueducthq.com/operators/lazy-vs.-eager-execution .

<a id="aqueduct.resources.mongodb.MongoDBCollectionResource.save"></a>

#### save

```python
@validate_is_connected()
def save(artifact: BaseArtifact, update_mode: LoadUpdateMode) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into this collection.
  update_mode:
  Defines the semantics of the save if a table already exists.
  Options are "replace", "append" (row-wise), or "fail" (if table already exists).

<a id="aqueduct.resources.mongodb.MongoDBResource"></a>

## MongoDBResource Objects

```python
class MongoDBResource(BaseResource)
```

Class for MongoDB resource. This works similar to mongo's `Database` object:

mongo_resource = client.resource("my_resource_name")
my_table_artifact = mongo_resource.collection("my_collection").find({})

<a id="aqueduct.resources.mongodb.MongoDBResource.collection"></a>

#### collection

```python
@validate_is_connected()
def collection(name: str) -> MongoDBCollectionResource
```

Returns a specific collection object to call `.find()` method.

**Example**:

  
  mongo_resource = client.resource("my_resource_name")
  my_table_artifact = mongo_resource.collection("my_collection").find({})

<a id="aqueduct.resources.mongodb.MongoDBResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the MongoDB resource.

<a id="aqueduct.resources.mongodb.MongoDBResource.save"></a>

#### save

```python
@validate_is_connected()
def save(artifact: BaseArtifact, collection: str,
         update_mode: LoadUpdateMode) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into the given collection.
  collection:
  The name of the collection to save to.
  update_mode:
  Defines the semantics of the save if a collection already exists.
  Options are "replace", "append" (row-wise), or "fail" (if table already exists).

