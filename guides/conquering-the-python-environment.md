# Conquering the Python Environment

**Problem**: You installed the Aqueduct package via `pip3`, but the package was installed to a different Python environment from what your `python3` is pointing to.
**Solution**: Instead of running `pip3 ...` run `python3 -m pip ...` to ensure `pip3` is installing to the same Python environment as what your python3 is pointing to. Note that it is still a good practice to fix your `pip3` alias to be consistent with `python3`.