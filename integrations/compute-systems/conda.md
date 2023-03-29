---
description: Run your ML pipelines in isolated Conda environments.
---

# Conda

## Connecting to Conda

Unlike other [.](./ "mention"), Aqueduct's Conda integration is an extension to the default Aqueduct execution engine. Aqueduct can use Conda to create isolated Python environments with different Python versions and the appropriate library dependencies.&#x20;

Before proceeding, please ensure you have the following libraries installed on your Aqueduct server:&#x20;

* [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html)
* [conda-build](https://docs.conda.io/projects/conda-build/en/stable/install-conda-build.html)

Once you have installed these packages, you can proceed to the Aqueduct UI, click on the Conda icon, and give your integration a name. Aqueduct will automatically configure Conda environments from you.

## Using Conda

You can create workflows that use Conda in one of two ways.

**First**, you can set the Aqueduct `global_config` to use your Conda integration:

```python
import aqueduct as aq
aq.global_config({ "engine": "my_conda_integration" })
```

As discussed in [configuring-your-python-environment.md](../../installation-and-configuration/configuring-your-python-environment.md "mention"), this will execute all code — both interactive function calls and workflows published from this environment — using `my_conda_integration`.

**Second**, you can set the `engine` field when calling `publish_flow`:

```python
flow = client.publish_flow(
    name="Conda Example",
    artifacts=output_artifact,
    engine="my_conda_integration",
)
```

In both cases, each function invocation will be executed in a separate Conda environment. If the Python version and `requirements.txt` for multiple functions is the same, Aqueduct will reuse the same Conda environment for multiple functions to avoid the overhead of creating a new Conda environment. For more details on how it works, see our [blog post](https://aqueducthq.com/post/managing-python-environments-in-aqueduct/) on managing Python environments.
