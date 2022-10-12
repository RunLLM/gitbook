# Connecting to AWS Lambda

Connecting Aqueduct to Lambda requires the following:
* The machine running Aqueduct has `Docker` installed. Lambda cannot pull from public ECR repositories, so Aqueduct uses Docker to create and populate private ECR repositories automatically.
* ARN for a IAM role with Lambda and S3 permissions.
* `AWS S3` as your metadata storage 
*  A datasource that Lambda has access to. The Aqueduct Demo DB is incompatible with this integration.

For documentation on how to create a IAM role for Lambda, [refer to the following](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-console). The following permissions must be granted to the role:
* `AWSXRayDaemonWriteAccess`
* `AWSLambdaBasicExecutionRole`
* `AmazonS3FullAccess`

On the Aqueduct UI, click on the Lambda logo and enter the following information:

* A unique name for your Lambda connection.
* The ARN for the executor role you created.

You should be good to go! Let us know if you run into any issues with your Lambda integrations on our community [Slack](https://slack.aqueducthq.com) or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).
