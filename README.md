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
   $ helm install my-hello-app sysintelligent/hello-app --version 1.0.0
   ```

Once installed, you can test the application by sending an HTTP request to its endpoint:

```sh
$ curl http://<cluster-node-ip>:30000
```
This should return a response similar to: `hello-app says hi! [version: 1.0.0]`

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
   ```

3. Send an HTTP request to the application's endpoint:
   ```sh
   $ curl $(minikube ip):30000
   ```

## Additional Information

- **Dependencies**: hello-app has no external dependencies and is designed to be lightweight and easy to deploy.
- **Troubleshooting**: If you encounter any issues during installation or testing, please refer to the Helm documentation or Kubernetes troubleshooting guides for assistance.
- **Uninstallation**: To remove hello-app from your cluster, you can use the `helm uninstall` command followed by the release name. For example:
  ```sh
  $ helm uninstall my-hello-app
  ```
