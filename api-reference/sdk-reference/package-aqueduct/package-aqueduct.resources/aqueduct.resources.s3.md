# Table of Contents

* [aqueduct.resources.s3](#aqueduct.resources.s3)
  * [S3Resource](#aqueduct.resources.s3.S3Resource)
    * [file](#aqueduct.resources.s3.S3Resource.file)
    * [save](#aqueduct.resources.s3.S3Resource.save)
    * [describe](#aqueduct.resources.s3.S3Resource.describe)

<a id="aqueduct.resources.s3"></a>

# aqueduct.resources.s3

<a id="aqueduct.resources.s3.S3Resource"></a>

## S3Resource Objects

```python
class S3Resource(BaseResource)
```

Class for S3 integration.

<a id="aqueduct.resources.s3.S3Resource.file"></a>

#### file

```python
@validate_is_connected()
def file(filepaths: Union[List[str], str],
         artifact_type: ArtifactType,
         format: Optional[str] = None,
         merge: Optional[bool] = None,
         name: Optional[str] = None,
         output: Optional[str] = None,
         description: str = "",
         lazy: bool = False) -> BaseArtifact
```

Reads one or more files from the S3 integration.

**Arguments**:

  filepaths:
  Filepath to retrieve from. The filepaths can either be:
  1) a single string that represents a file name or a directory name. The directory
  name must ends with a `/`. In case of a file name, we attempt to retrieve that file.
  In case of a directory name, we do a prefix search on the directory and retrieve
  all matched files in alphabetical order, returning them as a TUPLE artifact.
  2) a list of strings representing the file name. Note that in this case, we do not
  accept directory names in the list. The fetched data in this case will always be of
  ArtifactType.TUPLE.
  artifact_type:
  The expected type of the S3 files. The `ArtifactType` class in `enums.py` contains all
  supported types, except for ArtifactType.UNTYPED. Note that when multiple files are
  retrieved, they must have the same artifact type.
  format:
  If the artifact type is ArtifactType.TABLE, the user has to specify the table format.
  We currently support JSON, CSV, and Parquet. Note that when multiple table files are
  retrieved, they must have the same format.
  merge:
  If the artifact type is ArtifactType.TABLE, we can optionally merge multiple tables
  into a single DataFrame if this flag is set to True. This merge is done with
  `pandas.concat(tables, ignore_index=True)`.
  name:
  Name of the query.
  output:
  Name to assign the output artifact. If not set, the default naming scheme will be used.
  description:
  Description of the query.
  lazy:
  Whether to run this operator lazily. See https://docs.aqueducthq.com/operators/lazy-vs.-eager-execution .
  

**Returns**:

  An artifact representing the S3 File(s). If multiple files are expected, the artifact
  will represent a tuple.

<a id="aqueduct.resources.s3.S3Resource.save"></a>

#### save

```python
@validate_is_connected()
def save(artifact: BaseArtifact,
         filepath: str,
         format: Optional[str] = None) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into S3.
  filepath:
  The S3 path to save to. Will overwrite any existing object at that path.
  format:
  Only required if saving a table artifact. Options are case-insensitive "json", "csv", "parquet".

<a id="aqueduct.resources.s3.S3Resource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the S3 integration.

