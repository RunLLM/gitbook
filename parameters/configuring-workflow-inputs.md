# Configuring Workflow Inputs

After publishing a parameterized workflow, you can trigger new runs of that workflow with different parameters. 
This allows you to easily re-run the same workflow with different parameters, without having to re-publish the workflow.

{% hint style="info" %}
Triggering a workflow with non-default parameters is a one-off operation. It does not change the default parameters for future runs of the workflow. That is to say, if a workflow is running on a schedule, every scheduled run will continue to use the same default parameters. You can always change the default value of a published parameter by re-running the workflow with the new default value.
{% endhint %}

Assuming we have publish a workflow with a string parameter named "country":

```python
churn = predict_churn(reviews, country_param)
flow = client.publish_flow("Churn Prediction", artifacts=[churn])

# Sleep for 10 seconds to wait for flow to finish.
import time
time.sleep(10) 

# This will trigger a new run of the already published flow "Churn Prediction",
# but with the country parameter value set to "China".
client.trigger(flow.id(), parameters={"country": "China"})
```

{% hint style="info" %}
Types are also enforced on parameters. You cannot set a new value that is of a different type as the default value.
{% endhint %}