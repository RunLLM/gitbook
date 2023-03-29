---
description: Power your Aqueduct workflow with data from AWS Athena
---

# AWS Athena

## Connecting to AWS Athena

You will need the following information:

* **Name**: A unique name for your connection. This is totally up to you!
* **Database**: The name of the database you're connecting to; each AWS Athena connection currently only supports a single database.
* **S3 Output Location**: AWS Athena works by writing the results of the query to a location in AWS S3. This bucket must be created in advance, and Aqueduct will read the results from this location.
* **AWS Credentials**: Aqueduct supports multiple forms of authentication for AWS services.
  * **Enter Access Keys**: You can enter your AWS Access Key ID, Secret Access Key, and AWS Region directly into the Aqueduct UI.
  * **Path to Credentials**: If your AWS credentials are on the same machine as the Aqueduct server, you can specify the path to a file with the AWS credentials. **This is the preferred connection method if you use AWS SSO.**
  * **Upload Credentials File**: Finally, you can upload a file that has your AWS access keys in it and specify the profile to use in that file.

{% hint style="info" %}
Since AWS Athena uses an S3 Bucket to store results, the credentials you specify must have access to both Athena **and** the S3 Bucket.&#x20;
{% endhint %}

### UI

To connect Aqueduct to AWS Athena, navigate to the Aqueduct integrations page and click on the AWS Athena icon. Select the authentication method you prefer, and enter the information above.

### SDK

```python
import aqueduct as aq
from aqueduct.integrations.connect_config import AthenaConfig, AWSCredentialType

client = aq.Client(s)

# Choose one of the two options below.
conf = AthenaConfig(
    database="<DATABASE>"
    type=AWSCredentialType.ACCESS_KEY,
    output_location="<PATH_TO_S3_LOCATION>",
    access_key_id="<ACCESS_KEY_ID>",
    secret_access_key="<SECRET_ACCESS_KEY>",
    region="<AWS_REGION>",
)

conf = AthenaConfig(
    database="<DATABASE>",
    type=AWSCredentialType.CONFIG_FILE_PATH,
    output_location="<PATH_TO_S3_LOCATION>",    
    config_file_path="<PATH_TO_CONFIG>"
    config_file_profile="<NAME_OF_PROFILE>"
)

client.connect_integration("my_sqlite_integration", "SQLite", conf)
```

## Using AWS Athena

Once you've creating a AWS Athena connection, you can access your data from the Aqueduct SDK.

### Reading Data

```python
import aqueduct as aq
client = aq.Client()
INTEGRATION_NAME = 'aws_athena'

db = client.integration(INTEGRATION_NAME)
wines = db.sql('SELECT * FROM wine;')
```

### Writing Data

Since AWS Athena is a query layer on top of an AWS S3 bucket, you cannot write data to Athena. Instead, you can write data to the underlying S3 bucket you use with Athena to access via another Athena query in the future.
