# Table of Contents

* [aqueduct.resources.google\_sheets](#aqueduct.resources.google_sheets)
  * [GoogleSheetsResource](#aqueduct.resources.google_sheets.GoogleSheetsResource)
    * [spreadsheet](#aqueduct.resources.google_sheets.GoogleSheetsResource.spreadsheet)
    * [save](#aqueduct.resources.google_sheets.GoogleSheetsResource.save)
    * [describe](#aqueduct.resources.google_sheets.GoogleSheetsResource.describe)

<a id="aqueduct.resources.google_sheets"></a>

# aqueduct.resources.google\_sheets

<a id="aqueduct.resources.google_sheets.GoogleSheetsResource"></a>

## GoogleSheetsResource Objects

```python
class GoogleSheetsResource(BaseResource)
```

Class for Google Sheets integration.

<a id="aqueduct.resources.google_sheets.GoogleSheetsResource.spreadsheet"></a>

#### spreadsheet

```python
@validate_is_connected()
def spreadsheet(spreadsheet_id: str,
                name: Optional[str] = None,
                output: Optional[str] = None,
                description: str = "") -> TableArtifact
```

Retrieves a spreadsheet from the Google Sheets integration.

**Arguments**:

  spreadsheet_id:
  Id of spreadsheet to retrieve. This can be found in the URL of the spreadsheet, e.g.
  https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit#gid=0
  name:
  Name of the query.
  output:
  Name to assign the output artifact. If not set, the default naming scheme will be used.
  description:
  Description of the query.
  

**Returns**:

  TableArtifact representing the Google Sheet.

<a id="aqueduct.resources.google_sheets.GoogleSheetsResource.save"></a>

#### save

```python
@validate_is_connected()
def save(
        artifact: BaseArtifact,
        filepath: str,
        save_mode: GoogleSheetsSaveMode = GoogleSheetsSaveMode.OVERWRITE
) -> None
```

Registers a save operator of the given artifact, to be executed when it's computed in a published flow.

**Arguments**:

  artifact:
  The artifact to save into Google Sheets.
  filepath:
  The absolute file path to the Google Sheet to save to.
  save_mode:
  Defines the semantics of the save. Options are
  - "overwrite"
  - "create": Creates a new spreadsheet.
  If the spreadsheet doesn't exist, has `overwrite` behavior.
  - "newsheet": Creates a new sheet in an existing spreadsheet.
  If the spreadsheet doesn't exist, has `create` behavior.

<a id="aqueduct.resources.google_sheets.GoogleSheetsResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the google sheets integration.

