# Connecting to AWS S3
To connect to S3, you need to provide the S3 bucket name, region, and credentials. Aqueduct supports two types of credentials: [AWS IAM Access Key](#aws-iam-access-key "mention") and [AWS SSO](#aws-sso "mention"). Before connecting, make sure the credentials has access to the S3 Bucket.&#x20; 
![](<../../.gitbook/assets/integration_s3.png>)
## AWS IAM Access Key
You connect to S3 using the Access Key for your AWS account, or you can also create a separate Access Key by following the instructions [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys). There are 3 ways to provide your credentials to Aqueduct: [manually enter credentials](#manually-enter-credentials "mention"), [upload your credentials file](#upload-your-credentials-file "mention"), and [specify the path to your credentials file](#specify-the-path-to-your-credentials-file "mention")

### Manually Enter Credentials
You can manually copy over the Key ID and the Secret Key.&#x20;
![](<../../.gitbook/assets/integration_s3_credential_manual.png>)

### Upload Your Credentials File
If you are using [AWS CLI](https://aws.amazon.com/cli/) or other AWS SDK on your machine, you might have your credentials file available to you, typically under `<HOME>/.aws/credentials`. You can upload this file to Aqueduct and specify the profile name you would like to use.&#x20;
![](<../../.gitbook/assets/integration_s3_credential_upload.png>)

### Specify the Path to Your Credentials File
If you are using [AWS CLI](https://aws.amazon.com/cli/) or other AWS SDK on your machine, you might have your credentials file available to you, typically under `<HOME>/.aws/credentials`. If your server is running on the same machine as this file, you can specify the **absolute path** to this file together with the profile name you would like to use. When you provide credentials by file path, any updates to the file contents will **automatically update all workflows** using this integration.&#x20;
![](<../../.gitbook/assets/integration_s3_credential_path.png>)

## AWS SSO
Aqueduct can connect to S3 using your [AWS SSO](https://aws.amazon.com/iam/identity-center/) credentials. For now, specifying the path is **the only way** to use these credentials. This also means your server **must** be running on the same machine as the one with SSO access.&#x20;

When using [AWS SSO](https://aws.amazon.com/iam/identity-center/), your credentials file is typically be under `<HOME>/.aws/config` (this is **different** from `<HOME>/.aws/credentials`). To connect to S3, you can specify the **absolute path** to this file together with the profile name you would like to use. Once the path is specified, any updates to the file contents (for example, [re-authenticate SSO using browser](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)) will **automatically update all workflows** using this integration.&#x20;

![](<../../.gitbook/assets/integration_s3_credential_path.png>)

