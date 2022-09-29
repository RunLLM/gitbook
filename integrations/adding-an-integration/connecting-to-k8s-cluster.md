# Connecting to K8s cluster

Connecting Aqueduct to a K8s cluster requires a `kubeconfig` file that allows a kubectl client to connect to the cluster.&#x20;

Note that in order to use K8s, you must use
* `AWS S3` or `GCS` as your metadata storage 
*  an external data source (SQLlite is incompatible).&#x20;

For documentation on how to launch K8s clusters, refer to the following:
* [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster)
* [AWS Elastic Kubernetes Service](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)

Note that there is a minimum node resource configuration needed to run Aqueduct on the engine: 
* 4vCPU
* 8GB Memory

On the Aqueduct UI, click on the K8s logo and enter the following information:

* A unique name for your K8s connection.
* Absolute path to the `kubeconfig` file. (Note that this file must be on the same machine running the Aqueduct server.)
* The cluster name

You should be good to go! Let us know if you run into any issues with your K8s integrations on our community Slack or on [GitHub](https://github.com/aqueducthq/aqueduct/issues/new).
