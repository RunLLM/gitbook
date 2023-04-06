---
description: Power your Aqueduct workflow with data from AWS S3
---

# AWS S3

## Connecting to AWS S3

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Bucket**: The name of the S3 Bucket you're connecting to.

* **AWS Credentials**: Aqueduct supports multiple forms of authentication for AWS services.
  * **Enter Access Keys**: You can enter your AWS Access Key ID, Secret Access Key, and AWS Region directly into the Aqueduct UI.
  * **Path to Credentials**: If your AWS credentials are on the same machine as the Aqueduct server, you can specify the path to a file with the AWS credentials. **This is the preferred connection method if you use AWS SSO.**
  * **Upload Credentials File**: Finally, you can upload a file that has your AWS access keys in it and specify the profile to use in that file.&#x20;

### UI

To connect Aqueduct to AWS S3, navigate to the Aqueduct integrations page and click on the AWS S3 icon. Select the authentication method you prefer, and enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import S3Config, AWSCredentialType

client = aq.Client(s)

# Choose one of the two options below.
conf = S3Config(
    database="<DATABASE>"
    type=AWSCredentialType.ACCESS_KEY,
    output_location="<PATH_TO_S3_LOCATION>",
    access_key_id="<ACCESS_KEY_ID>",
    secret_access_key="<SECRET_ACCESS_KEY>",
    region="<AWS_REGION>",
)

conf = S3Config(
    database="<DATABASE>",
    type=AWSCredentialType.CONFIG_FILE_PATH,
    output_location="<PATH_TO_S3_LOCATION>",    
    config_file_path="<PATH_TO_CONFIG>"
    config_file_profile="<NAME_OF_PROFILE>"
)

client.connect_integration("my_s3_integration", "S3", conf)
```

## Using AWS S3

Once you've creating a AWS S3 connection, you can access your data from the Aqueduct SDK. AWS S3 is an object store, so you can load any type of data from S3. See [artifacts.md](../../../artifacts.md "mention") for the full list of data types that Aqueduct supports. If you don't see the type you require, you can fall back to either `bytes` or `picklable`.

{% hint style="info" %}
When reading or writing tabular data from/to S3, you will need to provide the serialization type of the tabular data. Aqueduct currently supports JSON files, CSV files, and Parquet files.
{% endhint %}

### Reading Data

{% hint style="info" %}
Using Aqueduct's S3 integration, you can either load one file or a list of files. The `filepaths` argument corresponding accepts either a string or a list of strings. If you specify a list of strings, you will receive the objects with the appropriate serialization mechanism collated into a single list. Note that all objects in the list must be of the same type.
{% endhint %}

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'aws_s3'

bucket = client.integration(INTEGRATION_NAME)
wines = bucket.file(
    filepaths='wines.csv', 
    artifact_type=aq.ArtifactType.TABLE, 
    format='csv'
)

my_object = bucket.file(
    filepaths='my_object.pkl',
    artifact_type=aq.ArtifactType.PICKLABLE
)
```

### Writing Data

```python
# When saving a non-tabular artifact, the correct type will automatically
# be inferred for you.
bucket.save(
    my_object,
    filepath="path/to/my_object",
)

# When saving a tabular artifact, you must specify the serialization type: 
# JSON, CSV, or Parquet.
bucket.save(
    wines,
    filepath="path/to/wines",
    format="json",
)
```
