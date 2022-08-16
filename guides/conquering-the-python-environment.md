# Conquering the Python Environment

**Problem**: You installed Aqueduct via `pip3 install aqueduct-ml`, but still got `No module named 'aqueduct'` when importing the package.

**Solution**: This is likely because the package was installed to a different Python environment from what your `python3` is pointing to. A quick fix is to run `python3 -m pip install aqueduct-ml`. However, it is still a good practice to fix your `pip3` alias to be consistent with `python3`.