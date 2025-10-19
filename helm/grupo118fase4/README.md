# Grupo 118 Fase 4 - Helm Chart

A Helm chart for deploying .NET 8 APIs on Kubernetes with support for Azure Container Registry (ACR), internal ingress for API Gateway integration, HPA, and generic secrets/configmaps.

## Features

- **ACR Pull Secret**: Automatic configuration of Azure Container Registry credentials
- **Service**: Kubernetes service with configurable type (ClusterIP, LoadBalancer, etc.)
- **Internal Ingress**: NGINX ingress for API Gateway integration
- **HPA**: Horizontal Pod Autoscaler with CPU and memory-based scaling
- **Generic Secrets**: Easily configurable secrets for environment variables
- **Generic ConfigMaps**: Easily configurable config maps for environment variables
- **Health Probes**: Liveness and readiness probes configured
- **Resource Management**: CPU and memory limits/requests

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Azure Container Registry (if using ACR)
- NGINX Ingress Controller (if using internal ingress)

## Installation

### Basic Installation

```bash
helm install my-api ./helm
```

### Installation with Custom Values

```bash
helm install my-api ./helm -f custom-values.yaml
```

### Installation with Inline Values

```bash
helm install my-api ./helm \
  --set image.repository=myacr.azurecr.io \
  --set image.imageName=my-dotnet-api \
  --set image.tag=v1.0.0 \
  --set acrCredentials.username=myacr \
  --set acrCredentials.password=mypassword
```

## Configuration

### ACR Credentials

To enable ACR pull secret:

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
    - name: my-api-service
      type: ClusterIP
      port: 80
      targetPort: 8080
      protocol: TCP
```

### Internal Ingress for API Gateway

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
          serviceName: my-api-service
          servicePort: 80
```

### HPA Configuration

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

### Generic Secrets

```yaml
secret:
  enabled: true
  envVars:
    - name: DATABASE_PASSWORD
      secretKey: db-password
    - name: API_KEY
      secretKey: api-key
  data:
    db-password: "my-secret-password"
    api-key: "my-api-key"
```

### Generic ConfigMaps

```yaml
configMap:
  enabled: true
  envVars:
    - name: DATABASE_HOST
      configMapKey: db-host
    - name: LOG_LEVEL
      configMapKey: log-level
  data:
    db-host: "postgresql.database.svc.cluster.local"
    log-level: "Information"
```

### Health Probes

The chart includes configurable health probes for .NET 8 applications:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

## Example Values for Different APIs

### Payment API

```yaml
image:
  repository: myacr.azurecr.io
  imageName: payment-api
  tag: "1.0.0"

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
  data:
    gateway-key: "your-payment-gateway-key"
```

### Order API

```yaml
image:
  repository: myacr.azurecr.io
  imageName: order-api
  tag: "1.0.0"

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
  data:
    timeout: "300"
```

## Upgrade

```bash
helm upgrade my-api ./helm -f custom-values.yaml
```

## Uninstall

```bash
helm uninstall my-api
```

## Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container registry URL | `your-acr-name.azurecr.io` |
| `image.imageName` | Image name | `your-api-name` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `acrCredentials.enabled` | Enable ACR secret creation | `true` |
| `acrCredentials.registry` | ACR registry URL | `your-acr-name.azurecr.io` |
| `acrCredentials.username` | ACR username | `""` |
| `acrCredentials.password` | ACR password | `""` |
| `service.instances[0].type` | Service type | `ClusterIP` |
| `service.instances[0].port` | Service port | `80` |
| `service.instances[0].targetPort` | Container port | `8080` |
| `internalIngress.enabled` | Enable internal ingress | `true` |
| `internalIngress.className` | Ingress class name | `nginx` |
| `autoscaling.enabled` | Enable HPA | `true` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU % | `70` |
| `autoscaling.targetMemoryUtilizationPercentage` | Target Memory % | `80` |
| `secret.enabled` | Enable generic secret | `false` |
| `configMap.enabled` | Enable generic configmap | `false` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `250m` |
| `resources.requests.memory` | Memory request | `256Mi` |

## Notes

- Make sure your .NET 8 API exposes health check endpoints at `/health` and `/health/ready`
- The ACR credentials should be provided securely (e.g., using CI/CD secrets)
- Adjust resource limits based on your API's requirements
- The internal ingress is configured for use with an API Gateway
- HPA requires the metrics-server to be installed in your cluster

## License

This chart is provided as-is for Grupo 118 Tech Challenge Fase 4.
