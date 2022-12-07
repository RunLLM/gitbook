# Improve Dependencies and Python Version Management Using Conda

By default, all Aqueduct operators runs directly in the Aqueduct server's environment. They share the server environment's python version and package dependencies. Operators also share all packages installed by their [requirement.txt](./specifying-a-requirements.txt.md) files.

You may find this causing some issue for you, especially if your SDK notebook and Aqueduct server is running on different machines, or if your operators have conflicting dependencies.

Aqueduct allows you to run operators in an isolated environment that's precisely consistent with the operator's [requirement.txt](./specifying-a-requirements.txt.md) file as well as your SDK notebook's python version. To do so, you would need to install [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html) and [conda-build](https://docs.conda.io/projects/conda-build/en/stable/install-conda-build.html). Once installed, you can connect to conda from the [integrations page](../integrations/README.md). You can also refer to [this guide](../integrations/adding-an-integration/connecting-to-conda.md) for more details.
