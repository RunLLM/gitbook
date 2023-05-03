---
description: Aqueduct comes with simple defaults but is highly customizable.
---

# Configuring Aqueduct

The Aqueduct server is designed to run seamlessly out of the box without any configuration necessary. However, depending on your environment, you may want to configure one of many options that Aqueduct provides.&#x20;

## Configuring Resources 

Every data system Aqueduct connects to has different required drivers. To ensure that Aqueduct can connect correctly, please run the following command for any data system you intend to use:

```bash
aqueduct install [system]
# aqueduct install postgres
# aqueduct install mysql
# aqueduct install s3
# ...
```

Once you've installed the requisite drivers, you can connect resources from the Aqueduct UI or SDK. See [resources](../resources/ "mention") for more details on how to configure each system.

## External Access to Aqueduct

By default, the Aqueduct server is only accessible from the machine it's running on. If you would like to access Aqueduct from outside that machine (e.g., if you're running on an EC2 server and want to access Aqueduct from your laptop), you must include the `--expose` flag when running your server:

```bash
aqueduct start --expose
```

**Importantly, even when run with external access enabled, all interactions on the UI and via the SDK are protected with the Aqueduct API key.**

## Artifact Storage

Aqueduct automatically captures and versions all of the intermediary artifacts in your workflows when they run. This allows you to capture the evolution of your workflows, ensure they're behaving as expected, and debug them when things go wrong.

By default, Aqueduct uses the local filesystem (`$HOME/.aqueduct`) for artifact storage. However, if you're running Aqueduct workflows on a cloud compute engine (e.g., Kubernetes) or simply don't want to use your local filesystem, you can configure Aqueduct's artifact to storage to live in cloud object storage like AWS S3 or Google Cloud Storage.&#x20;

You can update the Aqueduct artifact store in two ways:&#x20;

1. From the command line, you can set the server to use a particular cloud bucket as the artifact store:\
   `aqueduct storage --use s3 --path / --region us-east-2 —credentials $HOME/.aws/credentials`\
   For more details see the [aqueduct-cli.md](../api-reference/aqueduct-cli.md "mention") reference.
2. From the UI, you can connect an AWS S3 or GCS resource. When creating the resource, you'll have the option to use this resource for artifact storage by checking this box:&#x20;

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### Retention Policy

Of course, storing all your intermediary data for every workflow can become expensive. Aqueduct allows you to configure a retention policy on a per-workflow basis that configures how many runs' worth of metadata to keep. When the numbers of runs exceeds this limit, older runs will be garbage collected. You can set this parameter when creating your workflow:

```python
import aqueduct as aq
client = aq.Client()

db = client.resource('aqueduct_demo')
table = db.sql('SELECT * FROM wine;')

client.publish_flow(
    'Wines Data',
    artifacts=[table],
    k_latest_runs=3 # After 3 runs, old data snapshots will be garbage collected.
)
```

## Usage Statistics

By default, Aqueduct collects **fully anonymized** usage statistics. We capture the route hit by each API call and its HTML response code but remove all data and naming metadata. This information helps us understand whether there are any issues with Aqueduct and to continuously improve the experience.&#x20;

If you would like to start Aqueduct with this collection disabled, you can run:

```bash
aqueduct start --disable-usage-stats 
```

Please refer to our [usage stats class](https://github.com/aqueducthq/aqueduct/blob/main/src/golang/cmd/server/middleware/usage/models.go) to see the full list of stats we collect.
