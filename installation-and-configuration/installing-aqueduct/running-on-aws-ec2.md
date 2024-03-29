---
description: Run Aqueduct on an AWS EC2 VM
---

# Running on AWS EC2

To run an Aqueduct server in production, we recommend running Aqueduct on a cloud server like AWS EC2. Setting up your EC2 instance to run Aqueduct will require a few extra steps.

## Prerequisites

Before you can run Aqueduct on EC2, please make sure you have the following:

* an AWS account with permissions to create and modify EC2 instances
  * you can verify this by following [this AWS guide to launching EC2 instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html)
* an [EC2 KeyPair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) that will allow you to SSH into your EC2 instance

{% hint style="info" %}
Aqueduct doesn't currently run on Windows or Windows Server.
{% endhint %}

## Getting Started

To begin, you will need to create an AWS EC2 VM from the AWS Console. You can follow the steps specified by the launch wizard, and you can also find the full AWS documentation for creating an EC2 instance [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html).

* **Operating system:** We recommend running Aqueduct on Ubuntu, but Aqueduct is configured to support all debian and CentOS-based installations, including Amazon Linux.
* **Resources**: We recommend running Aqeuduct on a server with at least 2 vCPUs and 4GB of RAM. If you plan to run workflows on Aqueduct without using an external compute system (e.g., Kubernetes, Databricks), you will need to allocate enough RAM to fit your datasets in memory.
* **Python version:** Aqueduct supports Python 3.7-3.10. See below for more on ensuring that you have the right Python version installed. If you need to run Aqueduct with multiple Python versions see our guide on using Aqueduct with [Conda](../../resources/compute-systems/conda.md).

### Verify your Python version

Different Linux installations have different Python versions installed by default (e.g., Ubuntu 18 has Python 3.6 installed, and Ubuntu 20 has Python 3.8 installed). **Aqueduct supports Python 3.7-3.10.** If your OS has an older version of Python installed, please upgrade to a valid Python version.

Here are some guides that provide instructions on updating Python installations for different Linux OSes:

* [Ubuntu](https://computingforgeeks.com/how-to-install-python-on-ubuntu-linux-system/)
* [RHEL](https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/8/html/configuring\_basic\_system\_settings/assembly\_installing-and-using-python\_configuring-basic-system-settings)
* [CentOS](https://techviewleo.com/how-to-install-python-on-centos-linux/)

For other operating systems, you should be able to find a guide by searching for "install python 3.10 \<operating system>"

{% hint style="info" %}
Please make sure that you install the corresponding version of `pip` when you install Python. You can verify what version of `pip` your running by checking the output of `pip --version` or `python3 -m pip --version`.
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

If you don't have access to the Aqueduct CLI, you will need to check where the Aqueduct server is installed. By default, it is installed in `$HOME/.local/bin`, which will need to be added to your `PATH` variable. If necessary, add the following line to your `.bashrc` file:

```bash
export PATH=$PATH:$HOME/.local/bin
```

### Starting Aqueduct

Once Aqueduct's installed, you can start the Aqueduct server with. Though it is not required, **we recommend running the Aqueduct server** [**within a tmux session**](https://github.com/tmux/tmux/wiki)**.**

```bash
aqueduct start --expose
```

The `--expose` flag here tells Aqueduct that you are planning to access the Aqueduct server from a different machine than the one you're running the server one. This makes minor changes to the internal Aqueduct config to ensure the server is externally accessible. See [configuring-aqueduct.md](../configuring-aqueduct.md "mention") for more details.

When you run this line, you will see automatic output in your terminal that shows you your API key and a link to your Aqueduct server. For example, if the public IP address of your EC2 server is `1.1.1.1`, you will see:

```
***************************************************
Your API Key: 9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G

The Web UI and the backend server are accessible at: http://1.1.1.1:8080/login?apikey=9NVWLBJ63XAQSU41RMCYKD0ZF87IOH2G
***************************************************
```

However, to access the server, you will need to configure your EC2 instance to enable this access. We'll cover this process next.

## Enabling Aqueduct Access

Once you've set up the Aqueduct server, we have to ensure that Aqueduct is externally accessible. We'll need access to port 8080, so you can access the Aqueduct server externally. This will require a few steps on the AWS console.

1. Log into AWS. Access the EC2 console -- if you don't see an EC2 option readily available, you can type "EC2" into the search bar at the top.
2.  From the EC2 console, navigate to the list of instances you've created:\\

    <figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
3.  Select the instance that you're running Aqueduct on from your list of instances, and select the security tab on the bottom pane: \\

    <figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
4.  Click on the security group your instance is in. From the security group view, select "Edit Inbound Rules":\\

    <figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>
5. Scroll to the bottom of the page, and click "Add Rule." Add a new "Custom TCP" rule that allows requests on port 8080 from anywhere:\
   ![](<../../.gitbook/assets/image (11).png>)
6. Click "Save Rules" and you're go to go!

### Accessing Aqueduct

We're ready to go! The last thing you'll need is the public IP address of your EC2 instance, which you can access from the Details tab of your instance overview on the EC2 console:

![](<../../.gitbook/assets/image (14).png>)

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

In the screenshot above, our IP address is `3.18.104.60`. You can navigate to `3.18.104.60:8080` in your browser, and you will be prompted to enter the API key displayed above.

To create an SDK client to access this Aqueduct server, you can run the following code:

```python
import aqueduct as aq

SERVER_ADDRESS = "3.18.104.60:8080"
API_KEY = "ABCDEF"

client = aq.Client(API_KEY, SERVER_ADDRESS)
```
