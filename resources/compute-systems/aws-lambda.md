---
description: Run your ML pipelines on AWS Lambda.
---

# AWS Lambda

## Connecting to AWS Lambda

{% hint style="info" %}
Before you can connect Aqueduct to AWS Lambda, you must be using AWS S3 as your artifact store. See [configuring-aqueduct.md](../../installation-and-configuration/configuring-aqueduct.md "mention") for more on how to update your artifact storage layer.
{% endhint %}

Before proceeding, please ensure Docker is installed ([instructions](https://docs.docker.com/engine/install/)). Aqueduct uses AWS Lambda's containerization functionality to ship code, so you will need Docker installed before proceeding.

To connect Aqueduct to AWS Lambda, you will need the following information:&#x20;

* **Name**: A unique name for your connection. This is totally up to you!
* **Lambda Role ARN:** The ARN for an IAM role you have created that has access to the following permissions: `AWSXRayDaemonWriteAccess`, `AWSLambdaBasicExecutionRole`, `AmazonS3FullAccess`. Please see these [instructions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-console) for more detail on how to create this role.

### UI

To connect Aqueduct to AWS Lambda, navigate to the Aqueduct resources page and click on the AWS Lambda icon. Enter the information above.

### SDK

Aqueduct does not currently support connecting an AWS Lambda resource via the SDK. If you need this capability, please [let us know](https://github.com/aqueducthq/aqueduct/issues/new?assignees=\&labels=enhancement\&template=feature\_request.md\&title=%5BFEATURE%5D)!

## Using AWS Lambda

You can create workflows that use AWS Lambda in one of two ways.

**First**, you can set the Aqueduct `global_config` to use your AWS Lambda resource:

```python
import aqueduct as aq
aq.global_config({ "engine": "my_lambda_resource" })
```

As discussed in [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention"), this will execute all code — both interactive function calls and workflows published from this environment — using `my_lambda_resource`.

**Second**, you can set the `engine` field when calling `publish_flow`:

```python
flow = client.publish_flow(
    name="Lambda Example",
    artifacts=output_artifact,
    engine="my_lambda_resource",
)
```

In both cases, each function invocation will cause Aqueduct to create a containerized Python environment with the appropriate library dependencies, and trigger a function execution on AWS Lambda.&#x20;

### Resource Requests

AWS Lambda allows you to set the amount of memory available to a function, which you can do via the `resources` map on the `@op` decorator:

```python
@op(
    resources={
        'memory': '16GB',
    }
)
def my_fn():
    return True
```
