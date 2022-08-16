# Conquering the Python Environment

**Problem**: You've ran `pip3 ...` but the command doesn't seem to be affecting your Python script.
**Solution**: Instead of running `pip3 ...` run `python3 -m pip ...` so that you can ensure the pip is installing in a place the Python interpreter you are using can find it.