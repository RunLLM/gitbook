---
description: Power your Aqueduct workflow with data from Snowflake
---

# Snowflake

## Connecting to Snowflake

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Account Identifier**: The unique ID of your Snowflake cluster. This is in the URL for your login; you will see something like `abc12345.snowflakecomputing.com`; your account's identifier is `abc12345`.
* **Warehouse**: The name of the compute warehouse to use for your Snowflake queries.&#x20;
* **Database**: The name of the database you're connecting to; each Snowflake connection currently only supports a single database.
* **Schema (optional)**: If required, the name of the schema within the above database to use. If left empty, the default schema will be used.&#x20;
* **Username**: The username of the account via which you're connecting to Snowflake.
* **Password**: The password corresponding to the above username.
* **Role (optional)**: If the user has multiple roles, the role which will be used when accessing the above database. If left empty, the default role will be used.

### UI

To connect Aqueduct to Snowflake, navigate to the Aqueduct integrations page and click on the Snowflake icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import SnowflakeConfig

client = aq.Client(s)

conf = SnowflakeConfig(
    username="<USERNAME>",
    password="<PASSWORD>",
    account_identifier="<ACCT_ID>",
    database="<DATABASE>",
    warehouse="<WAREHOUSE>",
    schema="<SCHEMA>",
    role="<ROLE>"
)
client.connect_integration("my_snowflake_integration", "Snowflake", conf)

```

## Using Snowflake

Once you've creating a Snowflake connection, you can access your data from the Aqueduct SDK.

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'snowflake'

db = client.integration(INTEGRATION_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Every relational database in Aqueduct supports two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
db.save(wines, "wines_2", "replace")
```
