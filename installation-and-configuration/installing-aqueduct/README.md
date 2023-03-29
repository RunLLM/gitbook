# Installing Aqueduct

Aqueduct can run on any Unix system (Linux or macOS) and can be installed via a `pip` package. If you don't have `pip` installed, please see the [official documentation](https://pip.pypa.io/en/stable/installation/) for more details.

## Prerequisites

Please make sure you have the requirements before installing Aqueduct:

* A Unix-based operating system: Aqueduct supports Debian and CentOS-based Linux distributions and macOS (both Intel and ARM-based Macs)
* Python3: Aqueduct supports Python3.7-Python3.10
* `pip3`

## Installing Aqueduct

To install Aqueduct, run:

```bash
pip3 install aqueduct-ml
```

{% hint style="info" %}
By default, the Aqueduct CLI will be installed into the `bin` directory within your Python installation. If you type `which aqueduct` and do not find the package, please locate your Python installation and add `<PATH_TO_PYTHON>/bin` to your `$PATH` variable.

By default, here is where Python packages are installed:

* macOS: `$HOME/Library/Python/3.8/bin`
* Ubuntu: `$HOME/.local/bin`
{% endhint %}

To check that the Aqueduct CLI is available, you can run:

```bash
which aqueduct
```

### Starting Aqueduct

Once Aqueduct is installed, the `start` command will start the Aqueduct server and UI:&#x20;

```bash
aqueduct start
```

This will display your server's API key and a link to the Aqueduct server in your terminal. If you're running in a browser-enabled system, this command will automatically open the Aqueduct server in your default browser.

```
***************************************************
Your API Key: 9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G

The Web UI and the backend server are accessible at: http://localhost:8080/login?apikey=9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G
***************************************************
```

To import the Aqueduct SDK and create an API client, you can run the following code in any Python environment:&#x20;

```python
import aqueduct
client = aqueduct.Client()
```

## Installing in other environments

## Installing only the SDK

The `aqueduct-ml` package includes the Python SDK. However, if you would like to install the SDK only on a client machine, the `aqueduct-sdk` package has a standalone installation of only the Python SDK:&#x20;

```python
pip3 install aqueduct-sdk
```

