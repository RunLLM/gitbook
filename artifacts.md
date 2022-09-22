---
description: Encapsulating data in Aqueduct
---

# Artifacts

_Artifacts_ are thin wrappers around data objects in Aqueduct. Wrapping data in an Artifact allows Aqueduct to track that artifact across workflow runs help you understand how your data is changing over time.&#x20;

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
* `untyped`: A special placeholder Artifact used to type the [lazily-executed operators](../operators/lazy-vs-eager-execution.md))

These types are inferred and tracked by the system, no user input is required! Aqueduct uses this type information to provide useful type enforcement like:
* Enforcing artifact type consistency across workflow runs. An artifact with type `t` will continue to have type `t` in future runs.
* Catching errors earlier on, such as writing non-relational data to a relational integration (eg. SQL).


There are three operations that can be done on artifacts:

* [Saving an Artifact](artifacts/saving-an-artifact.md)
* Transforming an Artifact -- this is done by passing an Artifact as an argument to an [Operator](operators.md)
* Attaching a [Metric or a Check](metrics-and-checks.md) to an artifact -- again, this can be done by passing an Artifact as an argument to the relevant Metric or Check
