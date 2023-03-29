---
description: Power your Aqueduct workflow with data from BigQuery
---

# Google BigQuery

## Connecting to Google BigQuery

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Project ID**: The name of the Google BigQuery Project that Aqueduct should connect to.&#x20;
* **Service Account Credentials**: A JSON file that has the credentials for your Service Account.

{% hint style="info" %}
Google Cloud has documentation [here](https://cloud.google.com/docs/authentication/getting-started#creating\_a\_service\_account) that will walk you through creating a `ServiceAccount` and then creating a corresponding key for that `ServiceAccount`. Once you've finished these steps, you should have a JSON file with the key for your `ServiceAccount`.
{% endhint %}

### UI&#x20;

To connect Aqueduct to Google BigQuery, navigate to the Aqueduct integrations page and click on the Google BigQuery icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import BigQueryConfig

client = aq.Client(s)

conf = BigQueryConfig(
    project_id="<BQ_PROJECT_ID>",
    service_account_credentials_path="<PATH_TO_CREDENTIALS_FILE>"
)
client.connect_integration("my_bigquery_integration", "BigQuery", conf)
```

## Using Google BigQuery

Once you've creating a Google BigQuery connection, you can access your data from the Aqueduct SDK.

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'google_bigquery'

db = client.integration(INTEGRATION_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Every relational database in Aqueduct supports two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
db.save(wines, "wines_2", "replace")
```
