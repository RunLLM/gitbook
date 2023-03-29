---
description: Power your Aqueduct workflow with data from MongoDB
---

# MongoDB

## Connecting to MongoDB

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **URI**: The address of your MongoDB server.
* **Database**: The name of the database you're connecting to; each MongoDB connection currently only supports a single database.

### UI

To connect Aqueduct to MongoDB, navigate to the Aqueduct integrations page and click on the MongoDB icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import MongoDBConfig

client = aq.Client(s)

conf = MongoDBConfig(
    auth_uri="<URI>"
    database="<DATABASE>",
)
client.connect_integration("my_mongodb_integration", "MongoDB", conf)
```

## Using MongoDB

Once you've creating a MongoDB connection, you can access your data from the Aqueduct SDK

### Reading Data

Aqueduct uses the underlying MongoDB collection API to retrieve data, and you can use the full [`find`](https://pymongo.readthedocs.io/en/stable/api/pymongo/collection.html#pymongo.collection.Collection.find) API with Aqueduct.

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'mongodb'

db = client.integration(INTEGRATION_NAME)

# This is equivalent to SELECT * FROM wines;
wines = db.collection("wines").find()

# The following is equivalent to 
# SELECT review FROM hotel_reviews WHERE nationality=" United Kingdom ";
# in SQL.
reviews = mongo_db.collection("hotel_reviews").find(
  { "nationality": " United Kingdom " },
  projection={"_id": 0, "review": 1},
)
```

### Writing Data

You can write data to MongoDB via Aqueduct with one of two update modes: `replace` and `append`. The former deletes and replaces the existing table in your database, while the latter appends data to an existing table. If the table does not exist or has a mismatched schema, the operation will fail.

```python
# You can call `.save()` on MongoDB the integration itself.
db.save(wines, collection='wines_2', update_mode='replace')

# Or, you can call `.save()` directly on the collection object 
# within the integration.
db.collection('wines_2').save(∑ines, update_mode='replace')
```
