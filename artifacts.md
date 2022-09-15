---
description: Encapsulating data in Aqueduct
---

# Artifacts

_Artifacts_ are thin wrappers around data objects in Aqueduct. Wrapping data in an Artifact allows Aqueduct to track that artifact across workflow runs help you understand how your data is changing over time.&#x20;

Aqueduct currently supports the following types of _Artifacts_:
* `TableArtifacts`, which are encapsulations of tabular or relational data
* `BoolArtifacts`, which are encapsulations of boolean data
* `NumericArtifacts`, which are encapsulations of numeric data
* `GenericArtifacts`, which are all other data type encapsulations that are not covered above. For supported data types, see [Artifact Data Types](artifacts/artifact-data-types.md)

There are three operations that can be done on artifacts:

* [Saving an Artifact](artifacts/saving-an-artifact.md)
* Transforming an Artifact -- this is done by passing an Artifact as an argument to an [Operator](operators.md)
* Attaching a [Metric or a Check](metrics-and-checks.md) to an artifact -- again, this can be done by passing an Artifact as an argument to the relevant Metric or Check
