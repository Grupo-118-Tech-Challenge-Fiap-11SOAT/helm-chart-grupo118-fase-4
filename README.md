# Helm Chart - Grupo 118 Fase 4

[![Helm Chart Pipeline](https://github.com/yourusername/helm-chart-grupo118-fase-4/actions/workflows/publish-helm-chart.yml/badge.svg)](https://github.com/yourusername/helm-chart-grupo118-fase-4/actions/workflows/publish-helm-chart.yml)

A production-ready Helm chart for deploying .NET 8 APIs on Kubernetes with Azure Container Registry (ACR) integration, internal ingress for API Gateway, Horizontal Pod Autoscaler (HPA), and support for secrets and configmaps.

## 📋 Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Installation Methods](#installation-methods)
- [Configuration](#configuration)
- [Usage Examples](#usage-examples)
- [CI/CD Integration](#cicd-integration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## ✨ Features

- 🐳 **ACR Integration**: Automatic Azure Container Registry credentials management
- 🚀 **Auto-scaling**: HPA with CPU and memory-based scaling
- 🔒 **Security**: Generic secrets and configmaps for environment variables
- 🌐 **Internal Ingress**: NGINX ingress for API Gateway integration
- 💚 **Health Checks**: Liveness and readiness probes for .NET 8 APIs
- 📊 **Resource Management**: Configurable CPU and memory limits/requests
- 🔄 **Multi-service Support**: Support for multiple service instances

## 📦 Prerequisites

Before using this Helm chart, ensure you have:

- Kubernetes cluster 1.19+
- Helm 3.0+
- `kubectl` configured to communicate with your cluster
- Azure Container Registry (if pulling from ACR)
- NGINX Ingress Controller (if using ingress features)
- Metrics Server (if using HPA)

### Install Prerequisites

```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Install Metrics Server (for HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## 🚀 Quick Start

### Option 1: Install from Local Chart

```bash
# Clone the repository
git clone https://github.com/yourusername/helm-chart-grupo118-fase-4.git
cd helm-chart-grupo118-fase-4

# Install the chart with default values
helm install my-api ./helm/grupo118fase4

# Install with custom namespace
helm install my-api ./helm/grupo118fase4 --namespace my-namespace --create-namespace
```

### Option 2: Install from ACR (OCI Registry)

```bash
# Login to ACR
helm registry login your-acr-name.azurecr.io \
  --username your-username \
  --password your-password

# Install from ACR
helm install my-api oci://your-acr-name.azurecr.io/helm/grupo118fase4 --version 1.0.0
```

## 📝 Installation Methods

### Basic Installation

```bash
helm install my-api ./helm/grupo118fase4
```

### Installation with Custom Values File

Create a `custom-values.yaml` file:

```yaml
image:
  repository: myacr.azurecr.io
  imageName: my-dotnet-api
  tag: "v1.0.0"

acrCredentials:
  enabled: true
  registry: myacr.azurecr.io
  username: "myacr"
  password: "your-password"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
```

Install with custom values:

```bash
helm install my-api ./helm/grupo118fase4 -f custom-values.yaml
```

### Installation with Inline Parameters

```bash
helm install my-api ./helm/grupo118fase4 \
  --set image.repository=myacr.azurecr.io \
  --set image.imageName=my-dotnet-api \
  --set image.tag=v1.0.0 \
  --set acrCredentials.username=myacr \
  --set acrCredentials.password=mypassword \
  --set autoscaling.minReplicas=3
```

## ⚙️ Configuration

### Core Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas (ignored if HPA enabled) | `1` |
| `image.repository` | Container registry URL | `your-acr-name.azurecr.io` |
| `image.imageName` | Image name | `your-api-name` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### ACR Configuration

```yaml
acrCredentials:
  enabled: true
  registry: your-acr-name.azurecr.io
  username: "your-username"
  password: "your-password"
```

### Service Configuration

```yaml
service:
  instances:
    - name: grupo118fase4-service
      type: ClusterIP
      port: 80
      targetPort: 8080
      protocol: TCP
      labels: {}
      annotations: {}
```

### Internal Ingress Configuration

```yaml
internalIngress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: api-internal.local
      paths:
        - path: /
          pathType: Prefix
          serviceName: grupo118fase4-service
          servicePort: 80
```

### Autoscaling (HPA)

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

### Secrets Configuration

```yaml
secret:
  enabled: true
  name: "my-api-secrets"  # Optional, defaults to release fullname
  envVars:
    - name: DATABASE_PASSWORD
      secretKey: db-password
    - name: API_KEY
      secretKey: api-key
  data:
    db-password: "my-secret-password"
    api-key: "my-api-key"
```

### ConfigMap Configuration

```yaml
configMap:
  enabled: true
  name: "my-api-config"  # Optional, defaults to release fullname
  envVars:
    - name: DATABASE_HOST
      configMapKey: db-host
    - name: LOG_LEVEL
      configMapKey: log-level
  data:
    db-host: "postgresql.database.svc.cluster.local"
    log-level: "Information"
```

### Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### Health Probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

## 📚 Usage Examples

### Example 1: Payment API

Create `payment-api-values.yaml`:

```yaml
image:
  repository: myacr.azurecr.io
  imageName: payment-api
  tag: "1.0.0"

acrCredentials:
  enabled: true
  registry: myacr.azurecr.io
  username: "myacr"
  password: "${ACR_PASSWORD}"

service:
  instances:
    - name: payment-api-service
      type: ClusterIP
      port: 80
      targetPort: 8080

internalIngress:
  enabled: true
  hosts:
    - host: payment-api.internal
      paths:
        - path: /api/payment
          pathType: Prefix
          serviceName: payment-api-service
          servicePort: 80

secret:
  enabled: true
  envVars:
    - name: PAYMENT_GATEWAY_KEY
      secretKey: gateway-key
    - name: DATABASE_PASSWORD
      secretKey: db-password
  data:
    gateway-key: "your-payment-gateway-key"
    db-password: "your-database-password"

configMap:
  enabled: true
  envVars:
    - name: PAYMENT_TIMEOUT_SECONDS
      configMapKey: timeout
  data:
    timeout: "30"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

Install:

```bash
helm install payment-api ./helm/grupo118fase4 -f payment-api-values.yaml
```

### Example 2: Order API

Create `order-api-values.yaml`:

```yaml
image:
  repository: myacr.azurecr.io
  imageName: order-api
  tag: "2.0.0"

service:
  instances:
    - name: order-api-service
      type: ClusterIP
      port: 80
      targetPort: 8080

internalIngress:
  enabled: true
  hosts:
    - host: order-api.internal
      paths:
        - path: /api/orders
          pathType: Prefix
          serviceName: order-api-service
          servicePort: 80

configMap:
  enabled: true
  envVars:
    - name: ORDER_TIMEOUT_SECONDS
      configMapKey: timeout
    - name: MAX_ORDER_ITEMS
      configMapKey: max-items
  data:
    timeout: "300"
    max-items: "100"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 15
```

Install:

```bash
helm install order-api ./helm/grupo118fase4 -f order-api-values.yaml
```

## 🔄 Managing Deployments

### Upgrade an Installation

```bash
# Upgrade with new values
helm upgrade my-api ./helm/grupo118fase4 -f custom-values.yaml

# Upgrade with specific values
helm upgrade my-api ./helm/grupo118fase4 --set image.tag=v2.0.0
```

### Rollback a Deployment

```bash
# List revisions
helm history my-api

# Rollback to previous version
helm rollback my-api

# Rollback to specific revision
helm rollback my-api 2
```

### Uninstall

```bash
helm uninstall my-api

# Uninstall from specific namespace
helm uninstall my-api --namespace my-namespace
```

### View Deployed Values

```bash
# Get all values
helm get values my-api

# Get all values including defaults
helm get values my-api --all
```

## 🔧 CI/CD Integration

This repository includes a GitHub Actions workflow ([`.github/workflows/publish-helm-chart.yml`](.github/workflows/publish-helm-chart.yml)) that automatically packages and publishes the Helm chart to ACR.

### GitHub Actions Workflow

The workflow:
- Triggers on push to `main` or `develop` branches
- Packages the Helm chart with versioning
- Pushes the chart to Azure Container Registry

### Required Secrets

Configure these secrets in your GitHub repository:

```
DOCKER_REGISTRY: your-acr-name.azurecr.io
ACR_USERNAME: your-acr-username
ACR_PASSWORD: your-acr-password
```

### Using Published Charts

```bash
# Login to ACR
helm registry login your-acr-name.azurecr.io \
  --username your-username \
  --password your-password

# Pull the chart
helm pull oci://your-acr-name.azurecr.io/helm/grupo118fase4 --version 1.0.X

# Install from ACR
helm install my-api oci://your-acr-name.azurecr.io/helm/grupo118fase4 --version 1.0.X
```

## 🐛 Troubleshooting

### Check Deployment Status

```bash
# View pods
kubectl get pods

# Describe pod
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# View events
kubectl get events --sort-by='.lastTimestamp'
```

### Common Issues

#### Image Pull Errors

**Problem**: `ImagePullBackOff` or `ErrImagePull`

**Solution**: Verify ACR credentials:

```bash
kubectl get secret acr-secret -o yaml
```

Ensure [`acrCredentials`](helm/grupo118fase4/values.yaml) are correctly configured.

#### HPA Not Scaling

**Problem**: HPA shows `<unknown>` for metrics

**Solution**: Verify metrics-server is running:

```bash
kubectl get deployment metrics-server -n kube-system
```

#### Ingress Not Working

**Problem**: Cannot access the API through ingress

**Solution**: 
1. Check ingress controller is running
2. Verify ingress resource: `kubectl get ingress`
3. Check ingress class matches: [`internalIngress.className`](helm/grupo118fase4/values.yaml)

### Debug Mode

```bash
# Dry run to see generated manifests
helm install my-api ./helm/grupo118fase4 --dry-run --debug

# Template without installing
helm template my-api ./helm/grupo118fase4
```

## 📖 Additional Resources

- [Helm Documentation](helm/grupo118fase4/README.md) - Detailed chart documentation
- [Chart.yaml](helm/grupo118fase4/Chart.yaml) - Chart metadata
- [values.yaml](helm/grupo118fase4/values.yaml) - Default values and configuration

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This Helm chart is provided as-is for Grupo 118 Tech Challenge Fase 4.

## 🆘 Support

For issues and questions:
- Open an issue in this repository
- Contact the Grupo 118 team

---

**Grupo 118 - Tech Challenge Fase 4**