# Table of Contents

* [aqueduct.resources.ecr](#aqueduct.resources.ecr)
  * [ECRResource](#aqueduct.resources.ecr.ECRResource)
    * [describe](#aqueduct.resources.ecr.ECRResource.describe)
    * [image](#aqueduct.resources.ecr.ECRResource.image)

<a id="aqueduct.resources.ecr"></a>

# aqueduct.resources.ecr

<a id="aqueduct.resources.ecr.ECRResource"></a>

## ECRResource Objects

```python
class ECRResource(BaseResource)
```

Class for ECR integration.

<a id="aqueduct.resources.ecr.ECRResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the ECR integration.

<a id="aqueduct.resources.ecr.ECRResource.image"></a>

#### image

```python
def image(image_name: str) -> Dict[str, str]
```

Returns a dictionary with the name of the ECR resource and the image URL, which can be
used as input to the `image` field of an operator's decorator. This method also verifies
that the image exists in the ECR repository.

**Arguments**:

- `image_name` - The name of the image to retrieve. Should be in the form of `image:tag`.
  No need to include the endpoint URL prefix such as 123456789012.dkr.ecr.us-east-1.amazonaws.com.

