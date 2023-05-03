---
description: Power your Aqueduct workflow with data from MySQL
---

# MySQL

## Connecting to MySQL

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Host**: The hostname or IP address of your MySQL installation.
* **Port**: The port at which your MySQL DB is accessible; by default, MySQL uses port 3306.
* **Database**: The name of the database you're connecting to; each MySQL connection currently only supports a single database.
* **Username**: The username of the account via which you're connecting to MySQL.
* **Password**: The password corresponding to the above username.

### UI

To connect Aqueduct to MySQL, navigate to the Aqueduct resources page and click on the MySQL icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.resources.connect_config import MySQLConfig

client = aq.Client(s)

conf = MySQLConfig(
    username="<USERNAME>",
    password="<PASSWORD>",
    host="<HOSTNAME>",
    port="<PORT>"
    database="<DATABASE>",
)
client.connect_resource("my_mysql_resource", "MySQL", conf)
```

## Using MySQL

Once you've creating a MySQL connection, you can access your data from the Aqueduct SDK

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
RESOURCE_NAME = 'mysql'

db = client.resource(RESOURCE_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Every relational database in Aqueduct supports two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
db.save(wines, "wines_2", "replace")
```
