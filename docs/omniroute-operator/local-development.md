---
url: /docs/omniroute-operator/local-development.md
description: >-
  Run the omniroute-operator against a local k3d cluster, build images, and run
  the controller outside Kubernetes.
---

# Local development

You need Docker >= 24, k3d >= 5.6, Go 1.27, kubectl, and Helm >= 3.12.

## Build the operator

```sh
go build -trimpath -o bin/manager ./cmd/
./bin/manager --help
```

Cross-compile the Linux binaries used by the container image:

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -o dist/linux_amd64/manager ./cmd/
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -trimpath -o dist/linux_arm64/manager ./cmd/
```

## Run the tests

```sh
go test -short -timeout=120s ./...
go test -race -short -timeout=120s ./...
go vet ./...
```

Integration tests require Docker:

```sh
INTEGRATION=1 go test -v -timeout=10m -tags=integration ./tests/integration/
```

## Create a local cluster

```sh
k3d cluster create omniroute \
  --port "20128:20128@server:0" \
  --k3s-arg "--disable=traefik@server:0"

kubectl apply -k config/k3d/infrastructure   # namespaces
kubectl apply -f config/crd/bases/           # OmniRoute CRD
```

## Install the operator from source

Build the image the same way the Dockerfile expects, import it into k3d, then
install the chart with the local image:

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -o dist/linux_amd64/manager ./cmd/
docker build -t omniroute-operator:dev .
k3d image import omniroute-operator:dev -c omniroute

helm install omniroute-operator charts/omniroute-operator \
  --namespace omniroute-system \
  --set image.repository=omniroute-operator \
  --set image.tag=dev \
  --set image.pullPolicy=IfNotPresent
```

Check the controller:

```sh
kubectl get pods -n omniroute-system
kubectl get crd omniroutes.gateway.omniroute.ai
```

## Run the controller outside the cluster

Useful when you want to debug reconciliation in your IDE. Install the CRD, then
run the manager against your kubeconfig:

```sh
kubectl apply -f config/crd/bases/
./bin/manager --health-probe-bind-address=:8081
```

## Helm checks

```sh
helm lint charts/omniroute-operator --strict
helm template omniroute-operator charts/omniroute-operator \
  --namespace omniroute-system > /tmp/omniroute-operator.yaml
```

## Clean up

```sh
k3d cluster delete omniroute
```
