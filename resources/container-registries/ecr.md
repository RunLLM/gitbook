---
description: Pull custom Docker images from ECR to be used by Aqueduct operators.
---

# ECR

The ECR container registry resource allows you to pull custom Docker images from ECR to be used by
Aqueduct operators. Currently, custom images are only supported for Kubernetes engines.

## When to use it

If your operator has complex requirements, such as involving both Python and non-Python dependencies, you
may want to package all of your dependencies into a custom Docker image. You can then use the ECR
resource to pull your custom image and use it to run your operator.

## How to use it

### Registering a ECR Resource

On the UI’s “Resources” page, under the “Container Registry” section, click “ECR” and input your AWS credentials. This can be done via either entering the access keys or specifying the path to your AWS credentials file on your Aqueduct server.

<figure><img src="../../../.gitbook/assets/ecr_registration.png" alt=""><figcaption></figcaption></figure>

You can also complete the registration step via the SDK as follows:

```python
from aqueduct import Client

client = Client()
client.connect_resource(name="my_ecr_resource", service="ECR", config={
    "config_file_path": "~/.aws/credentials",
    "config_file_profile": "default",
})
```

### Building a Custom Docker Image

To build a custom Docker image, you need to use one of the base images provided by Aqueduct. For non-GPU
environments, you can use the `aqueducthq/function<PYTHON_VERSION>:<AQUEDUCT_VERSION>` image.
Replace `<PYTHON_VERSION>` with the Python version you want to use (e.g. `310` for Python 3.10) and
replace `<AQUEDUCT_VERSION>` with the Aqueduct version you want to use (e.g. `0.3.0`). For GPU
environments, you can use the `aqueducthq/function<PYTHON_VERSION>gpu:<AQUEDUCT_VERSION>` image.

Aqueduct relies on a built-in Docker start script to run the operator. Therefore, your custom image
should *not* have an entrypoint or a `CMD` specified.

Here is an example `Dockerfile` that builds a custom image for an operator running on Python 3.10:

```dockerfile
FROM aqueducthq/function310:0.3.0

COPY my_custom_operator_dependencies /
```

### Using a Custom Docker Image

Here is an example of how to use a custom Docker image in your operator:

```python
from aqueduct import Client, op

client = Client()

# This API validates that the image exists in the ECR registry and that ECR has the
# permissions to pull the image. It returns an `Image` object that can be used as the `image`
# parameter for the `op` decorator.
custom_image = client.resource("my_ecr_resource").image("my-custom-image:v2")

@op(
    engine="my_k8s_engine",
    image=custom_image,
)
def run_in_custom_image():
    return "I am running in a custom Docker image!"

run_in_custom_image() # This operator will run in the custom Docker image
```

Please refer to the [API documentation](https://docs.aqueducthq.com/api-reference/sdk-reference/package-aqueduct/package-aqueduct.resources/aqueduct.resources.ecr) for more details on the `image` API of the ECR resource.