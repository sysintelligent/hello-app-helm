# Hello App Helm Chart

A lightweight Kubernetes application for testing and validating cluster deployments with multiple access methods.

## Overview

Hello App is a simple HTTP service designed for rapid deployment and validation on Kubernetes clusters. It responds to HTTP requests with a friendly greeting message along with its version number.

**Key Features:**
- Lightweight container (sysintelligent/hello-app:2.0.1)
- Multiple access methods (NodePort, LoadBalancer, Port Forwarding, Istio VirtualService)
- Dedicated namespace isolation (hello-app)
- RBAC security configuration
- Resource-constrained deployment (50m CPU, 20Mi memory)
- Easy configuration through Helm values
- **Verified Istio/Kyma Integration**: Successfully tested with Kyma cluster domain routing

**Chart Version:** 2.0.1  
**App Version:** 2.0.1

## Installation

### Prerequisites
- Kubernetes cluster (1.16+)
- Helm 3.x installed
- kubectl configured for your cluster

**Note:** To connect to your cluster before installation, set the kubeconfig:
```bash
export KUBECONFIG=/path/to/your/kubeconfig.yaml
# Verify cluster connection
kubectl cluster-info
```

### Quick Start

1. **Clone or download the chart:**
   ```bash
   git clone <repository-url>
   cd hello-app-helm
   ```

2. **Install with default configuration:**
   ```bash
   helm install hello-app .
   ```

3. **Install from remote repository:**
   ```bash
   helm repo add sysintelligent https://sysintelligent.github.io/hello-app-helm/
   helm install hello-app sysintelligent/hello-app --version 2.0.1
   ```

4. **Uninstall the chart (when done):**
   ```bash
   helm uninstall hello-app
   ```

### Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `sysintelligent/hello-app` |
| `image.tag` | Container image tag | `2.0.1` |
| `service.type` | Primary service type | `NodePort` |
| `service.nodePort` | NodePort port number | `30000` |
| `loadBalancer.enabled` | Enable additional LoadBalancer service | `true` |
| `virtualService.enabled` | Enable Istio VirtualService for domain routing | `false` |
| `virtualService.domain` | Cluster domain for VirtualService | `c-3bc6feb.stage.kyma.ondemand.com` |
| `resources.limits.cpu` | CPU limit | `50m` |
| `resources.limits.memory` | Memory limit | `20Mi` |

## External Access Methods

After installation, you can access the hello-app using multiple methods:

### 1. NodePort Access (Default)

The chart creates a NodePort service (`hello-app-nodeport`) accessible on port 30000:

```bash
# Get node IPs
kubectl get nodes -o wide

# Access via any node IP
curl http://<node-ip>:30000
```

**Example:**
```bash
curl http://10.250.0.133:30000
# Response: hello-app says hi! [version: 2.0.1]
```

### 2. LoadBalancer Access (Production)

By default, a LoadBalancer service (`hello-app-lb`) is created alongside the NodePort service:

```bash
# Check LoadBalancer status
kubectl get service hello-app-lb -n hello-app

# Wait for external IP assignment
kubectl get service hello-app-lb -n hello-app -w

# Access via external IP
curl http://<external-ip>
```

### 3. Port Forwarding (Development)

For local development and testing:

```bash
# Forward local port to service
kubectl port-forward service/hello-app-nodeport -n hello-app 8080:80

# Access locally
curl http://localhost:8080
```

**Note:** Port forwarding may not work in all cluster environments due to network policies or firewall restrictions. If port forwarding fails, use LoadBalancer or VirtualService access methods instead.

### 4. Istio VirtualService Access (Production with Domain)

When `virtualService.enabled=true`, an Istio VirtualService is created for domain-based access:

#### **Getting Your Cluster Domain:**

```bash
# Method 1: From cluster configmap (recommended)
kubectl get configmap shoot-info -n kube-system -o jsonpath='{.data.domain}'

# Method 2: From Istio Gateway
kubectl get gateway kyma-gateway -n kyma-system -o jsonpath='{.spec.servers[0].hosts[0]}'

# Method 3: From existing VirtualServices
kubectl get virtualservice -A --no-headers | grep -o '[^"]*\.[^"]*\.stage\.kyma\.ondemand\.com'
```

