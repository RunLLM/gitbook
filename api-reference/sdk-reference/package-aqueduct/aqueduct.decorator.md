# Table of Contents

* [aqueduct.decorator](#aqueduct.decorator)
  * [wrap\_spec](#aqueduct.decorator.wrap_spec)
  * [op](#aqueduct.decorator.op)
  * [metric](#aqueduct.decorator.metric)
  * [check](#aqueduct.decorator.check)
  * [to\_operator](#aqueduct.decorator.to_operator)

<a id="aqueduct.decorator"></a>

# aqueduct.decorator

<a id="aqueduct.decorator.wrap_spec"></a>

#### wrap\_spec

```python
def wrap_spec(
    spec: OperatorSpec,
    *input_artifacts: BaseArtifact,
    op_name: str,
    output_artifact_type_hints: List[ArtifactType],
    output_artifact_names: Optional[List[str]] = None,
    description: str = "",
    execution_mode: ExecutionMode = ExecutionMode.EAGER
) -> Union[BaseArtifact, List[BaseArtifact]]
```

Applies a python function to existing artifacts.
The function must be named predict() on a class named "Function",
in a file named "model.py":

>>> class Function:
>>>     def predict(self, *args):
>>>         ...

**Arguments**:

  spec:
  The spec of the operator.
  *input_artifacts:
  All the artifacts that will serve as input to the python function.
  The function must have the same number of parameters as input
  artifacts.
  op_name:
  The name of the operator that generated this artifact.
  output_artifact_names:
  If set, provides the custom output artifact names.
  output_artifact_type_hints:
  The artifact types that the function is expected to output, in the correct order.
  description:
  The description for this operator.
  

**Returns**:

  A list of artifacts, representing the outputs of the function.

<a id="aqueduct.decorator.op"></a>

#### op

```python
def op(
    name: Optional[Union[str, UserFunction]] = None,
    description: Optional[str] = None,
    engine: Optional[str] = None,
    file_dependencies: Optional[List[str]] = None,
    requirements: Optional[Union[str, List[str]]] = None,
    num_outputs: Optional[int] = None,
    outputs: Optional[List[str]] = None,
    resources: Optional[Dict[str, Any]] = None
) -> Union[DecoratedFunction, OutputArtifactsFunction]
```

Decorator that converts regular python functions into an operator.

Calling the decorated function returns an Artifact. The decorated function
can take any number of artifact inputs.

To run the wrapped code locally, without Aqueduct, use the `local` attribute. Eg:
>>> compute_recommendations.local(customer_profiles, recent_clicks)

**Arguments**:

  name:
  Operator name. Defaults to the function name if not provided (or is of a non-string type).
  description:
  A description for the operator.
  engine:
  The name of the compute integration this operator will run on. Defaults to the Aqueduct engine.
  file_dependencies:
  A list of relative paths to files that the function needs to access.
  Python classes/methods already imported within the function's file
  need not be included.
  requirements:
  Defines the python package requirements that this operator will run with.
  Can be either a path to the requirements.txt file or a list of pip requirements specifiers.
  (eg. ["transformers==4.21.0", "numpy==1.22.4"]. If not supplied, we'll first
  look for a `requirements.txt` file in the same directory as the decorated function
  and install those. Otherwise, we'll attempt to infer the requirements with
  `pip freeze`.
  num_outputs:
  The number of outputs the decorated function is expected to return.
  Will fail at runtime if a different number of outputs is returned by the function.
  outputs:
  The name to assign the output artifacts of for this operator. The number of names provided
  must match the number of return values of the decorated function. If not set, the artifact
  names will default to "<op_name> artifact <optional counter>".
  resources:
  A dictionary containing the custom resource configurations that this operator will run with.
  These configurations are guaranteed to be followed, we will not silently ignore any of them.
  If a resource configuration is unsupported by a particular execution engine, we will fail at
  execution time. The supported keys are:
  
  "num_cpus" (int):
  The number of cpus that this operator will run with. This operator will execute with *exactly*
  this number of cpus. If not enough cpus are available, operator execution will fail.
  "memory" (int, str):
  The amount of memory this operator will run with. This operator will execute with *exactly*
  this amount of memory. If not enough memory is available, operator execution will fail.
  
  If an integer value is supplied, the memory unit is assumed to be MB. If a string is supplied,
  a suffix indicating the memory unit must be supplied. Supported memory units are "MB" and "GB",
  case-insensitive.
  
  For example, the following values are valid: 100, "100MB", "1GB", "100mb", "1gb".
  "gpu_resource_name" (str):
  Name of the gpu resource to use (only applicable for Kubernetes engine).
  
  For example, the following value is valid: "nvidia.com/gpu".
  "cuda_version" (str):
  Version of CUDA to use with GPU (only applicable for Kubernetes engine). The currently supported
  values are "11.4.1" and "11.8.0".
  

**Examples**:

  The op name is inferred from the function name. The description is pulled from the function
  docstring or can be explicitly set in the decorator.
  
  >>> @op
  ... def compute_recommendations(customer_profiles, recent_clicks):
  ...     return recommendations
  >>> customer_profiles = db.sql("SELECT * from user_profile", db="postgres")
  >>> recent_clicks = db.sql("SELECT * recent_clicks", db="google_analytics/shopping")
  >>> recommendations = compute_recommendations(customer_profiles, recent_clicks)
  
  `recommendations` is an Artifact representing the result of `compute_recommendations()`.
  
  >>> recommendations.get()

<a id="aqueduct.decorator.metric"></a>

#### metric

```python
def metric(
    name: Optional[Union[str, MetricFunction]] = None,
    description: Optional[str] = None,
    file_dependencies: Optional[List[str]] = None,
    requirements: Optional[Union[str, List[str]]] = None,
    output: Optional[str] = None,
    engine: Optional[str] = None,
    resources: Optional[Dict[str, Any]] = None
) -> Union[DecoratedMetricFunction, OutputArtifactFunction]
```

Decorator that converts regular python functions into a metric.

Calling the decorated function returns a NumericArtifact. The decorated function
can take any number of artifact inputs.

The requirements.txt file in the current directory is used, if it exists.

To run the wrapped code locally, without Aqueduct, use the `local` attribute. Eg:
>>> compute_recommendations.local(customer_profiles, recent_clicks)

**Arguments**:

  name:
  Operator name. Defaults to the function name if not provided (or is of a non-string type).
  description:
  A description for the metric.
  file_dependencies:
  A list of relative paths to files that the function needs to access.
  Python classes/methods already imported within the function's file
  need not be included.
  requirements:
  Defines the python package requirements that this operator will run with.
  Can be either a path to the requirements.txt file or a list of pip requirements specifiers.
  (eg. ["transformers==4.21.0", "numpy==1.22.4"]. If not supplied, we'll first
  look for a `requirements.txt` file in the same directory as the decorated function
  and install those. Otherwise, we'll attempt to infer the requirements with
  `pip freeze`.
  output:
  An optional custom name for the output metric artifact. Otherwise, the default naming scheme
  will be used.
  engine:
  The name of the compute integration this operator will run on.
  resources:
  A dictionary containing the custom resource configurations that this operator will run with.
  These configurations are guaranteed to be followed, we will not silently ignore any of them.
  If a resource configuration is unsupported by a particular execution engine, we will fail at
  execution time. The supported keys are:
  
  "num_cpus" (int):
  The number of cpus that this operator will run with. This operator will execute with *exactly*
  this number of cpus. If not enough cpus are available, operator execution will fail.
  "memory" (int, str):
  The amount of memory this operator will run with. This operator will execute with *exactly*
  this amount of memory. If not enough memory is available, operator execution will fail.
  
  If an integer value is supplied, the memory unit is assumed to be MB. If a string is supplied,
  a suffix indicating the memory unit must be supplied. Supported memory units are "MB" and "GB",
  case-insensitive.
  
  For example, the following values are valid: 100, "100MB", "1GB", "100mb", "1gb".
  "gpu_resource_name" (str):
  Name of the gpu resource to use (only applicable for Kubernetes engine).
  

**Examples**:

  The metric name is inferred from the function name. The description is pulled from the function
  docstring or can be explicitly set in the decorator.
  
  >>> @metric()
  ... def avg_churn(churn_table):
  ...     return churn_table['pred_churn'].mean()
  >>> churn_table = db.sql("SELECT * from churn_table")
  >>> churn_metric = avg_churn(churn_table)
  
  `churn_metric` is a NumericArtifact representing the result of `avg_churn()`.
  
  >>> churn_metric.get()

<a id="aqueduct.decorator.check"></a>

#### check

```python
def check(
    name: Optional[Union[str, CheckFunction]] = None,
    description: Optional[str] = None,
    severity: CheckSeverity = CheckSeverity.WARNING,
    file_dependencies: Optional[List[str]] = None,
    requirements: Optional[Union[str, List[str]]] = None,
    output: Optional[str] = None,
    engine: Optional[str] = None,
    resources: Optional[Dict[str, Any]] = None
) -> Union[DecoratedCheckFunction, OutputArtifactFunction]
```

Decorator that converts a regular python function into a check.

Calling the decorated function returns a BoolArtifact. The decorated python function
can have any number of artifact inputs.

A check can be set with either WARNING or ERROR severity. A failing check with ERROR severity
will fail the workflow when run in our system.

To run the wrapped code locally, without Aqueduct, use the `local` attribute. Eg:
>>> compute_recommendations.local(customer_profiles, recent_clicks)

**Arguments**:

  name:
  Operator name. Defaults to the function name if not provided (or is of a non-string type).
  description:
  A description for the check.
  severity:
  The severity level of the check if it fails.
  file_dependencies:
  A list of relative paths to files that the function needs to access.
  Python classes/methods already imported within the function's file
  need not be included.
  requirements:
  Defines the python package requirements that this operator will run with.
  Can be either a path to the requirements.txt file or a list of pip requirements specifiers.
  (eg. ["transformers==4.21.0", "numpy==1.22.4"]. If not supplied, we'll first
  look for a `requirements.txt` file in the same directory as the decorated function
  and install those. Otherwise, we'll attempt to infer the requirements with
  `pip freeze`.
  output:
  An optional custom name for the output metric artifact. Otherwise, the default naming scheme
  will be used.
  engine:
  The name of the compute integration this operator will run on.
  resources:
  A dictionary containing the custom resource configurations that this operator will run with.
  These configurations are guaranteed to be followed, we will not silently ignore any of them.
  If a resource configuration is unsupported by a particular execution engine, we will fail at
  execution time. The supported keys are:
  
  "num_cpus" (int):
  The number of cpus that this operator will run with. This operator will execute with *exactly*
  this number of cpus. If not enough cpus are available, operator execution will fail.
  "memory" (int, str):
  The amount of memory this operator will run with. This operator will execute with *exactly*
  this amount of memory. If not enough memory is available, operator execution will fail.
  
  If an integer value is supplied, the memory unit is assumed to be MB. If a string is supplied,
  a suffix indicating the memory unit must be supplied. Supported memory units are "MB" and "GB",
  case-insensitive.
  
  For example, the following values are valid: 100, "100MB", "1GB", "100mb", "1gb".
  "gpu_resource_name" (str):
  Name of the gpu resource to use (only applicable for Kubernetes engine).
  

**Examples**:

  The check name is inferred from the function name. The description is pulled from the function
  docstring or can be explicitly set in the decorator.
  
  >>> @check(
  ... description="The average predicted churn is below 0.1",
  ... severity=CheckSeverity.ERROR,
  ... )
  ... def avg_churn_is_low(churn_table):
  ...     return churn_table['pred_churn'].mean() < 0.1
  >>> churn_is_low_check = avg_churn_is_low(churn_table_artifact)
  
  `churn_is_low_check` is a BoolArtifact representing the result of `avg_churn_is_low()`.
  
  >>> churn_is_low_check.get()

<a id="aqueduct.decorator.to_operator"></a>

#### to\_operator

```python
def to_operator(
    func: UserFunction,
    name: Optional[str] = None,
    description: Optional[str] = None,
    file_dependencies: Optional[List[str]] = None,
    requirements: Optional[Union[str, List[str]]] = None
) -> OutputArtifactsFunction
```

Convert a function that returns a dataframe into an Aqueduct operator.

**Arguments**:

  func:
  the python function that is to be converted into operator.
  name:
  Operator name.
  description:
  A description for the operator.
  file_dependencies:
  A list of relative paths to files that the function needs to access.
  Python classes/methods already imported within the function's file
  need not be included.
  requirements:
  Defines the python package requirements that this operator will run with.
  Can be either a path to the requirements.txt file or a list of pip requirements specifiers.
  (eg. ["transformers==4.21.0", "numpy==1.22.4"]. If not supplied, we'll first
  look for a `requirements.txt` file in the same directory as the decorated function
  and install those. Otherwise, we'll attempt to infer the requirements with
  `pip freeze`.

**Returns**:

  An Aqueduct operator that can be used just like any decorated operator.

