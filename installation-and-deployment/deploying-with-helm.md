# Deploying on Kubernetes with Helm Chart

To deploy Aqueduct on a Kubernetes cluster, you can use the helm chart we've created. Please follow the steps below.

### Prerequisites

* Follow the instruction [here](https://helm.sh/docs/intro/install/) to install Helm on your local machine.
* You need to have access to an existing Kubernetes cluster. This means your local machine needs to have a kubeconfig file (typically located in `~/.kube/config`) that points to your cluster.
* Make sure [`kubectl`](https://kubernetes.io/docs/tasks/tools/) is installed on your local machine.

### Setting Up

1. Add the repository to your local Helm manager.
   ```
   helm repo add aqueduct https://aqueducthq.github.io/aqueduct-helm/
   ```

2. Sync versions available in the repo to your local Helm cache.
   ```
   helm repo update
   ```

3. Search for the latest stable versions of the charts found.
   ```
   helm search repo aqueduct
   ```

4. Install the Helm chart.
   ```
   helm install aqueduct/aqueduct --generate-name
   ```

### Customizing Install Options

The installation command above deploys Aqueduct in the default namespace with the server running with Python 3.8. The name of the deployment is generated with the `--generate-name` flag. You can customize all of these by running the following:
   ```
   helm install <DEPLOYMENT_NAME> aqueduct/aqueduct --namespace <NAMESPACE> --set image.repository=<REPO_NAME> 
   ```

Supported image repositories are:
* `aqueducthq/aqueduct-py37`
* `aqueducthq/aqueduct-py38`
* `aqueducthq/aqueduct-py39`
* `aqueducthq/aqueduct-py310`

You can customize based on which Python version you want the Aqueduct server to be running with. By default, `aqueducthq/aqueduct-py38` is used.

### Accessing the Server

After the installation, you can get the server endpoint by running the following:
   ```
   export SERVICE_IP=$(kubectl get svc <DEPLOYMENT_NAME> --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}" -n <NAMESPACE>)
   echo http://$SERVICE_IP:8080
   ```

The API key is printed to the log of the Aqueduct server pod:
   ```
   kubectl logs <AQUEDUCT_POD_NAME> -n <NAMESPACE> | grep "API Key"
   ```