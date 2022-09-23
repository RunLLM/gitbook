# Installing Aqueduct

### Prerequisites

Please make sure you have the requirements before installing Aqueduct:

* A Unix-based operating system: Aqueduct supports Debian and CentOS-based Linux distributions and macOS (both Intel and ARM-based Macs)
* Python3: Aqueduct supports Python3.7-Python3.10
* `pip3`

### Installing Aqueduct

Aqueduct is packaged as a [Python pip package](https://pypi.org/project/aqueduct-ml/) that you can download and install:

```bash
pip3 install aqueduct-ml
```

This downloads and installs the Aqueduct server, SDK, and CLI. You can start Aqueduct by running `aqueduct start` now. You can import the Aqueduct SDK in Python with `import aqueduct`.

{% hint style="info" %}
By default, the Aqueduct CLI will be installed into the `bin` directory within your Python installation. If you type `which aqueduct` and do not find the package, please locate your Python installation and add `<PATH_TO_PYTHON>/bin` to your `$PATH` variable.

By default, here is where Python packages are installed:

* macOS: `$HOME/Library/Python/3.8/bin`
* Ubuntu: `$HOME/.local/bin`
{% endhint %}

### Installing only the SDK

To install _only the SDK_ with `pip`, you can run:

```python
pip3 install aqueduct-sdk
```

This will install only the Aqueduct SDK without the server -- this package is useful if you're running the Aqueduct server on a different machine from your client machine.
