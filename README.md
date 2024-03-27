# Hello-App

This repository contains a test application that can be swiftly deployed to a Kubernetes cluster for validation purposes.

## Usage

To install the chart, run the following command:

```sh
$ helm repo add sysintelligent https://sysintelligent.github.io/hello-app-helm/
$ helm install my-hello-app sysintelligent/hello-app --version 1.0.0
```

To test it, execute:

```sh
$ curl http://<cluster-node-ip>:30000
hello-app says hi! [version: 1.0.0]
```

To test it locally with Minikube, use:

```
curl $(minikube ip):30000
```