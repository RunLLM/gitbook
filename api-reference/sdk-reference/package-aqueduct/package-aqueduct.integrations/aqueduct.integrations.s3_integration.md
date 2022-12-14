# Table of Contents

* [aqueduct.integrations.s3\_integration](#aqueduct.integrations.s3_integration)
  * [S3Integration](#aqueduct.integrations.s3_integration.S3Integration)
    * [file](#aqueduct.integrations.s3_integration.S3Integration.file)
    * [config](#aqueduct.integrations.s3_integration.S3Integration.config)
    * [save](#aqueduct.integrations.s3_integration.S3Integration.save)
    * [describe](#aqueduct.integrations.s3_integration.S3Integration.describe)

<a id="aqueduct.integrations.s3_integration"></a>

# aqueduct.integrations.s3\_integration

<a id="aqueduct.integrations.s3_integration.S3Integration"></a>

## S3Integration Objects

```python
class S3Integration(Integration)
```

Class for S3 integration.

<a id="aqueduct.integrations.s3_integration.S3Integration.file"></a>

#### file

```python
def file(filepaths: Union[List[str], str],
         artifact_type: ArtifactType,
         format: Optional[str] = None,
         merge: Optional[bool] = None,
         name: Optional[str] = None,
         description: str = "",
         lazy: bool = False) -> BaseArtifact
```

Reads one or more files from the S3 integration.

**Arguments**:

  filepaths:
  Filepath to retrieve from. The filepaths can either be:
  1) a single string that represents a file name or a directory name. The directory
  name must ends with a `/`. In case of a file name, we attempt to retrieve that file,
  and in case of a directory name, we do a prefix search on the directory and retrieve
  all matched files and concatenate them into a single file.
  2) a list of strings representing the file name. Note that in this case, we do not
  accept directory names in the list.
  artifact_type:
  The expected type of the S3 files. The `ArtifactType` class in `enums.py` contains all
  supported types, except for ArtifactType.UNTYPED. Note that when multiple files are
  retrieved, they must have the same artifact type.
  format:
  If the artifact type is ArtifactType.TABLE, the user has to specify the table format.
  We currently support JSON, CSV, and Parquet. Note that when multiple files are retrieved,
  they must have the same format.
  merge:
  If the artifact type is ArtifactType.TABLE, we can optionally merge multiple tables
  into a single DataFrame if this flag is set to True.
  name:
  Name of the query.
  description:
  Description of the query.
  

**Returns**:

  Artifact or a tuple of artifacts representing the S3 Files.

<a id="aqueduct.integrations.s3_integration.S3Integration.config"></a>

#### config

```python
def config(filepath: str,
           format: Optional[S3TableFormat] = None) -> SaveConfig
```

TODO(ENG-2035): Deprecated and will be removed.
Configuration for saving to S3 Integration.

**Arguments**:

  filepath:
  S3 Filepath to save to.
  format:
  S3 Fileformat to save as. Can be CSV, JSON, or Parquet.

**Returns**:

  SaveConfig object to use in Artifact.save()

<a id="aqueduct.integrations.s3_integration.S3Integration.save"></a>

#### save

```python
def save(artifact: BaseArtifact,
         filepath: str,
         format: Optional[S3TableFormat] = None) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into S3.
  filepath:
  The S3 path to save to. Will overwrite any existing object at that path.
  format:
  Defines the format that the artifact will be saved as.
  Options are "CSV", "JSON", "Parquet".

<a id="aqueduct.integrations.s3_integration.S3Integration.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the S3 integration.

