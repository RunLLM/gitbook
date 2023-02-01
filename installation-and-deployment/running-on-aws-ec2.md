# Running on AWS EC2

To run an Aqueduct server in production, we recommend running Aqueduct on a cloud server like AWS EC2. Setting up your EC2 instance to run Aqueduct will require a few extra steps, and this guide will walk you through that process.&#x20;

### Prerequisites

Before you can run Aqueduct on EC2, please make sure you have the following:

* an AWS account with permissions to create and modify EC2 instances
  * you can verify this by following [this AWS guide to launching EC2 instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html)
* an [EC2 KeyPair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) that will allow you to SSH into your EC2 instance

**Note**: Aqueduct does not currently run on Windows or Windows Server.

### Setting Up

We recommend running Aqueduct on Ubuntu, but Aqueduct is configured to support all debian and CentOS-based installations, including Amazon Linux.&#x20;

#### Check Your Python Version

Different Linux installations have different Python versions installed by default (e.g., Ubuntu 18 has Python 3.6 installed, and Ubuntu 20 has Python 3.8 installed). **Aqueduct supports Python 3.7-3.10.** If your OS has an older version of Python installed, please upgrade to a valid Python version.&#x20;

Here are some guides that provide instructions on updating Python installations for different Linux OSes:&#x20;

* [Ubuntu](https://computingforgeeks.com/how-to-install-python-on-ubuntu-linux-system/)
* [RHEL](https://access.redhat.com/documentation/en-us/red\_hat\_enterprise\_linux/8/html/configuring\_basic\_system\_settings/assembly\_installing-and-using-python\_configuring-basic-system-settings)
* [CentOS](https://techviewleo.com/how-to-install-python-on-centos-linux/)

For other operating systems, you should be able to find a guide by searching for "install python 3.10 \<operating system>"

**Note:** Please make sure that you install the corresponding version of `pip` when you install Python. You can verify what version of `pip` your running by checking the output of `pip --version` or `python3 -m pip --version`.

#### Install Aqueduct&#x20;

Once you have the correct version of Python installed, you can go ahead and install Aqueduct by running:

```bash
# You can also use python3 -m pip depending on how you installed python & pip.
pip3 install aqueduct-ml
```

#### Starting Aqueduct

Once Aqueduct's installed, you can start the Aqueduct server with. Though it is not required, **we recommend running the Aqueduct server** [**within a tmux session**](https://github.com/tmux/tmux/wiki)**.**

```bash
aqueduct start --expose
```

The `--expose` flag here tells Aqueduct that you are planning to access the Aqueduct server from a different machine than the one you're running the server one. This makes minor changes to the internal Aqueduct config to ensure the server is externally accessible.&#x20;

#### Enabling Aqueduct Access

Once you've set up the Aqueduct server, we have to ensure that Aqueduct is externally accessible. What we'll need to do is open up port 8080, so you can access the Aqueduct server externally. This will require a few steps on the AWS console.

1. Log into AWS. Access the EC2 console -- if you don't see an EC2 option readily available, you can type "EC2" into the search bar at the top.
2. From the EC2 console, navigate to the list of instances you've created: \
   &#x20;![](<../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (2).png>)
3. Select the instance that you're running Aqueduct on from your list of instances, and select the security tab on the bottom pane: \
   ![](<../.gitbook/assets/image (9).png>)
4. Click on the security group your instance is in. From the security group view, select "Edit Inbound Rules":\
   ![](<../.gitbook/assets/image (10).png>)
5. Scroll to the bottom of the page, and click "Add Rule." Add a new "Custom TCP" rule that allows requests on port 8080 from anywhere: \
   ![](<../.gitbook/assets/image (11).png>)
6. Click "Save Rules" and you're go to go!

#### Accessing Aqueduct

We're ready to go! The last thing you'll need is the public IP address of your EC2 instance, which you can access from the Details tab of your instance overview on the EC2 console:

![](<../.gitbook/assets/image (14).png>)
