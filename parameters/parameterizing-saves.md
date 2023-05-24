# Parameterizing Saves

Currently, only saves to relational databases and S3 buckets can be parameterized.

### Parameterizing Relational Database Saves

The table name to which a table artifact is saved in a relational database can be represented as a string parameter.

```python
# The table to be saved.
table_artifact = compute_output_table(inputs)

# The demo database is a relational SQLite database.
db = client.resource("Demo")

table_name_param = client.create_param("table_name", default="output_table")
db.save(table_artifact, table_name_param, update_mode="replace")

# When run, this flow will save the table to `output_table`, the default parameter value.
flow = client.publish_flow("Parameterized Table Name Demo", artifacts=table_name_param)

# You can trigger this published flow to save to a different table name ("another_table_name").
client.trigger(flow.id(), parameters={"table_name": "another_table_name"})
```

{& hint style="info" %}
To publish a flow without running it immediately, set `run_now=False` in `client.publish_flow()`.
{& endhint %}

### Parameterizing S3 Saves

The filepath to which an artifact is saved in an S3 bucket can be parameterized using the '{' and '}' syntax.

In the following example, we publish a flow with parameters that control which subdirectory and file an image
is saved to in the S3 bucket.

```python
# The artifact to save
img_artifact = ...

s3_bucket = client.resource("S3 Resource")

client.create_param("dirname", default="images")
client.create_param("filename", default="image_1.jpg")
s3_bucket.save(img_artifact, "{dirname}/{filename}")

# When run, this flow will save the image to `images/image_1.jpg`, the default parameter value.
flow = client.publish_flow("Parameterized S3 Save Demo", artifacts=img_artifact)

# You can trigger this published flow to save to a different subdirectory and filename.
# In this case, we just decide to save it to a different filename.
client.trigger(flow.id(), parameters={"filename": "image_2.jpg"})
```

{& hint style="warning" %}
Support for deleting saved S3 objects that were written using parameterized filpaths does not yet exist. This means that 
deleting all S3 objects with `client.delete_flow(flow.list_saved_objects())` from the above case will error out.
{& endhint %}

