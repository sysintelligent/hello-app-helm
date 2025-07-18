# hello-app

Welcome to hello-app, a simple test application designed for rapid deployment and validation on Kubernetes clusters.

## Overview

hello-app is a lightweight application intended for testing and validating Kubernetes deployments. It responds to HTTP requests with a friendly greeting message along with its version number.

## Usage

To install the hello-app chart, follow these steps:

1. Add the sysintelligent Helm repository:
   ```sh
   $ helm repo add sysintelligent https://sysintelligent.github.io/hello-app-helm/
   ```

2. Install the hello-app chart:
   ```sh
   $ helm install my-hello-app sysintelligent/hello-app --version <application version#>
   ```

Once installed, you can test the application by sending an HTTP request to its endpoint:

```sh
$ curl http://<cluster-node-ip>:30000
```
This should return a response similar to: `hello-app says hi! [<application version$>]`

If you are using Minikube, you can test it with the following command:

```sh
$ curl $(minikube ip):30000
```

### Local Testing

To test hello-app locally, follow these steps:

1. Navigate to the hello-app-helm directory:
   ```sh
   $ cd hello-app-helm
   ```

2. Install the hello-app chart:
   ```sh
   $ helm install hello-app .

   Or
   
   $ helm install hello-app ./hello-app-<application version#>.tgz
   ```

3. Send an HTTP request to the application's endpoint:
   ```sh
   $ curl $(minikube ip):30000
   ```

### Deploying to Local Cluster

To deploy the hello-app Helm chart to your local Kubernetes cluster using a kubeconfig file, follow these steps:

1. Navigate to the hello-app-helm directory:
   ```sh
   $ cd hello-app-helm
   ```

2. Set the KUBECONFIG environment variable to point to your kubeconfig file:
   ```sh
   $ export KUBECONFIG=/path/to/your/kubeconfig.yaml
   ```

3. Verify that you can connect to your cluster:
   ```sh
   $ kubectl cluster-info
   ```

4. Install the hello-app chart:
   ```sh
   $ helm install hello-app .
   ```

5. Check the deployment status:
   ```sh
   $ kubectl get pods -n hello-app
   $ kubectl get services -n hello-app
   ```

6. Get the service URL and test the application:
   ```sh
   $ kubectl get service hello-app -n hello-app -o wide
   $ kubectl get nodes -o wide
   ```

   Since your service is configured as NodePort on port 30000, you can access it using any of the node internal IPs:
   ```sh
   $ curl http://<node-internal-ip>:30000
   ```

   For example, if your nodes have internal IPs like 10.250.0.133, 10.250.0.5, 10.250.1.5:
   ```sh
   $ curl http://10.250.0.133:30000
   $ curl http://10.250.0.5:30000
   $ curl http://10.250.1.5:30000
   ```

   **Alternative: Port Forwarding** (if you can't access internal IPs directly):
   ```sh
   $ kubectl port-forward service/hello-app -n hello-app 30000:80
   $ curl http://localhost:30000
   ```

7. To uninstall the chart:
   ```sh
   $ helm uninstall hello-app
   ```

**Note**: Make sure your kubeconfig file has the correct permissions and contains valid cluster credentials. If you encounter authentication issues, verify your cluster access with `kubectl auth can-i get pods -n hello-app`.

## Additional Information

- **Dependencies**: hello-app has no external dependencies and is designed to be lightweight and easy to deploy.
- **Troubleshooting**: If you encounter any issues during installation or testing, please refer to the Helm documentation or Kubernetes troubleshooting guides for assistance.
- **Uninstallation**: To remove hello-app from your cluster, you can use the `helm uninstall` command followed by the release name. For example:
  ```sh
  $ helm uninstall my-hello-app
  ```
