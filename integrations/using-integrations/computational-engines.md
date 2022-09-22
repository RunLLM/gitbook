# Computational Engines

{% hint style="info" %}
Before starting, make sure you've added your Computational Engines as an integration to Aqueduct (see [adding-an-integration](../adding-an-integration/ "mention")).
{% endhint %}
Currently, Aqueduct supports computational engines including Airflow, Kubernetes, and Lambda. First, we're going to get a connection to your computational engines(we are using Kubernetes as an example) by calling `.integration()` on the Aqueduct Client.&#x20;

```python
k8s_integration = client.integration('k8s_integration')

```

### Publishing Workflow from the Connected Engines
See [create-a-workflow](../../workflows/creating-a-workflow "mention")
