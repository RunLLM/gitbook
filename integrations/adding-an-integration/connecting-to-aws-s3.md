# Connecting to AWS S3

Aqueduct connects to AWS S3 using an [AWS IAM Access Key](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html). You can either use the Access Key for your account, or you can create a separate Access Key by following the instructions [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys).&#x20; 

Ensure that your IAM user has access to the S3 Bucket you're connecting to Aqueduct. There are 3 ways to provide your credentials to Aqueduct.

## Manually Enter Credentials
You can manually copy over the Key ID and the Secret Key.
![](<../../.gitbook/assets/integration_s3_credential_manual.png>)

## Upload Your Credentials File
If you are using [AWS cli](https://aws.amazon.com/cli/) or other AWS SDK on your machine, you might have your credentials file available to you, typically under `<HOME>/.aws/credentials`. You can upload this file to Aqueduct and specify the profile name you would like to use.
![](<../../.gitbook/assets/integration_s3_credential_upload.png>)

## Specify the Path to Your Credentials File
### Access Key and Secret Access Key
If you are using [AWS cli](https://aws.amazon.com/cli/) or other AWS SDK on your machine, you might have your credentials file available to you, typically under `<HOME>/.aws/credentials`. If your server is running on the same machine as this file, you can specify the **absolute path** to this file together with the profile name you would like to use. When you provide credentials by file path, any updates to the file contents will **automatically update all workflows** using this integration.
![](<../../.gitbook/assets/integration_s3_credential_path.png>)

### SSO Credentials
If you are using [AWS SSO](https://aws.amazon.com/iam/identity-center/) to access your S3 buckets, your credentials file might be under `<HOME>/.aws/config` (this is **different** from `<HOME>/.aws/credentials`). You can specify the **absolute path** to this file together with the profile name you would like to use.

For now, specifying the path is the only way to use [AWS SSO](https://aws.amazon.com/iam/identity-center/) credentials. This also means your server **must** be running on the same machine as the one with SSO access.

