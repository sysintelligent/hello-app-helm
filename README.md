# hello-app

This is merely a test application that can be rapidly deployed to a Kubernetes cluster to perform some validation.

## Usage

Run command below to install the chart:

```sh
$ helm repo add sysintelligent https://sysintelligent.github.io/hello-app-helm/
$ helm install hello-app sysintelligent/hello-app --version 1.0.0
```

Test it

```sh
$ curl http://localhost:30000
hello-app says hi! [version: 1.0.0]