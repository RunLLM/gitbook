# Table of Contents

* [aqueduct.resources.gar](#aqueduct.resources.gar)
  * [GARResource](#aqueduct.resources.gar.GARResource)
    * [describe](#aqueduct.resources.gar.GARResource.describe)
    * [image](#aqueduct.resources.gar.GARResource.image)

<a id="aqueduct.resources.gar"></a>

# aqueduct.resources.gar

<a id="aqueduct.resources.gar.GARResource"></a>

## GARResource Objects

```python
class GARResource(BaseResource)
```

Class for GAR resource.

<a id="aqueduct.resources.gar.GARResource.describe"></a>

#### describe

```python
def describe() -> None
```

Prints out a human-readable description of the GAR resource.

<a id="aqueduct.resources.gar.GARResource.image"></a>

#### image

```python
def image(image_name: str) -> Dict[str, str]
```

Returns a dictionary with the name of the GAR resource and the image URL, which can be
used as input to the `image` field of an operator's decorator. This method also verifies
that the image exists in the GAR repository.

**Arguments**:

- `image_name` - The name of the image to retrieve. Should be in the form of `location/project_id/repo/image:tag`.

