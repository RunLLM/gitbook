# Saving an Artifact

Artifacts can be saved to any storage system that Aqueduct connects to by calling the `.save()` method on the destination integration.&#x20;

The signature of `save()` may be different depending on the integration type. For example, the configuration required to store objects in
Postgres is different from the configuration required by S3.

For more details on how to write Artifacts to Integrations, please see our documentation on using integrations:

{% content-ref url="../integrations/using-integrations/" %}
[using-integrations](../integrations/using-integrations/)
{% endcontent-ref %}
