# Databricks

{% hint style="info" %}
Before starting, please make sure you've added a Databricsk integration to Aqueduct (see [adding-an-integration](../adding-an-integration/connecting-to-databricks.md "mention")).&#x20;
{% endhint %}

### Running a Workflow

The recommended way to use the Databricks integration is to [set the global config](./compute-integrations.md "mention") to use the Databricks integration and lazy execution mode. 

```python
aq.global_config({'engine': '<databricks_integration name>', 'lazy': True})
```

We spin up an on-demand cluster for each workflow; pre-setting lazy execution allows us to avoid unnecessary spin-up of clusters as we build our workflow. See [lazy-vs.-eager-execution.md](../../operators/lazy-vs.-eager-execution.md "mention") for more information about execution mode.

After setting this mode, we are ready to launch workflows on Databricks!