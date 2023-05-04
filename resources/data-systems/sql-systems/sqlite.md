---
description: Power your Aqueduct workflow with data from SQLite
---

# SQLite

## Connecting to SQLite

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Path**: The path to the file on the Aqueduct server at which the SQLite database lives.

{% hint style="info" %}
Since SQLite is an in-process database, the SQLite DB file _must_ be on the same machine as the Aqueduct server.
{% endhint %}

### UI

To connect Aqueduct to SQLite, navigate to the Aqueduct resources page and click on the SQLite icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.resources.connect_config import SQLiteConfig

client = aq.Client(s)

conf = SQLiteConfig(
    database="<PATH_TO_DATABASE>",
)
client.connect_resource("my_sqlite_resource", "SQLite", conf)
```

## Using SQLite

Once you've creating a SQLite connection, you can access your data from the Aqueduct SDK

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
RESOURCE_NAME = 'sqlite'

db = client.resource(RESOURCE_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Every relational database in Aqueduct supports two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
db.save(wines, "wines_2", "replace")
```
