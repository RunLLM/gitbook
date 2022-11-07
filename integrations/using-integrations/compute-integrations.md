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

Once we have an integration, we can use it in one of two ways:

1. We can specify the integration to be used for any workflow that is created in this notebook, using Aqueduct's `global_config`.&#x20;
2. We can use that integration to create a specific workflow that runs on that integration.

#### Setting the global configuration

`aq.global_config` allows us to set the configuration for this notebook. Here, we will set the default integration to be used in the notebook and point it to our Kubernetes integration. Now, any workflow we create from this notebook will use our Kubernetes integration.

```python
aq.global_config({'engine': k8s})
```

Using `global_config`, you can also specify [lazy-vs.-eager-execution.md](../../operators/lazy-vs.-eager-execution.md "mention").

#### Creating a single workflow

Assuming we've defined a workflow, we can publish that workflow to run on a compute integration using the `engine` parameter in `publish_flow`:

<pre class="language-python"><code class="lang-python"><strong>flow = client.publish_flow(name='average_acidity', 
</strong>                           artifacts=[output_artf],
                           engine='lambda_us_east_2')</code></pre>

The SDK will now publish the workflow to the Aqueduct server, and in this case, the workflow will automatically be deployed onto AWS Lambda. Note that you can use either the name of the integration or use `client.integration` to retrieve a pointer to the integration; the Aqueduct SDK supports either pattern.\
