---
url: /docs/omniroute-operator/omniroute-cr.md
description: Reference for the gateway.omniroute.ai/v1alpha1 OmniRoute custom resource.
---

# OmniRoute CR

An `OmniRoute` resource describes one OmniRoute deployment. The operator
reconciles a Deployment, Service, PVC, and optional Ingress from it.

```yaml
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
  credentials:
    existingSecret: omniroute-secrets
  config:
    existingConfigMap: omniroute-config
  persistence:
    enabled: true
    size: 1Gi
```

## Core

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.image.repository` | string | `diegosouzapw/omniroute` | OmniRoute container image |
| `spec.image.tag` | string | `3.8.51` | Image tag |
| `spec.image.pullPolicy` | string | `IfNotPresent` | Image pull policy |
| `spec.replicas` | int | `1` | Pod count (max 1 — SQLite is single-writer) |
| `spec.resources` | ResourceRequirements | — | CPU and memory requests and limits |
| `spec.env` | \[]EnvVar | — | Additional environment variables |

## Service

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.service.type` | string | `ClusterIP` | `ClusterIP`, `NodePort`, or `LoadBalancer` |
| `spec.service.port` | int | `20128` | Service port |
| `spec.service.annotations` | map\[string]string | — | Service annotations, for example for an AWS NLB |

## Configuration and credentials

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.config.existingConfigMap` | string | — | ConfigMap with OmniRoute environment variables (injected with `envFrom`) |
| `spec.credentials.existingSecret` | string | — | Secret with `JWT_SECRET`, `API_KEY_SECRET`, and `INITIAL_PASSWORD` |

## Persistence

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.persistence.enabled` | bool | `true` | Create a PVC for the data directory |
| `spec.persistence.size` | string | `1Gi` | PVC storage size |
| `spec.persistence.storageClass` | string | cluster default | StorageClass name |
| `spec.persistence.accessMode` | string | `ReadWriteOnce` | PVC access mode |

## Health probes

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.probes.livenessPath` | string | `/api/monitoring/health` | HTTP path for the liveness probe |
| `spec.probes.readinessPath` | string | `/api/monitoring/health` | HTTP path for the readiness probe |
| `spec.probes.startupPath` | string | `/api/monitoring/health` | HTTP path for the startup probe |

The operator fills in sensible timing defaults for all three probes.

## Ingress

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.ingress.enabled` | bool | `false` | Create an Ingress resource |
| `spec.ingress.className` | string | — | IngressClass name, such as `nginx` |
| `spec.ingress.host` | string | — | Hostname for the Ingress rule |
| `spec.ingress.tls.secretName` | string | — | TLS Secret name |
| `spec.ingress.annotations` | map\[string]string | — | Ingress annotations |

## Redis (rate limiting)

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.redis.url` | string | — | Redis connection URL, such as `redis://redis:6379` |
| `spec.redis.existingSecret` | string | — | Secret containing a `REDIS_URL` key |
| `spec.redis.keyPrefix` | string | — | Redis key namespace prefix |

## Encryption (database at rest)

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.encryption.existingSecret` | string | — | Secret containing `STORAGE_ENCRYPTION_KEY` |
| `spec.encryption.keyVersion` | string | — | Encryption key version, for key rotation |

## Reverse proxy

| Field | Type | Default | Description |
|---|---|---|---|
| `spec.basePath` | string | — | Sets `OMNIROUTE_BASE_PATH`, for example `/omniroute` |

## Auto-derived environment variables

The operator sets these from the resource, so you do not repeat them in `spec.env`:

| Variable | Source | Description |
|---|---|---|
| `NODE_OPTIONS` | `spec.resources.limits.memory` | `--max-old-space-size=<75% of the memory limit in MB>` to prevent Node.js OOM kills |
| `OMNIROUTE_BASE_PATH` | `spec.basePath` | Reverse-proxy base path |
| `REDIS_URL` | `spec.redis.url` or the referenced Secret | Redis connection URL |
| `REDIS_KEY_PREFIX` | `spec.redis.keyPrefix` | Redis key namespace prefix |
| `STORAGE_ENCRYPTION_KEY` | `spec.encryption.existingSecret` | Database encryption key |
| `STORAGE_ENCRYPTION_KEY_VERSION` | `spec.encryption.keyVersion` | Encryption key version |

## Status

| Field | Description |
|---|---|
| `status.conditions` | Standard conditions describing reconciliation |
| `status.readyReplicas` | Number of ready OmniRoute replicas |
| `status.observedGeneration` | Generation last processed by the operator |
| `status.phase` | High-level phase of the deployment |

Check it with:

```sh
kubectl get omniroute omniroute -n ai-platform -o yaml
```
