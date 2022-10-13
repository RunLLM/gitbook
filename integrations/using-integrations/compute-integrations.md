# Compute Integrations

{% hint style="info" %}
Before starting, please make sure you've added a compute integration to Aqueduct (see [adding-an-integration](../adding-an-integration/ "mention")).&#x20;
{% endhint %}

Aqueduct supports Kubernetes, Apache Airflow, and AWS Lambda as compute backends. First, we will load an Aqueduct connection to your integration:

```python
import aqueduct as aq
client = aq.Client()

k8s = client.integration('k8s_integration')
```

Assuming we've defined a workflow using the Aqueduct API, we can publish that workflow to run on a compute integration using the `config` parameter in `publish_flow`:

<pre class="language-python"><code class="lang-python"><strong>flow = client.publish_flow(name='average_acidity', 
</strong>                           artifacts=[output_artf],
                           config=aq.FlowConfig(engine=k8s))</code></pre>

The `FlowConfig` object allows us to specify that we would like to execute this workflow on the engine specified -- in this case, our Kubernetes cluster. The SDK will now publish the workflow to the Aqueduct server and automatically deploy it onto your Kubernetes cluster.\
