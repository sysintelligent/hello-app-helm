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
   $ kubectl get service hello-app-nodeport -n hello-app -o wide
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
   $ kubectl port-forward service/hello-app-nodeport -n hello-app 30000:80
   $ curl http://localhost:30000
   ```

   **Alternative: LoadBalancer Access** (if available in your cluster):
   ```sh
   # Option A: Using Helm chart with LoadBalancer enabled
   $ helm install hello-app . --set loadBalancer.enabled=true
   
   # Option B: Manual LoadBalancer service creation
   $ kubectl expose deployment hello-app -n hello-app --type=LoadBalancer --port=80 --target-port=8080 --name=hello-app-lb
   
   # Wait for external IP assignment
   $ kubectl get service hello-app-lb -n hello-app -w
   
   # Access via external IP (example: 132.220.189.9)
   $ curl http://<external-ip>
   ```

7. To uninstall the chart:
   ```sh
   $ helm uninstall hello-app
   ```

**Note**: Make sure your kubeconfig file has the correct permissions and contains valid cluster credentials. If you encounter authentication issues, verify your cluster access with `kubectl auth can-i get pods -n hello-app`.

### External Access Methods

After deploying hello-app, you can access it using different methods depending on your cluster setup:

#### 1. **NodePort Access** (Default)
```sh
# Get node IPs
$ kubectl get nodes -o wide

# Access via node internal IPs
$ curl http://<node-internal-ip>:30000
```

#### 2. **Port Forwarding** (Recommended for development)
```sh
# Forward local port to service
$ kubectl port-forward service/hello-app-nodeport -n hello-app 30000:80

# Access locally
$ curl http://localhost:30000
```

#### 3. **LoadBalancer Access** (Production-ready)
```sh
# Option A: Using Helm chart with LoadBalancer service type
$ helm install hello-app . --set service.type=LoadBalancer

# Option B: Using Helm chart with additional LoadBalancer service
$ helm install hello-app . --set loadBalancer.enabled=true

# Option C: Manual LoadBalancer service creation
$ kubectl expose deployment hello-app -n hello-app --type=LoadBalancer --port=80 --target-port=8080 --name=hello-app-lb

# Check external IP assignment
$ kubectl get service hello-app -n hello-app  # for main LoadBalancer service
$ kubectl get service hello-app-lb -n hello-app  # for additional LoadBalancer service

# Access via external IP
$ curl http://<external-ip>
```

**Example Response**: `hello-app says hi! [version: 2.0.1]`

### Service Names and Types

The chart creates different services based on configuration:

| Service Name | Type | Purpose | When Created |
|--------------|------|---------|--------------|
| `hello-app-nodeport` | NodePort | Default internal access | `service.type=NodePort` (default) |
| `hello-app` | LoadBalancer | Main external access | `service.type=LoadBalancer` |
| `hello-app-lb` | LoadBalancer | Additional external access | `loadBalancer.enabled=true` |

### Service Configuration

The chart supports different service types that can be configured in `values.yaml`:

```yaml
service:
  type: NodePort  # Options: NodePort, LoadBalancer, ClusterIP
  name: hello-app-nodeport
  port: 80
  targetPort: 8080
  nodePort: 30000

loadBalancer:
  enabled: false  # Set to true to create additional LoadBalancer service
  name: hello-app-lb
  port: 80
  targetPort: 8080

# Main service names
mainService:
  nodeport: hello-app-nodeport
  loadbalancer: hello-app
```

**Installation Examples:**
```sh
# Install with default NodePort service (hello-app-nodeport)
$ helm install hello-app .

# Install with LoadBalancer service type (hello-app)
$ helm install hello-app . --set service.type=LoadBalancer

# Install with both NodePort and additional LoadBalancer services
$ helm install hello-app . --set loadBalancer.enabled=true

# Install with LoadBalancer service type and additional LoadBalancer
$ helm install hello-app . --set service.type=LoadBalancer --set loadBalancer.enabled=true
```

## Additional Information

- **Dependencies**: hello-app has no external dependencies and is designed to be lightweight and easy to deploy.

### Troubleshooting

**Common Commands:**
```sh
# Check all services
$ kubectl get services -n hello-app

# Check pods status
$ kubectl get pods -n hello-app

# Check service endpoints
$ kubectl get endpoints -n hello-app

# Describe service for details
$ kubectl describe service hello-app-nodeport -n hello-app
$ kubectl describe service hello-app -n hello-app
$ kubectl describe service hello-app-lb -n hello-app

# Check logs
$ kubectl logs -l app=hello-app -n hello-app

# Test connectivity from within cluster
$ kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://hello-app-nodeport:80
```

**Common Issues:**
- If you can't access the service, check if pods are running: `kubectl get pods -n hello-app`
- If LoadBalancer external IP is pending, wait for cloud provider to assign IP
- If port forwarding fails, ensure the service name is correct (hello-app-nodeport vs hello-app)

- **Uninstallation**: To remove hello-app from your cluster, you can use the `helm uninstall` command followed by the release name. For example:
  ```sh
  $ helm uninstall my-hello-app
  ```
