# Parameters

**A Parameter is simply an argument that is provided to a workflow at runtime.**

See this [**Workflow Parameters Tutorial**](example-workflows/using-parameters.md) notebook for a full walkthrough of the API with concrete examples.

### Creating Parameters

Parameters are identified by their name and _must_ have default values.

To create a parameter explicitly, use `client.create_param(<param_name>, default=<default_val>)`. These parameters can be passed as arguments to operators just like any other artifacts.

To create a parameter implicitly, pass a regular Python object `v` into any operator. Aqueduct will implicitly convert the input into a parameter for you, with the default value `v`. The name of the parameter is the name of its corresponding argument on the function signature.

### Using Parameters

As mentioned above, parameters can be passed into Python functions as regular arguments.

They can also be referenced within SQL queries using the double-bracket syntax. For example, a query `select ... where col_name={{country_names}}` is completely valid, as long as a parameter named "country\_names" exists. If "country\_names" has current value `"'Argentina'"`, the query will implicitly expand into `select ... where col_name='Argentina'`. Note that SQL parameters can only be strings. Also note that when doing interpolation, we don't add quotes around the parameter value for you - it's a direct string substitution.

We provide the following built-in SQL parameters:

* `today`: expands to a string date in the format `'%Y-%m-%d'`. Eg. `SELECT * from hotel_reviews WHERE review_date={{ today }};`

#### Customizing Parameters

For any artifact, you can materialize it with custom parameters fed into `.get()`. For example, for artifact `churn_result` that depends on an upstream parameter named `p`, can be customized with `churn_result.get(parameters={"p": <new val>})`

{% hint style="info" %}
Types are also enforced for parameters. You cannot provide a new, custom parameter that has a different type.
{% endhint %}

Similarly, for any flow that has been published with parameters, you can always trigger a new flow run with `client.trigger(parameters={<param_name>: <new_param_val>}`. Note that triggering a workflow with non-default parameters is a one-off operation. It does not change the default parameters for future runs of the workflow. That is to say, if a workflow is running on a schedule, every scheduled run will always use the same default parameters. You can always change the default value of a published parameter by re-running the workflow with the new default value.
