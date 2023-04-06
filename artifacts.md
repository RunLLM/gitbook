---
description: Encapsulating data in Aqueduct
---

# Artifacts

_Artifacts_ are thin wrappers around data objects in Aqueduct. Wrapping data in an Artifact allows Aqueduct to track that artifact across workflow runs help you understand how your data is changing over time.

We support the following data types:

* `string`
* `boolean`: This is also the data type returned by Check operators.
* `numeric`: This is also the data type returned by Metric operators.
* `dictionary`
* `tuple`
* `table`
* `json`
* `bytes`
* `image`: Corresponds to PIL.Image type.
* `picklable`: Any object that can be pickled. This is the catch-all type used for all types that are not included above.
* `untyped`: A special placeholder Artifact used to type [lazily-executed operators](operators/lazy-vs-eager-execution.md) — **you should never use the type yourself**.&#x20;

These types are inferred and tracked by the system, no user input is required! Aqueduct uses this type information to provide useful type enforcement like:

* Enforcing artifact type consistency across workflow runs. An artifact with type `t` will continue to have type `t` in future runs.
* Catching errors earlier on, such as writing non-relational data to a relational integration (eg. SQL).

An Artifact can be saved to a storage system by calling `.save` on the integration object in the Aqueduct SDK:

```python
import aqueduct as aq
client = aq.Client()

db = client.integration('aqueduct_demo')
wines = db.sql('SELECT * FROM wine;')

db.save(wines, 'wines_2', 'replace')
```

The specific requirements for each data system will vary based on what data types they accept and what metadata they need. To learn more, visit the documentation page for the data system you're using.
