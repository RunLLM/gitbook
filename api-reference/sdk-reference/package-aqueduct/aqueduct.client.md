# Table of Contents

* [aqueduct.client](#aqueduct.client)
  * [global\_config](#aqueduct.client.global_config)
  * [get\_apikey](#aqueduct.client.get_apikey)
  * [Client](#aqueduct.client.Client)
    * [\_\_init\_\_](#aqueduct.client.Client.__init__)
    * [github](#aqueduct.client.Client.github)
    * [create\_param](#aqueduct.client.Client.create_param)
    * [connect\_integration](#aqueduct.client.Client.connect_integration)
    * [connect\_resource](#aqueduct.client.Client.connect_resource)
    * [delete\_integration](#aqueduct.client.Client.delete_integration)
    * [delete\_resource](#aqueduct.client.Client.delete_resource)
    * [list\_integrations](#aqueduct.client.Client.list_integrations)
    * [list\_resources](#aqueduct.client.Client.list_resources)
    * [integration](#aqueduct.client.Client.integration)
    * [resource](#aqueduct.client.Client.resource)
    * [list\_flows](#aqueduct.client.Client.list_flows)
    * [flow](#aqueduct.client.Client.flow)
    * [publish\_flow](#aqueduct.client.Client.publish_flow)
    * [trigger](#aqueduct.client.Client.trigger)
    * [delete\_flow](#aqueduct.client.Client.delete_flow)
    * [describe](#aqueduct.client.Client.describe)

<a id="aqueduct.client"></a>

# aqueduct.client

<a id="aqueduct.client.global_config"></a>

#### global\_config

```python
def global_config(config_dict: Dict[str, Any]) -> None
```

Sets any global configuration variables in the current Aqueduct context.

**Arguments**:

  config_dict:
  A dict from the configuration key to its new value.
  
  Available configuration keys:
  "lazy":
  A boolean indicating whether any new functions will be constructed lazily (True) or eagerly (False).
  "engine":
  The name of the default compute resource to run all functions against.
  This can still be overriden by the `engine` argument in `client.publish_flow()` or
  on the @op spec. To set this to run against the Aqueduct engine, use "aqueduct" (case-insensitive).

<a id="aqueduct.client.get_apikey"></a>

#### get\_apikey

```python
def get_apikey() -> str
```

Get the API key if the server is running locally.

**Returns**:

  The API key.

<a id="aqueduct.client.Client"></a>

## Client Objects

```python
class Client()
```

This class allows users to interact with flows on their Aqueduct cluster.

<a id="aqueduct.client.Client.__init__"></a>

#### \_\_init\_\_

```python
def __init__(api_key: str = "",
             aqueduct_address: str = "http://localhost:8080",
             logging_level: int = logging.WARNING)
```

Creates an instance of Client.

**Arguments**:

  api_key:
  The user unique API key provided by Aqueduct. If no key is
  provided, the client attempts to read the key stored on the
  local server and errors if non exists.
  aqueduct_address:
  The address of the Aqueduct Server service. If no address is
  provided, the client attempts to connect to
  http://localhost:8080.
  logging_level:
  A indication of what level and above to print logs from the sdk.
  Defaults to printing warning and above only. Types defined in: https://docs.python.org/3/howto/logging.html
  

**Returns**:

  A Client instance.

<a id="aqueduct.client.Client.github"></a>

#### github

```python
def github(repo: str, branch: str = "") -> Github
```

Retrieves a Github object connecting to specified repos and branch.

You can only connect to a public repo if you didn't already connect a
Github account to your Aqueduct account.

**Arguments**:

  repo:
  The full github repo URL, e.g. "my_organization/my_repo"
  branch:
  Optional branch name. The default main branch will be used if not specified.
  

**Returns**:

  A github resource object linked to the repo and branch.

<a id="aqueduct.client.Client.create_param"></a>

#### create\_param

```python
def create_param(name: str,
                 default: Any,
                 description: str = "",
                 use_local: bool = False,
                 as_type: Optional[ArtifactType] = None,
                 format: Optional[str] = None) -> BaseArtifact
```

Creates a parameter artifact that can be fed into other operators.

Parameter values are configurable at runtime.

**Arguments**:

  name:
  The name to assign this parameter.
  default:
  The default value to give this parameter, if no value is provided.
  Every parameter must have a default. If we decide to use local data,
  a path to the local data file must be specified.
  description:
  A description of what this parameter represents.
  use_local:
  Whether this parameter uses local data source or not.
  as_type:
  The expected type of the local data. Only supported types are ArtifactType.TABLE and ArtifactType.IMAGE.
  format:
  If local data type is ArtifactType.TABLE, the user has to specify the table format.
  We currently support "json", "csv", and "parquet".

**Returns**:

  A parameter artifact.

<a id="aqueduct.client.Client.connect_integration"></a>

#### connect\_integration

```python
def connect_integration(name: str, service: Union[str, ServiceType],
                        config: Union[Dict[str, str], ResourceConfig]) -> None
```

Deprecated. Use `client.connect_resource()` instead.

<a id="aqueduct.client.Client.connect_resource"></a>

#### connect\_resource

```python
def connect_resource(name: str, service: Union[str, ServiceType],
                     config: Union[Dict[str, str], ResourceConfig]) -> None
```

Connects the Aqueduct server to an resource.

**Arguments**:

  name:
  The name to assign this resource. Will error if an resource with that name
  already exists.
  service:
  The type of resource to connect to.
  config:
  Either a dictionary or an ResourceConnectConfig object that contains the
  configuration credentials needed to connect.

<a id="aqueduct.client.Client.delete_integration"></a>

#### delete\_integration

```python
def delete_integration(name: str) -> None
```

Deprecated. Use `client.delete_resource()` instead.

<a id="aqueduct.client.Client.delete_resource"></a>

#### delete\_resource

```python
def delete_resource(name: str) -> None
```

Deletes the resource from Aqueduct.

**Arguments**:

  name:
  The name of the resource to delete.

<a id="aqueduct.client.Client.list_integrations"></a>

#### list\_integrations

```python
def list_integrations() -> Dict[str, ResourceInfo]
```

Deprecated. Use `client.list_resources()` instead.

<a id="aqueduct.client.Client.list_resources"></a>

#### list\_resources

```python
def list_resources() -> Dict[str, ResourceInfo]
```

Retrieves a dictionary of resources the client can use.

**Returns**:

  A dictionary mapping from resource name to additional info.

<a id="aqueduct.client.Client.integration"></a>

#### integration

```python
def integration(
    name: str
) -> Union[SalesforceResource, S3Resource, GoogleSheetsResource,
           RelationalDBResource, AirflowResource, LambdaResource,
           MongoDBResource, DatabricksResource, SparkResource, AWSResource,
           ECRResource, DynamicK8sResource, GARResource, ]
```

Deprecated. Use `client.resource()` instead.

<a id="aqueduct.client.Client.resource"></a>

#### resource

```python
def resource(
    name: str
) -> Union[SalesforceResource, S3Resource, GoogleSheetsResource,
           RelationalDBResource, AirflowResource, LambdaResource,
           MongoDBResource, DatabricksResource, SparkResource, AWSResource,
           ECRResource, DynamicK8sResource, GARResource, ]
```

Retrieves a connected resource object.

**Arguments**:

  name:
  The name of the resource
  

**Returns**:

  The resource object with the given name.
  

**Raises**:

  InvalidResourceException:
  An error occurred because the cluster is not connected to the
  provided resource or the provided resource is of an
  incompatible type.

<a id="aqueduct.client.Client.list_flows"></a>

#### list\_flows

```python
def list_flows() -> List[Dict[str, str]]
```

Lists the flows that are accessible by this client.

**Returns**:

  A list of flows, each represented as a dictionary providing essential
  information (eg. name, id, etc.), which the user can use to inspect
  the flow further in the UI or SDK.

<a id="aqueduct.client.Client.flow"></a>

#### flow

```python
def flow(flow_identifier: Optional[Union[str, uuid.UUID]] = None,
         flow_id: Optional[Union[str, uuid.UUID]] = None,
         flow_name: Optional[str] = None) -> Flow
```

Fetches a flow corresponding to the given flow id.

**Arguments**:

  flow_identifier:
  Used to identify the flow to fetch from the system.
  Use either the flow name or id as identifier to fetch
  from the system.
  flow_id:
  Used to identify the flow to fetch from the system.
  Between `flow_id` and `flow_name`, at least one must be provided.
  If both are specified, they must correspond to the same flow.
  flow_name:
  Used to identify the flow to fetch from the system.
  
  flow_identifier takes precedence over flow_id or flow_name arguments
  

**Raises**:

  InvalidUserArgumentException:
  If the provided flow id or name does not correspond to a flow the client knows about.

<a id="aqueduct.client.Client.publish_flow"></a>

#### publish\_flow

```python
def publish_flow(name: str,
                 description: str = "",
                 schedule: str = "",
                 engine: Optional[Union[str, DynamicK8sResource]] = None,
                 artifacts: Optional[Union[BaseArtifact,
                                           List[BaseArtifact]]] = None,
                 metrics: Optional[List[NumericArtifact]] = None,
                 checks: Optional[List[BoolArtifact]] = None,
                 k_latest_runs: Optional[int] = None,
                 source_flow: Optional[Union[Flow, str, uuid.UUID]] = None,
                 run_now: Optional[bool] = None,
                 use_local: Optional[bool] = False) -> Flow
```

Uploads and kicks off the given flow in the system.

If a flow already exists with the same name, the existing flow will be updated
to this new state.

The default execution engine of the flow is Aqueduct. In order to specify which
execution engine the flow will be running on, use "config" parameter. Eg:
>>> flow = client.publish_flow(
>>>     name="k8s_example",
>>>     artifacts=[output],
>>>     engine="k8s_resource",
>>> )

**Arguments**:

  name:
  The name of the newly created flow.
  description:
  A description for the new flow.
  schedule:
  A cron expression specifying the cadence that this flow
  will run on. If empty, the flow will only execute manually.
  For example, to run at the top of every hour:
  
  >> schedule = aqueduct.hourly(minute: 0)
  engine:
  The name of the compute resource (eg. "my_lambda_resource") this the flow will
  be computed on.
  artifacts:
  All the artifacts that you care about computing. These artifacts are guaranteed
  to be computed. Additional artifacts may also be computed if they are upstream
  dependencies.
  metrics:
  All the metrics that you would like to compute. If not supplied, we include by default
  all metrics computed on artifacts in the flow.
  checks:
  All the checks that you would like to compute. If not supplied, we will by default
  all checks computed on artifacts in the flow.
  k_latest_runs:
  Number of most-recent runs of this flow that Aqueduct should keep. Runs outside of
  this bound are garbage collected. Defaults to persisting all runs.
  source_flow:
  Used to identify the source flow for this flow. This can be identified
  via an object (Flow), name (str), or id (str or uuid).
  run_now:
  Used to specify if the flow should run immediately at publish time. The default
  behavior is 'True'.
  use_local:
  Must be set if any artifact in the flow is derived from local data.
  

**Raises**:

  InvalidUserArgumentException:
  An invalid combination of parameters was provided.
  InvalidCronStringException:
  An error occurred because the supplied schedule is invalid.
  IncompleteFlowException:
  An error occurred because you are missing some required fields or operators.
  

**Returns**:

  A flow object handle to be used to fetch information about this productionized flow.

<a id="aqueduct.client.Client.trigger"></a>

#### trigger

```python
def trigger(flow_identifier: Optional[Union[str, uuid.UUID]] = None,
            parameters: Optional[Dict[str, Any]] = None,
            flow_id: Optional[Union[str, uuid.UUID]] = None,
            flow_name: Optional[str] = None) -> None
```

Immediately triggers another run of the provided flow.

**Arguments**:

  flow_identifier:
  The uuid or name of the flow to delete.
  parameters:
  A map containing custom values to use for the designated parameters. The mapping
  is expected to be from parameter name to the custom value. These custom values
  are not persisted to the workflow. To actually change the default parameter values
  edit the workflow itself through `client.publish_flow()`.
  flow_id:
  Used to identify the flow to fetch from the system.
  Between `flow_id` and `flow_name`, at least one must be provided.
  If both are specified, they must correspond to the same flow.
  flow_name:
  Used to identify the flow to fetch from the system.
  
  flow_identifier takes precedence over flow_id or flow_name arguments

**Raises**:

  InvalidRequestError:
  An error occurred when attempting to fetch the workflow to
  delete. The provided `flow_id` may be malformed.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.client.Client.delete_flow"></a>

#### delete\_flow

```python
def delete_flow(flow_identifier: Optional[Union[str, uuid.UUID]] = None,
                flow_id: Optional[Union[str, uuid.UUID]] = None,
                flow_name: Optional[str] = None,
                saved_objects_to_delete: Optional[DefaultDict[Union[
                    str, BaseResource], List[SavedObjectUpdate]]] = None,
                force: bool = False) -> None
```

Deletes a flow object.

**Arguments**:

  flow_identifier:
  The id of the flow to delete. Must be name or uuid
  flow_id:
  Used to identify the flow to fetch from the system.
  Between `flow_id` and `flow_name`, at least one must be provided.
  If both are specified, they must correspond to the same flow.
  flow_name:
  Used to identify the flow to fetch from the system.
  
  flow_identifier takes precedence over flow_id or flow_name arguments
  
  saved_objects_to_delete:
  The tables or storage paths to delete grouped by resource name.
  force:
  Force the deletion even though some workflow-written objects in the writes_to_delete argument had UpdateMode=append
  

**Raises**:

  InvalidRequestError:
  An error occurred when attempting to fetch the workflow to
  delete. The provided `flow_id` may be malformed.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.client.Client.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out info about this client in a human-readable format.

