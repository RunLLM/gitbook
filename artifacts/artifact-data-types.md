# Artifact Data Types

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
