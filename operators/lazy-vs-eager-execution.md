# Lazy vs Eager Execution

The default mode of operator execution is `eager`. This means that whenever an operator is called, it will immediately run. 
This is just like executing a regular Python function, and allows us to provide useful features like type enforcement after
the first run.

Alternatively, you may execute an operator in `lazy` mode. This means that we will only run that operator once the entire flow is published. 
This removes any pre-publish guardrails that `eager` mode provides. However, for workflows processing large amounts of data, this can greatly 
speed up the workflow creation process, since we no longer need to run the flow before publishing it.

These modes also apply for [Metric & Checks](../metrics-and-checks.md). You can configure laziness on a global or operator-level granularity. See below:

```python
import aqueduct as aq
import pandas as pd
from aqueduct import Client, op

client = Client("<API_KEY>", "<SERVER_ADDRESS>")

@op
def predict_churn(df):
    ...

input_df: pd.DataFrame = ...

# In the default eager execution mode, we immediately compute the churn artifact.
# If there was a bug in `predict_churn()`, we would find out here.
churn = predict_churn(input_df)

# We can use the `.lazy()` attribute to have `predict_churn` only execute when the entire flow is published.
# If there was a bug in `predict_churn()`, we would not find out until the uploaded flow executes.
churn = predict_churn.lazy(input_df)
client.publish_flow("Flow with a Lazily Executed Operator", artifacts=[churn])

# We can also decide that we want all our operators to be lazy from now on.
aq.global_config({"lazy" : True})
churn = predict_churn(input_df)
client.publish_flow("Another Flow with a Lazily Executed Operator", artifacts=[churn])

```

{% hint style="info" %}
You may mix lazily and eagerly defined operators in the same flow. However, if an eager operator depends on a lazy operator, 
that lazy operator will ultimately have to be executed, and will gain all the type enforcement benefits of an eagerly executed
operator!
{% endhint %}

