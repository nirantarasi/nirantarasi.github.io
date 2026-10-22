---
url: /docs/omniroute-operator.md
description: >-
  A Kubebuilder operator that deploys and manages OmniRoute — the unified AI
  proxy/router — on self-hosted Kubernetes.
---

# omniroute-operator

`omniroute-operator` is a Kubernetes operator built with
[Kubebuilder](https://kubebuilder.io/) that deploys and manages
[OmniRoute](https://github.com/diegosouzapw/OmniRoute), a unified AI
proxy/router, in self-hosted Kubernetes environments.

The operator watches `OmniRoute` custom resources and reconciles Deployments,
Services, PersistentVolumeClaims, and Ingresses, so you get a managed OmniRoute
instance from a single `kubectl apply`.

## Features

* Declarative lifecycle via the `gateway.omniroute.ai/v1alpha1` CRD.
* Automatic Deployment, Service, PVC, and Ingress reconciliation.
* Credentials injected from existing Secrets.
* Configuration injected from an existing ConfigMap with `envFrom`.
* Auto-derived `NODE_OPTIONS` from memory limits to prevent OOM kills.
* Configurable liveness, readiness, and startup probes.
* Redis-backed rate limiting, from a URL or a Secret.
* Database encryption at rest with `STORAGE_ENCRYPTION_KEY`.
* Ingress with TLS, `className`, and annotations.
* Reverse-proxy support through `basePath`.
* Service annotations for cloud load balancers.
* Persistence for OmniRoute's SQLite data directory.
* Leader election for highly available operator deployments.
* Multi-arch images (`linux/amd64`, `linux/arm64`) and a Helm chart.

## Prerequisites

| Tool | Version |
|---|---|
| Kubernetes | >= 1.28 |
| Helm | >= 3.12 |
| kubectl | matching your cluster |

For local development you also need k3d >= 5.6, Docker >= 24, and Go 1.27.

## How it works

1. You install the operator and its CRD into `omniroute-system`.
2. You create a Secret with OmniRoute's credentials and, optionally, a
   ConfigMap with environment settings.
3. You apply an `OmniRoute` resource in your application namespace.
4. The operator creates the Deployment, Service, PVC, and (when enabled) the
   Ingress, then reports readiness on the resource status.

## Deploy a first instance

```sh
kubectl create secret generic omniroute-secrets \
  --namespace ai-platform \
  --from-literal=JWT_SECRET="$(openssl rand -base64 48)" \
  --from-literal=API_KEY_SECRET="$(openssl rand -hex 32)" \
  --from-literal=INITIAL_PASSWORD="changeme"

kubectl apply -f config/samples/gateway_v1alpha1_omniroute.yaml
kubectl wait --for=condition=available deployment/omniroute-sample \
  --namespace ai-platform --timeout=120s
```

Then reach the UI:

```sh
kubectl port-forward svc/omniroute-sample 20128:20128 -n ai-platform
```

Open `http://localhost:20128` and log in with the password from
`INITIAL_PASSWORD`.

## Next steps

* [Installation](./installation.md): install the operator with Helm.
* [OmniRoute CR](./omniroute-cr.md): every field in the custom resource.
* [Local development](./local-development.md): a k3d cluster on your machine.
