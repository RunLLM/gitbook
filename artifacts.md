---
description: Encapsulating data in Aqueduct
---

# Artifacts

_Artifacts_ are thin wrappers around data objects in Aqueduct. Wrapping data in an Artifact allows Aqueduct to track that artifact across workflow runs help you understand how your data is changing over time.&#x20;

All _Artifacts_ that are not of Type `GenericArtifact` are already aware of the underlying data types they hold. For example, the `BooleanArtifact` holds a `boolean`. A `GenericArtifact` holds a generic data type that is not encapsulated by the other explicit artifacts. A full list of Artifact data types within the system can be found below:

* `string`
* `boolean`
* `numeric`
* `dictionary`
* `tuple`
* `table`
* `json`
* `bytes`
* `image`: Corresponds to PIL.Image.Image type
* `picklable`: An object that can be pickled
* `untyped`: An unknown Artifact type for results of [lazy execution](../operators/lazy-vs-eager-execution.md))

There are three operations that can be done on artifacts:

* [Saving an Artifact](artifacts/saving-an-artifact.md)
* Transforming an Artifact -- this is done by passing an Artifact as an argument to an [Operator](operators.md)
* Attaching a [Metric or a Check](metrics-and-checks.md) to an artifact -- again, this can be done by passing an Artifact as an argument to the relevant Metric or Check
