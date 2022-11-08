# Table of Contents

* [aqueduct.aqueduct\_client](#aqueduct.aqueduct_client)
  * [get\_apikey](#aqueduct.aqueduct_client.get_apikey)
  * [infer\_requirements](#aqueduct.aqueduct_client.infer_requirements)
  * [Client](#aqueduct.aqueduct_client.Client)
    * [\_\_init\_\_](#aqueduct.aqueduct_client.Client.__init__)
    * [github](#aqueduct.aqueduct_client.Client.github)
    * [create\_param](#aqueduct.aqueduct_client.Client.create_param)
    * [list\_integrations](#aqueduct.aqueduct_client.Client.list_integrations)
    * [integration](#aqueduct.aqueduct_client.Client.integration)
    * [list\_flows](#aqueduct.aqueduct_client.Client.list_flows)
    * [flow](#aqueduct.aqueduct_client.Client.flow)
    * [publish\_flow](#aqueduct.aqueduct_client.Client.publish_flow)
    * [trigger](#aqueduct.aqueduct_client.Client.trigger)
    * [delete\_flow](#aqueduct.aqueduct_client.Client.delete_flow)
    * [describe](#aqueduct.aqueduct_client.Client.describe)

<a id="aqueduct.aqueduct_client"></a>

# aqueduct.aqueduct\_client

<a id="aqueduct.aqueduct_client.get_apikey"></a>

#### get\_apikey

```python
def get_apikey() -> str
```

Get the API key if the server is running locally.

**Returns**:

  The API key.

<a id="aqueduct.aqueduct_client.infer_requirements"></a>

#### infer\_requirements

```python
def infer_requirements() -> List[str]
```

Obtains the list of pip requirements specifiers from the current python environment using `pip freeze`.

**Returns**:

  A list, for example, ["transformers==4.21.0", "numpy==1.22.4"].

<a id="aqueduct.aqueduct_client.Client"></a>

## Client Objects

```python
class Client()
```

This class allows users to interact with flows on their Aqueduct cluster.

<a id="aqueduct.aqueduct_client.Client.__init__"></a>

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

<a id="aqueduct.aqueduct_client.Client.github"></a>

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

  A github integration object linked to the repo and branch.

<a id="aqueduct.aqueduct_client.Client.create_param"></a>

#### create\_param

```python
def create_param(name: str,
                 default: Any,
                 description: str = "") -> BaseArtifact
```

Creates a parameter artifact that can be fed into other operators.

Parameter values are configurable at runtime.

**Arguments**:

  name:
  The name to assign this parameter.
  default:
  The default value to give this parameter, if no value is provided.
  Every parameter must have a default.
  description:
  A description of what this parameter represents.
  

**Returns**:

  A parameter artifact.

<a id="aqueduct.aqueduct_client.Client.list_integrations"></a>

#### list\_integrations

```python
def list_integrations() -> Dict[str, IntegrationInfo]
```

Retrieves a dictionary of integrations the client can use.

**Returns**:

  A dictionary mapping from integration name to additional info.

<a id="aqueduct.aqueduct_client.Client.integration"></a>

#### integration

```python
def integration(
    name: str
) -> Union[SalesforceIntegration, S3Integration, GoogleSheetsIntegration,
           RelationalDBIntegration, AirflowIntegration, K8sIntegration,
           LambdaIntegration, MongoDBIntegration, ]
```

Retrieves a connected integration object.

**Arguments**:

  name:
  The name of the integration
  

**Returns**:

  The integration object with the given name.
  

**Raises**:

  InvalidIntegrationException:
  An error occurred because the cluster is not connected to the
  provided integration or the provided integration is of an
  incompatible type.

<a id="aqueduct.aqueduct_client.Client.list_flows"></a>

#### list\_flows

```python
def list_flows() -> List[Dict[str, str]]
```

Lists the flows that are accessible by this client.

**Returns**:

  A list of flows, each represented as a dictionary providing essential
  information (eg. name, id, etc.), which the user can use to inspect
  the flow further in the UI or SDK.

<a id="aqueduct.aqueduct_client.Client.flow"></a>

#### flow

```python
def flow(flow_id: Union[str, uuid.UUID]) -> Flow
```

Fetches a flow corresponding to the given flow id.

**Arguments**:

  flow_id:
  Used to identify the flow to fetch from the system.
  

**Raises**:

  InvalidUserArgumentException:
  If the provided flow id does not correspond to a flow the client knows about.

<a id="aqueduct.aqueduct_client.Client.publish_flow"></a>

#### publish\_flow

```python
def publish_flow(name: str,
                 description: str = "",
                 schedule: str = "",
                 engine: Optional[str] = None,
                 artifacts: Optional[Union[BaseArtifact,
                                           List[BaseArtifact]]] = None,
                 metrics: Optional[List[NumericArtifact]] = None,
                 checks: Optional[List[BoolArtifact]] = None,
                 k_latest_runs: Optional[int] = None,
                 config: Optional[FlowConfig] = None) -> Flow
```

Uploads and kicks off the given flow in the system.

If a flow already exists with the same name, the existing flow will be updated
to this new state.

The default execution engine of the flow is Aqueduct. In order to specify which
execution engine the flow will be running on, use "config" parameter. Eg:
>>> k8s_integration = client.integration("k8s_integration")
>>> flow = client.publish_flow(
>>>     name = "k8s_example",
>>>     artifacts = [output],
>>>     config = FlowConfig(engine=k8s_integration),
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
  The name of the compute integration (eg. "my_lambda_integration") this the flow will
  be computed on.
  artifacts:
  All the artifacts that you care about computing. These artifacts are guaranteed
  to be computed. Additional artifacts may also be computed if they are upstream
  dependencies.
  metrics:
  All the metrics that you would like to compute. If not supplied, we will implicitly
  include all metrics computed on artifacts in the flow.
  checks:
  All the checks that you would like to compute. If not supplied, we will implicitly
  include all checks computed on artifacts in the flow.
  k_latest_runs:
  Number of most-recent runs of this flow that Aqueduct should keep. Runs outside of
  this bound are garbage collected. Defaults to persisting all runs.
  config:
  This field will be deprecated. Please use `engine` and `k_latest_runs` instead.
  
  An optional set of config fields for this flow.
  - engine: Specify where this flow should run with one of your connected integrations.
  - k_latest_runs: Number of most-recent runs of this flow that Aqueduct should store.
  Runs outside of this bound are deleted. Defaults to persisting all runs.
  

**Raises**:

  InvalidUserArgumentException:
  An invalid combination of parameters was provided.
  InvalidCronStringException:
  An error occurred because the supplied schedule is invalid.
  IncompleteFlowException:
  An error occurred because you are missing some required fields or operators.
  

**Returns**:

  A flow object handle to be used to fetch information about this productionized flow.

<a id="aqueduct.aqueduct_client.Client.trigger"></a>

#### trigger

```python
def trigger(flow_id: Union[str, uuid.UUID],
            parameters: Optional[Dict[str, Any]] = None) -> None
```

Immediately triggers another run of the provided flow.

**Arguments**:

  flow_id:
  The id of the workflow to delete (not the name)
  parameters:
  A map containing custom values to use for the designated parameters. The mapping
  is expected to be from parameter name to the custom value. These custom values
  are not persisted to the workflow. To actually change the default parameter values
  edit the workflow itself through `client.publish_flow()`.
  

**Raises**:

  InvalidRequestError:
  An error occurred when attempting to fetch the workflow to
  delete. The provided `flow_id` may be malformed.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.aqueduct_client.Client.delete_flow"></a>

#### delete\_flow

```python
def delete_flow(flow_id: Union[str, uuid.UUID],
                saved_objects_to_delete: Optional[DefaultDict[Union[
                    str, Integration], List[SavedObjectUpdate]]] = None,
                force: bool = False) -> None
```

Deletes a flow object.

**Arguments**:

  flow_id:
  The id of the workflow to delete (not the name)
  saved_objects_to_delete:
  The tables or storage paths to delete grouped by integration name.
  force:
  Force the deletion even though some workflow-written objects in the writes_to_delete argument had UpdateMode=append
  

**Raises**:

  InvalidRequestError:
  An error occurred when attempting to fetch the workflow to
  delete. The provided `flow_id` may be malformed.
  InternalServerError:
  An unexpected error occurred within the Aqueduct cluster.

<a id="aqueduct.aqueduct_client.Client.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out info about this client in a human-readable format.

