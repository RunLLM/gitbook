# Specifying a requirements.txt

The simplest way to specify a `requirements.txt` is to create one in the directory where you're running the Aqueduct SDK. When Aqueduct detects that a `requirements.txt` is present, it will automatically use this file to determine the requirements.

If no `requirements.txt` is specified, Aqueduct will automatically use the local `pip` dependencies (calling `pip freeze`) to detect what packages are installed, and it will recreate that environment exactly when executing the function.&#x20;

**Note that if you have a large number of dependencies installed in your development environment, this could negatively impact performance**. If this is the case, we strongly recommend providing a specific `requirements.txt` file.

### Using Function-Specific Requirements

In some rare cases, individual operators might have different and conflicting requirements. Aqueduct supports specifying requirements on a function-by-function basis. This is simple: in the `@op` decorator, set `reqs_path` to the path to the function-specific `requirements.txt`:

```python
@op(reqs_path="requirements/requirements.txt")
def valid_sentiment_prediction(reviews: pd.DataFrame) -> pd.DataFrame:
    import transformers

    model = transformers.pipeline("sentiment-analysis")
    return reviews.join(pd.DataFrame(model(list(reviews["review"]))))
```
