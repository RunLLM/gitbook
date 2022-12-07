# Improve Dependencies and Python Version Management Using Conda

By default, all Aqueduct operators runs directly in the Aqueduct server's environment. They share the server environment's Python version and package dependencies. Operators also share all packages installed by their [requirements.txt](./specifying-a-requirements.txt.md) files.

This maybe cause some issues, especially if you have different Python versions or library dependencies in your development environment and on the Aqueduct server.

Aqueduct allows you to run operators in an isolated environment that's precisely consistent with the operator's [requirements.txt](./specifying-a-requirements.txt.md) file as well as your SDK notebook's Python version. You can refer to [this guide](../integrations/adding-an-integration/connecting-to-conda.md) to connect Aqueduct to conda.
