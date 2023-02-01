# Table of Contents

* [aqueduct.integrations.google\_sheets\_integration](#aqueduct.integrations.google_sheets_integration)
  * [GoogleSheetsIntegration](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration)
    * [spreadsheet](#aqueduct.integrations.google_sheets_integration.GoogleSheetsIntegration.spreadsheet)
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

