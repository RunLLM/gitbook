# SQL Systems

Aqueduct comes with connectors to the following SQL databases:

* [snowflake.md](snowflake.md "mention")
* [postgres.md](postgres.md "mention")
* [mysql.md](mysql.md "mention")
* [mariadb.md](mariadb.md "mention")
* [sqlite.md](sqlite.md "mention")
* [aws-athena.md](aws-athena.md "mention")
* [aws-redshift.md](aws-redshift.md "mention")
* [google-bigquery.md](google-bigquery.md "mention")

### Chaining SQL queries

Aqueduct supports chaining SQL queries together, allowing you to break down complex queries into smaller pieces for better readability.

To do so, simply call `.sql()` with a list of strings. In the query chain, you can use the special placeholder `$` to refer to the previous query in. The output of the last query will be returned as the output artifact of this execution.

```python
import aqueduct aq aq 
client = aq.Client()


db = client.resource('aqueduct_demo')
customers = db.sql([
  'SELECT * FROM customers WHERE company_size < 50;',
  # `$` represents the output of the above query.
  'SELECT * FROM $ WHERE n_workflows < 10;',
  # The output of this query will be returned.
  'SELECT cust_id AS id, n_data_eng FROM $;',
])
```

This example returns the `cust_id` and `n_data_eng` of all customers with `company_size < 50` and `n_workflows < 10`. Under the hood, we compile this chain to a single query using a `WITH` clause, and the compiled query is executed:

```sql
WITH
  tmp_0 AS (SELECT * FROM customers WHERE company_size < 50),
  tmp_1 AS (SELECT * FROM tmp_0 WHERE n_workflows < 10)
SELECT cust_id AS id, n_data_eng FROM tmp_1;
```