#### **Installation with Domain:**

```bash
# Get your cluster domain
export CLUSTER_DOMAIN=$(kubectl get configmap shoot-info -n kube-system -o jsonpath='{.data.domain}')

# Install with VirtualService enabled (auto-discover domain)
helm install hello-app . --set virtualService.enabled=true --set virtualService.domain=$CLUSTER_DOMAIN

# Or manually specify the domain
helm install hello-app . --set virtualService.enabled=true --set virtualService.domain=c-3bc6feb.stage.kyma.ondemand.com
```

#### **Check and Access:**

```bash
# Check VirtualService status
kubectl get virtualservice -n hello-app

# Access via domain (Kyma cluster domain)
# HTTP (redirects to HTTPS)
curl http://hello-app.c-3bc6feb.stage.kyma.ondemand.com

# HTTPS (recommended)
curl -k https://hello-app.c-3bc6feb.stage.kyma.ondemand.com
```

**Note:** For Istio/Kyma clusters, the VirtualService routes traffic from the Istio Gateway to your application service. The domain pattern is `{app-name}.{cluster-domain}`.

### 5. Service Configuration Examples

**Multiple service types can be configured:**

```yaml
# values.yaml examples

# Default: Both NodePort and LoadBalancer (no changes needed)
service:
  type: NodePort
loadBalancer:
  enabled: true

# NodePort only
service:
  type: NodePort
loadBalancer:
  enabled: false

# LoadBalancer as primary service
service:
  type: LoadBalancer
loadBalancer:
  enabled: false

# With VirtualService (Production)
service:
  type: NodePort
loadBalancer:
  enabled: true
virtualService:
  enabled: true
  domain: c-3bc6feb.stage.kyma.ondemand.com  # Replace with your cluster domain

```
## Testing

### Basic Connectivity Test

1. **Verify deployment status:**
   ```bash
   kubectl get pods -n hello-app
   kubectl get services -n hello-app
   ```

2. **Test via NodePort:**
   ```bash
   curl http://<node-ip>:30000
   ```

3. **Test via LoadBalancer (if enabled):**
   ```bash
   kubectl get service hello-app-lb -n hello-app
   curl http://<external-ip>
   ```

4. **Test via port forwarding:**
   ```bash
   kubectl port-forward service/hello-app-nodeport -n hello-app 8080:80
   # In another terminal:
   curl http://localhost:8080
   ```

5. **Test via VirtualService (if enabled):**
   ```bash
   kubectl get virtualservice -n hello-app
   curl -k https://hello-app.c-3bc6feb.stage.kyma.ondemand.com
   ```

### Expected Response

All tests should return:
```
hello-app says hi! [version: 2.0.1]
```

### Internal Cluster Testing

Test connectivity from within the cluster:

```bash
# Create test pod
kubectl run test-pod --image=busybox --rm -it --restart=Never -n hello-app -- sh

# Inside the pod
wget -qO- http://hello-app-nodeport:80
# Or
wget -qO- http://hello-app-lb:80
```

### Load Testing

For performance testing:

```bash
# Simple load test
for i in {1..10}; do curl http://<service-endpoint> & done
```

### Comprehensive Testing Example

Here's a complete testing sequence that was successfully verified:

```bash
# 1. Install with VirtualService enabled
helm install hello-app . --set virtualService.enabled=true --set virtualService.domain=c-3bc6feb.stage.kyma.ondemand.com

# 2. Verify all resources
kubectl get all -n hello-app
kubectl get virtualservice -n hello-app

# 3. Test all access methods
# VirtualService (Primary)
curl -k https://hello-app.c-3bc6feb.stage.kyma.ondemand.com

# HTTP redirect
curl -I http://hello-app.c-3bc6feb.stage.kyma.ondemand.com

# LoadBalancer
kubectl get service hello-app-lb -n hello-app
curl http://<external-ip>

# Internal cluster access
kubectl run test-pod --image=busybox --rm -it --restart=Never -n hello-app -- wget -qO- http://hello-app-nodeport:80
```

**Expected Results:**
- All access methods return: `hello-app says hi! [version: 2.0.1]`
- HTTP requests redirect to HTTPS (301 status)
- VirtualService properly routes traffic through Istio Gateway

