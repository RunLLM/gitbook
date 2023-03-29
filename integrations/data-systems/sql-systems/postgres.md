---
description: Power your Aqueduct workflow with data from Postgres
---

# Postgres

## Connecting to Postgres

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Host**: The hostname or IP address of your Postgres installation.
* **Port**: The port at which your Postgres DB is accessible; by default, Postgres uses port 5432.
* **Database**: The name of the database you're connecting to; each Postgres connection currently only supports a single database.
* **Username**: The username of the account via which you're connecting to Postgres.
* **Password**: The password corresponding to the above username.

### UI

To connect Aqueduct to Postgres, navigate to the Aqueduct integrations page and click on the Postgres icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import PostgresConfig

client = aq.Client(s)

conf = PostgresConfig(
    username="<USERNAME>",
    password="<PASSWORD>",
    host="<HOSTNAME>",
    port="<PORT>"
    database="<DATABASE>",
)
client.connect_integration("my_postgres_integration", "Postgres", conf)
```

## Using Postgres

Once you've creating a Postgres connection, you can access your data from the Aqueduct SDK.

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'postgres'

db = client.integration(INTEGRATION_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Every relational database in Aqueduct supports two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
db.save(wines, "wines_2", "replace")
```
