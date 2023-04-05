---
description: Use a Helm Chart to run an Aqueduct server on your Kubernetes cluster.
---

# Running on Kubernetes with Helm

To deploy Aqueduct on a Kubernetes cluster, you can use the Helm chart we've created. Please follow the steps below.

## Prerequisites

* You need to have access to an existing Kubernetes cluster, and you must have access to your cluster's `kubeconfig` file (typically located in `~/.kube/config`).&#x20;
* Make sure [`kubectl`](https://kubernetes.io/docs/tasks/tools/) is installed. You can verify access to `kubectl` by running `which kubectl`.
* Follow the instructions [here](https://helm.sh/docs/intro/install/) to install Helm on your local machine.

## Setting Up

1.  Add the repository to your local Helm manager.

    ```bash
    helm repo add aqueduct https://aqueducthq.github.io/aqueduct-helm/
    ```
2.  Sync versions available in the repo to your local Helm cache.

    ```bash
    helm repo update
    ```
3.  Search for the latest stable versions of the charts found.

    ```bash
    helm search repo aqueduct
    ```
4.  Install the Helm chart.

    ```bash
    helm install aqueduct/aqueduct --generate-name
    ```

### Customizing your installation

The installation command above deploys the latest version of Aqueduct in the default namespace with a Helm-generated release name. **By default, the server runs with Python 3.8.** You can customize all of these by running the following:

```bash
helm install \
    <RELEASE_NAME> aqueduct/aqueduct \
    --namespace <NAMESPACE> \ 
    --version <VERSION_NUMBER> \ 
    --set image.repository=<REPO_NAME>
```

* `RELEASE_NAME`: the name of the release. This can be any string that you want.
* `NAMESPACE`: the Kubernetes namespace under which the Aqueduct release will be created.
* `VERSION_NUMBER`: the version number of Aqueduct (e.g., 0.2.7).
* `REPO_NAME`: the name of the Docker image repository to use for the deployment. These images are pre-built by Aqueduct and  indicates which Python version the server uses.
  * Python 3.7: `aqueducthq/aqueduct-py37`
  * Python 3.8: `aqueducthq/aqueduct-py38`
  * Python 3.9: `aqueducthq/aqueduct-py39`
  * Python 3.10: `aqueducthq/aqueduct-py310`

### Accessing the Server

After the installation, you can get the server endpoint by running the following:

```bash
export SERVICE_IP=$(kubectl get svc <RELEASE_NAME> --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}" -n <NAMESPACE>)
echo http://$SERVICE_IP:8080
```

To access your API key, you will need to first get the name of the pod running the Aqueduct server and inspect its logs. The API key is printed in the logs of the Aqueduct pod:

```bash
kubectl logs <AQUEDUCT_POD_NAME> -n <NAMESPACE> | grep "API Key"
```