## Troubleshooting

### Common Issues and Solutions

#### 1. Pods Not Running

**Check pod status:**
```bash
kubectl get pods -n hello-app
kubectl describe pod <pod-name> -n hello-app
kubectl logs <pod-name> -n hello-app
```

**Common causes:**
- Image pull issues
- Resource constraints
- RBAC permission problems

#### 2. Service Not Accessible

**Check services:**
```bash
kubectl get services -n hello-app
kubectl describe service hello-app-nodeport -n hello-app
kubectl get endpoints -n hello-app
```

**Verify connectivity:**
```bash
# Check if service exists
kubectl get service hello-app-nodeport -n hello-app

# Check endpoint availability
kubectl get endpoints hello-app-nodeport -n hello-app
```

#### 3. LoadBalancer Pending

**Check LoadBalancer status:**
```bash
kubectl get service hello-app-lb -n hello-app -o wide
```

**Common causes:**
- Cloud provider doesn't support LoadBalancer
- Insufficient cloud provider quotas
- Network configuration issues

**Solutions:**
- Use NodePort or port forwarding instead
- Check cloud provider documentation
- Verify cluster LoadBalancer controller

#### 4. Port Forward Issues

**Check port forwarding:**
```bash
# Ensure correct service name
kubectl get services -n hello-app

# Use correct port mapping
kubectl port-forward service/hello-app-nodeport -n hello-app 8080:80

# Note: Port forwarding may not work if cluster has network policies
# Use LoadBalancer or VirtualService access instead
```

#### 5. RBAC Issues

**Check RBAC configuration:**
```bash
kubectl get serviceaccount hello-app-service-account -n hello-app
kubectl get role pod-reader -n hello-app
kubectl get rolebinding pod-reader-binding -n hello-app
```

#### 6. VirtualService Issues

**Check VirtualService:**
```bash
kubectl get virtualservice -n hello-app
kubectl describe virtualservice hello-app-virtualservice -n hello-app
```

**Common causes:**
- VirtualService not created (required for Istio/Kyma clusters)
- Incorrect host configuration
- Gateway not properly configured
- Domain validation errors (check for extra quotes in template)

**Solutions:**
- Ensure VirtualService is created when virtualService.enabled=true
- Verify domain matches your cluster domain pattern
- Check Istio Gateway configuration
- Use `kubectl get virtualservice -n hello-app -o yaml` to verify configuration

**Domain Troubleshooting:**
```bash
# Verify your cluster domain
kubectl get configmap shoot-info -n kube-system -o jsonpath='{.data.domain}'

# Check if domain is accessible
nslookup hello-app.c-3bc6feb.stage.kyma.ondemand.com

# Verify VirtualService host configuration
kubectl get virtualservice hello-app-virtualservice -n hello-app -o jsonpath='{.spec.hosts[0]}'
```

### Diagnostic Commands

**Complete health check:**
```bash
# Check all resources
kubectl get all -n hello-app

# Check events
kubectl get events -n hello-app --sort-by='.lastTimestamp'

# Check resource usage
kubectl top pods -n hello-app

# Check service endpoints
kubectl get endpoints -n hello-app

# Detailed service information
kubectl describe service hello-app-nodeport -n hello-app
kubectl describe service hello-app-lb -n hello-app

# Check VirtualService details
kubectl describe virtualservice hello-app-virtualservice -n hello-app
```

### Getting Help

If you continue to experience issues:

1. Check the [Kubernetes documentation](https://kubernetes.io/docs/)
2. Verify your cluster has sufficient resources
3. Ensure your kubeconfig is properly configured
4. Check firewall and network policies
5. Review cloud provider LoadBalancer requirements

**Support Information:**
- Chart Version: 2.0.1
- Kubernetes Version: 1.16+
- Container Image: sysintelligent/hello-app:2.0.1
- Cluster Domain: c-3bc6feb.stage.kyma.ondemand.com (discoverable via `kubectl get configmap shoot-info -n kube-system -o jsonpath='{.data.domain}'`)
- **Tested and Verified**: Istio VirtualService configuration works correctly with Kyma clusters
