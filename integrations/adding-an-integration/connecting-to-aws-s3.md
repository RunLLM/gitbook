# Connecting to AWS S3

{% hint style="danger" %}
This process does not currently work with Aqueduct Community Edition.
{% endhint %}

Connecting Aqueduct to AWS S3 requires special configuration to manages AWS IAM (Identiy & Access Management) permissions. This guide will walk you through creating and IAM policy and an IAM role, which Aqueduct will use to connect to your S3 bucket.

### Creating an IAM Policy

1. On your AWS Console, navigate to the [IAM Page](https://console.aws.amazon.com/iam/home).
2. Click on `Create Policy`.
3.  Enter the following policy document into the JSON editor. **Make sure to replace `{your-bucket-name}` with the name of the bucket you'd like to connect to Aqueduct.**\
    ****

    ```json
    {
    "Version": "2012-10-17",
    "Statement": [
        {
          "Effect": "Allow",
          "Action": ["s3:*"],
          "Resource": "arn:aws:s3:::{YOUR-BUCKET-NAME}/*"
        },
        {
          "Effect": "Allow",
          "Action": ["s3:*"],
          "Resource": "arn:aws:s3:::{YOUR-BUCKET-NAME}"
        }
      ]
    }
    ```
4. Assign your policy a unique name and note down that name.

### Creating an IAM Role

1. Return to the [IAM page](https://console.aws.amazon.com/iam/home).
2. Click `Create Role`.
3. Under `Trusted Entity Type`, select `AWS Account`.
4. Next, select `Another AWS Account` and enter Aqueduct's AWS account ID, which is `123456789`.
5. Select `Require External ID` and note down the unique ID (a hyphenated sequence of words) generated by the AWS UI.
6. Add your previously created policy to this role.
7. Provide a role name.
8. Once you click enter, you should be given an ARN (an AWS unique ID) for this new role.

### Connect Aqueduct to your S3 Bucket

1. Click on the AWS S3 icon on the Aqueduct UI.
2. Give your S3 connection a unique name and specify which bucket you're connecting to.
3. Enter the external ID created for you under `Creating an IAM Role`.
4. Copy the ARN of the role you created under `Creating an IAM Role` into the final section.

And.. you're done! 🎉

We know that wasn't the most pleasant experience, but unfortunately, AWS has quite a complex set of permissioning schemes, and there's not much we can do about it. If you found anything in this guide confusing, please let us know on our community Slack or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).