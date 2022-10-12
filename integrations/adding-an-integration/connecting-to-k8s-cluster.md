# Connecting to Kubernetes cluster

Connecting Aqueduct to a Kubernetes cluster requires a `kubeconfig` file that allows a kubectl client to connect to the cluster. *This file must be on the same machine running the Aqueduct server.*

Note that in order to use Kubernetes, you must use
* `AWS S3` or `GCS` as your metadata storage 
*  A datasource the Kubernetes cluster has access to. The Aqueduct Demo DB is incompatible with this integration.

Note that there is a minimum node resource configuration needed to run Aqueduct on the engine: 
* 4 vCPU
* 8GB Memory

On the Aqueduct UI, click on the Kubernetes logo and enter the following information:

* A unique name for your Kubernetes connection.
* Absolute path to the `kubeconfig` file.
* The cluster name. You can find this in your `kubeconfig` file.

You should be good to go! Let us know if you run into any issues with your Kubernetes integrations on our community [Slack](https://slack.aqueducthq.com) or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).

For documentation on how to launch Kubernetes clusters, refer to the following:
* [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster)
* [AWS Elastic Kubernetes Service](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
