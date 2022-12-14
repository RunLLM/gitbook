# Table of Contents

* [aqueduct.integrations.google\_sheets\_integration](#aqueduct.integrations.google_sheets_integration)
  * [GoogleSheetsIntegration](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration)
    * [spreadsheet](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.spreadsheet)
    * [config](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.config)
    * [save](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.save)
    * [describe](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.describe)

<a id="aqueduct.integrations.google_sheets_integration"></a>

# aqueduct.integrations.google\_sheets\_integration

<a id="aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration"></a>

## GoogleSheetsIntegration Objects

```python
class GoogleSheetsIntegration(Integration)
```

Class for Google Sheets integration.

<a id="aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.spreadsheet"></a>

#### spreadsheet

```python
def spreadsheet(spreadsheet_id: str,
                name: Optional[str] = None,
                description: str = "") -> TableArtifact
```

Retrieves a spreadsheet from the Google Sheets integration.

**Arguments**:

  spreadsheet_id:
  Id of spreadsheet to retrieve. This can be found in the URL of the spreadsheet, e.g.
  https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit#gid=0
  name:
  Name of the query.
  description:
  Description of the query.
  

**Returns**:

  TableArtifact representing the Google Sheet.

<a id="aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.config"></a>

#### config

```python
def config(
    filepath: str,
    save_mode: GoogleSheetsSaveMode = GoogleSheetsSaveMode.OVERWRITE
) -> SaveConfig
```

TODO(ENG-2035): Deprecated and will be removed.
Configuration for saving to Google Sheets Integration.

**Arguments**:

  filepath:
  Google Sheets filepath to save to. Any directory in the path that does not exist
  will be created.
  save_mode:
  The save mode to use when saving this artifact to Google Sheets.
  Possible values are:
  - OVERWRITE: If a spreadsheet with the same name exists, it will overwrite it.
  Otherwise, it will create a new spreadsheet.
  - NEWSHEET: If a spreadsheet with the same name exists, it will create a new sheet for this artifact.
  Otherwise, it will create a new spreadsheet.
  - CREATE: This will create a new spreadsheet, even if one with the same name exists. The previous
  spreadsheet will not be deleted, as Google Sheets allows for multiple spreadsheets of the same name.

**Returns**:

  SaveConfig object to use in TableArtifact.save()

<a id="aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.save"></a>

#### save

```python
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

<a id="aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the google sheets integration.

