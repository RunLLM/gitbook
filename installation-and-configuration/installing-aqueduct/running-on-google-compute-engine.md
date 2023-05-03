---
description: Run Aqueduct on a GCE VM
---

# Running on Google Compute Engine

To run an Aqueduct server in production, we recommend running Aqueduct on a cloud server like AWS EC2. Setting up your EC2 instance to run Aqueduct will require a few extra steps.

## Prerequisites

Before you can run Aqueduct on GCE, please make sure you have the following

* a Google Cloud account with permissions to create and modify GCE VMs
* capabilities to use of the following two methods to SSH into your GCE instance:
  * the ability to connect via Google's in-browser SSH
  * the ability to use the [`gcloud`](https://cloud.google.com/sdk/docs/install-sdk) SDK to SSH into your instance
  * keys that enable you to connect via vanilla SSH

## Getting Started

To begin, you will need to create a GCE VM from the GCP console. You can follow the steps specified by the GCE launch wizard.

* **Operating system**: By default, GCE uses Debian GNU, which Aqueduct supports out of the box. You can also use a CentOS-based Linux distribution if you would prefer. **Aqueduct does not work on Windows or Windows Server.**
* **Resources**: We recommend running Aqueduct on a server with at least 2 vCPUs and 4GB of RAM. If you plan to run workflows on Aqueduct without using an external compute system, you will need to allocate enough RAM to fit your datasets in memory.
* **Python version**: Aqueduct supports Python3.7-3.10. See below for more on ensuring that you have the right Python version installed. If you need to run Aqueduct with multiple Python versions, see our guide on using Aqueduct with [Conda](../../resources/compute-systems/conda.md).
* **HTTP + HTTPS Access:** Ensure that you allow HTTP and HTTPS access to this VM when you are creating it.

### Verify your Python version

Different Linux installations have different Python versions installed by default (e.g., Ubuntu 18 has Python 3.6 installed, and Ubuntu 20 has Python 3.8 installed). **Aqueduct supports Python 3.7-3.10.** If your OS has an older version of Python installed, please upgrade to a valid Python version.&#x20;

Here are some guides that provide instructions on updating Python installations for different Linux OSes:&#x20;

* [Ubuntu](https://computingforgeeks.com/how-to-install-python-on-ubuntu-linux-system/)
* [RHEL](https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/8/html/configuring\_basic\_system\_settings/assembly\_installing-and-using-python\_configuring-basic-system-settings)
* [CentOS](https://techviewleo.com/how-to-install-python-on-centos-linux/)

For other operating systems, you should be able to find a guide by searching for "install python 3.10 \<operating system>"

{% hint style="info" %}
&#x20;Please make sure that you install the corresponding version of `pip` when you install Python. You can verify what version of `pip` your running by checking the output of `pip --version` or `python3 -m pip --version`.
{% endhint %}

## Installing Aqueduct

Once you have the correct version of Python installed, you can go ahead and install Aqueduct by running:

```bash
# You can also use python3 -m pip depending on how you installed python & pip.
pip3 install aqueduct-ml
```

Confirm that you have access to the Aqueduct CLI by running:

```bash
which aqueduct
```

If you don't have access to the Aqueduct CLI, you will need to check where the Aqueduct server is installed. By default, it is installed in `$HOME/.local/bin`, which will need to be added to your `PATH` variable. If necessary, add the following line to your `.bashrc` file:&#x20;

```bash
export PATH=$PATH:$HOME/.local/bin
```

### Starting Aqueduct

Once Aqueduct's installed, you can start the Aqueduct server with. Though it is not required, **we recommend running the Aqueduct server** [**within a tmux session**](https://github.com/tmux/tmux/wiki)**.**

```bash
aqueduct start --expose
```

The `--expose` flag here tells Aqueduct that you are planning to access the Aqueduct server from a different machine than the one you're running the server one. This makes minor changes to the internal Aqueduct config to ensure the server is externally accessible. See [configuring-aqueduct.md](../configuring-aqueduct.md "mention") for more details.

When you run this line, you will see automatic output in your terminal that shows you your API key and a link to your Aqueduct server. For example, if the public IP address of your GCE server is `1.1.1.1`, you will see:

```
***************************************************
Your API Key: 9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G

The Web UI and the backend server are accessible at: http://1.1.1.1:8080/login?apikey=9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G
***************************************************
```

However, to access the server, you will need to configure your GCE instance to enable this access. We'll cover this process next.

## Enabling Aqueduct Access

1.  On the GCP console, navigate to the left hand side menu and select the "VPC network section": \


    <figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
2.  Under "VPC network" select the Firewall section: \


    <figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
3.  At the top of the page, select "Create Firewall Rule": \


    <figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
4.  Enter the following configuration parameters and click Create.

    1. Name your firewall rule (e.g., `allow-aqueduct-access`)
    2. Set the Direction of Traffic to be Ingress
    3. Under "Targets" select either "All instances in the network" or "Specified target tags"
       1. If you selected "Specified target tags" ensure that your target tag matches a tag that you add to your GCE VM.
    4. Under "Source IPv4 ranges" enter `0.0.0.0/0`
    5. check the TCP box under "Specified protocols and ports" and enter `8080` as the port number



    1.

    <figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### Accessing Aqueduct

We're ready to go! The last thing you'll need is the public IP address of your GCE VM. You can find this on the GCE VM console under "External IP": \


<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

In the screenshot above, our IP address is `34.133.162.87`. You can navigate to `34.133.162.87:8080` in your browser, and you will be prompted to enter the API key displayed above.&#x20;

To create an SDK client to access this Aqueduct server, you can run the following code:&#x20;

```python
import aqueduct as aq

SERVER_ADDRESS = "34.133.162.87:8080"
API_KEY = "ABCDEF"

client = aq.Client(API_KEY, SERVER_ADDRESS)
```
