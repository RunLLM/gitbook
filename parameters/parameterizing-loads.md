# Parameterizing Loads

Only SQL queries to relational database resource can be parameterized. 

### Parameterizing SQL Queries
We use the Postgres-inspired $1, $2, etc. placeholder syntax. 
The number after the dollar sign indicates which parameter to replace the placeholder with. This allows a parameter to be reused 
multiple times within the same query and is similar to how traditional RDBMS interfaces work.

{% hint style="warning" %}
A parameter embedded in a SQL query must be of string type.
{% endhint %}

```python
table_name_param = client.create_param("table_name", default="hotel_reviews")

# TThe value of `table_name_param` is substituted into $1.
db = client.resource("Demo")
table_artifact = db.sql(query="select * from $1", parameters=[table_name_param])

# This returns the result of `select * from hotel_reviews;`
table_artifact.get()

# This returns the result of `select * from tips;`
table_artifact.get(parameters={"table_name": "tips"})
```

{% hint style="info" %}
We perform a direct substitute of the parameter string into the $-placeholder token, so it is still up to you to construct your SQL query appropriately. For example, we do not automatically add quotes around column names for you.
{% endhint %}

### Using Built-in SQL Parameters

We also provide the following built-in SQL parameters:

* `today`: expands to a string date in the format `'%Y-%m-%d'`. Eg. `SELECT * from hotel_reviews WHERE review_date={{today}};`

In order to use the built-in parameters, you must use the `{{ }}` syntax. For example:

```python
reviews_before_today = db.sql("select * from hotel_reviews where review_date < {{ today }}")
reviews_before_today.get()
```
