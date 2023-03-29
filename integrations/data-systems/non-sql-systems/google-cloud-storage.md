---
description: Power your Aqueduct workflow with data from Google Cloud Storage
---

# Google Cloud Storage

{% hint style="warning" %}
Aqueduct currently only supports Google Cloud Storage as an artifact storage mechanism. If you would like to use GCS for data storage, [please reach out to us](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D).
{% endhint %}

## Connecting to Google Cloud Storage

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Bucket**: The name of the GCS Bucket that Aqueduct should connect to.&#x20;
* **Service Account Credentials**: A JSON file that has the credentials for your Service Account.

{% hint style="info" %}
Google Cloud has documentation [here](https://cloud.google.com/docs/authentication/getting-started#creating\_a\_service\_account) that will walk you through creating a `ServiceAccount` and then creating a corresponding key for that `ServiceAccount`. Once you've finished these steps, you should have a JSON file with the key for your `ServiceAccount`.
{% endhint %}

### UI&#x20;

To connect Aqueduct to Google Cloud Storage, navigate to the Aqueduct integrations page and click on the Google Cloud Storage icon. Enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import GCSConfig

client = aq.Client(s)

conf = GCSConfig(
    bucket="<BUCKET>",
    service_account_credentials_path="<PATH_TO_CREDENTIALS_FILE>"
)
client.connect_integration("my_gcs_integration", "GCS", conf)
```

## Using Google Cloud Storage

We're working on adding support for reading and writing data from GCS. Please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D) if you need support for this feature!
