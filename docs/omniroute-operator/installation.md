---
url: /docs/omniroute-operator/installation.md
description: Install the omniroute-operator with Helm, then deploy an OmniRoute instance.
---

# Installation

The operator and the OmniRoute instances it manages are separate concerns. You
install the operator once per cluster; each application namespace then gets one
or more `OmniRoute` resources.

## Install the operator

From the OCI registry:

```sh
helm install omniroute-operator oci://ghcr.io/vinaycharlie01/helm-charts/omniroute-operator \
  --namespace omniroute-system --create-namespace
```

Or from the chart in the repository:

```sh
helm install omniroute-operator charts/omniroute-operator \
  --namespace omniroute-system --create-namespace
```

Verify the controller is running:

```sh
kubectl get pods -n omniroute-system
```

## Helm values

| Value | Default | Description |
|---|---|---|
| `replicaCount` | `1` | Operator pod replicas |
| `image.repository` | `ghcr.io/vinaycharlie01/omniroute-operator` | Operator image |
| `image.tag` | `""` (appVersion) | Operator image tag |
| `leaderElection.enabled` | `true` | Enable leader election |
| `resources.requests.cpu` | `10m` | CPU request |
| `resources.requests.memory` | `64Mi` | Memory request |
| `resources.limits.cpu` | `500m` | CPU limit |
| `resources.limits.memory` | `128Mi` | Memory limit |

## Create the OmniRoute secrets

OmniRoute requires three secrets: `JWT_SECRET`, `API_KEY_SECRET`, and
`INITIAL_PASSWORD`.

```sh
kubectl create secret generic omniroute-secrets \
  --namespace ai-platform \
  --from-literal=JWT_SECRET="$(openssl rand -base64 48)" \
  --from-literal=API_KEY_SECRET="$(openssl rand -hex 32)" \
  --from-literal=INITIAL_PASSWORD="changeme"
```

## Optional: environment configuration

All keys in the referenced ConfigMap are injected into the OmniRoute container
as environment variables with `envFrom`:

```sh
kubectl create configmap omniroute-config \
  --namespace ai-platform \
  --from-literal=PORT=20128 \
  --from-literal=APP_LOG_LEVEL=info \
  --from-literal=REQUIRE_API_KEY=false
```

## Deploy an instance

```sh
kubectl apply -f config/samples/gateway_v1alpha1_omniroute.yaml
```

Or apply a minimal resource inline:

```sh
kubectl apply -f - <<'EOF'
apiVersion: gateway.omniroute.ai/v1alpha1
kind: OmniRoute
metadata:
  name: omniroute
  namespace: ai-platform
spec:
  image:
    repository: diegosouzapw/omniroute
    tag: "3.8.51"
  replicas: 1
  service:
    type: ClusterIP
    port: 20128
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: "1"
      memory: 1Gi
  credentials:
    existingSecret: omniroute-secrets
  config:
    existingConfigMap: omniroute-config
  persistence:
    enabled: true
    size: 1Gi
EOF
```

Wait for the Deployment, then reach the UI:

```sh
kubectl wait --for=condition=available deployment/omniroute \
  --namespace ai-platform --timeout=120s

kubectl port-forward svc/omniroute 20128:20128 -n ai-platform
```

Open `http://localhost:20128` and log in with `INITIAL_PASSWORD`.

## Expose OmniRoute

The CR supports `ClusterIP`, `NodePort`, `LoadBalancer`, and Ingress. See
[OmniRoute CR](./omniroute-cr.md) for the Service and Ingress fields, and the
complete spec reference.

```yaml
spec:
  ingress:
    enabled: true
    className: nginx
    host: omniroute.example.com
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    tls:
      secretName: omniroute-tls
```

## Use the API

OmniRoute exposes an OpenAI-compatible API:

```sh
curl http://localhost:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Uninstall

```sh
kubectl delete omniroute omniroute -n ai-platform
helm uninstall omniroute-operator -n omniroute-system
```
