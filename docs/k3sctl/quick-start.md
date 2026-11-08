---
url: /docs/k3sctl/quick-start.md
description: >-
  Step-by-step guides for single-node, multi-node, and high-availability K3s
  clusters with k3sctl.
---

# Quick Start

This guide walks through three cluster topologies — pick the one that fits
your use case.

| Topology | Hosts | Best for |
|---|---|---|
| [Single-node (localhost)](#single-node-localhost) | 1 | Development, testing, CI |
| [Multi-node (SSH)](#multi-node-ssh) | 2+ | Small production, staging |
| [High availability (HA)](#high-availability-ha) | 3+ servers | Production with etcd redundancy |

## Single-node (localhost)

Run a one-node K3s cluster on the machine where `k3sctl` is installed. No
SSH is needed — `k3sctl` runs commands directly on the local host.

### Prerequisites

* Linux machine with `k3sctl` [installed](./installation.md)
* Root access or passwordless `sudo`

### Steps

**1. Generate the config**

```sh
k3sctl init --localhost --name dev > k3sctl.yaml
```

This produces a minimal config with `localhost: true` and no SSH block:

```yaml
apiVersion: k3sctl.k3sproject.io/v1beta1
kind: Cluster
metadata:
  name: dev
spec:
  hosts:
  - localhost: true
    role: server
  k3s:
    version: ""
```

When `version` is left empty, `k3sctl` automatically resolves the latest
stable K3s release.

**2. Apply the cluster**

```sh
k3sctl apply --config k3sctl.yaml
```

`k3sctl` handles `sudo` internally — do not prefix the command with `sudo`.

**3. Get the kubeconfig**

```sh
k3sctl kubeconfig --config k3sctl.yaml > kubeconfig.yaml
export KUBECONFIG="$PWD/kubeconfig.yaml"
kubectl get nodes
```

**4. Tear down (when done)**

```sh
k3sctl reset --config k3sctl.yaml
```

***

## Multi-node (SSH)

A server node plus one or more agent (worker) nodes, connected over SSH.

### Prerequisites

* 2+ Linux hosts reachable over SSH
* SSH key-based authentication configured
* Root or passwordless `sudo` on each host

### Steps

**1. Generate the config**

```sh
k3sctl init \
  --name staging \
  --address 10.0.0.1 \
  --user root \
  --key-path ~/.ssh/id_ed25519 \
  --agent \
  --agent-address 10.0.0.2 \
  > k3sctl.yaml
```

This generates a config with one server and one agent:

```yaml
apiVersion: k3sctl.k3sproject.io/v1beta1
kind: Cluster
metadata:
  name: staging
spec:
  hosts:
  - role: server
    ssh:
      address: 10.0.0.1
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
  - role: agent
    ssh:
      address: 10.0.0.2
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
  k3s:
    version: ""
```

To add more agents, duplicate the agent block with different addresses:

```yaml
  - role: agent
    ssh:
      address: 10.0.0.3
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
```

**2. (Optional) Edit the config**

```sh
$EDITOR k3sctl.yaml
```

Common changes:

* Pin a K3s version: `version: "v1.31.0+k3s1"`
* Disable traefik: add `installFlags: ["--disable=traefik"]` to the server
* Add labels or taints to nodes

**3. Validate the config**

```sh
k3sctl config validate --config k3sctl.yaml
```

**4. Apply the cluster**

```sh
k3sctl apply --config k3sctl.yaml
```

**5. Get the kubeconfig**

```sh
k3sctl kubeconfig --config k3sctl.yaml > kubeconfig.yaml
export KUBECONFIG="$PWD/kubeconfig.yaml"
kubectl get nodes
```

You should see both the server and agent nodes in `Ready` state.

**6. Tear down (when done)**

```sh
k3sctl reset --config k3sctl.yaml
```

***

## High availability (HA)

Three or more server nodes with embedded etcd for fault tolerance. Agents
are optional — server nodes can also run workloads.

### Prerequisites

* 3+ Linux hosts for servers (use an odd number for etcd quorum: 3, 5, …)
* Optional additional hosts for agents
* SSH key-based authentication to all hosts
* Root or passwordless `sudo` on each host

### Steps

**1. Generate a starter config**

```sh
k3sctl init --name production --address 10.0.0.1 > k3sctl.yaml
```

**2. Edit the config for HA**

Open `k3sctl.yaml` and add the additional server and agent hosts:

```yaml
apiVersion: k3sctl.k3sproject.io/v1beta1
kind: Cluster
metadata:
  name: production
spec:
  hosts:
  - role: server
    ssh:
      address: 10.0.0.1
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
    installFlags:
      - --disable=traefik

  - role: server
    ssh:
      address: 10.0.0.2
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
    installFlags:
      - --disable=traefik

  - role: server
    ssh:
      address: 10.0.0.3
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519
    installFlags:
      - --disable=traefik

  - role: agent
    ssh:
      address: 10.0.0.11
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519

  - role: agent
    ssh:
      address: 10.0.0.12
      user: root
      port: 22
      keyPath: ~/.ssh/id_ed25519

  k3s:
    version: "v1.31.0+k3s1"

    etcdSnapshots:
      scheduled: true
      scheduleCron: "0 */6 * * *"
      retention: 5
      compress: true

    healthCheck:
      enabled: true
      timeout: 120s

  options:
    drain:
      enabled: true
      timeout: 5m
```

Key points for HA:

* Use 3 or 5 server nodes (odd numbers maintain etcd quorum)
* K3s automatically initializes embedded etcd when it sees multiple servers
* `k3sctl` joins servers sequentially in quorum-safe batches
* Enable `etcdSnapshots` for automated backups
* Enable `healthCheck` to verify all nodes are ready after apply
* Enable `drain` for safe rolling upgrades

**3. Validate and preview**

```sh
k3sctl config validate --config k3sctl.yaml
k3sctl apply --config k3sctl.yaml --dry-run
```

**4. Apply the cluster**

```sh
k3sctl apply --config k3sctl.yaml
```

`k3sctl` will:

1. Connect to each host over SSH
2. Install K3s on the first server and initialize etcd
3. Join additional servers to the etcd cluster
4. Join agent nodes to the cluster
5. Run health checks to verify the cluster is ready

**5. Get the kubeconfig**

```sh
k3sctl kubeconfig --config k3sctl.yaml > kubeconfig.yaml
export KUBECONFIG="$PWD/kubeconfig.yaml"
kubectl get nodes
```

All server and agent nodes should show `Ready`.

**6. Day-two operations**

Upgrade K3s by changing `version` and re-running apply — `k3sctl` performs
a rolling upgrade with drain/cordon:

```sh
# Edit k3sctl.yaml and change the version
k3sctl config diff --config k3sctl.yaml   # preview the changes
k3sctl apply --config k3sctl.yaml         # rolling upgrade
```

Restore from an etcd snapshot if something goes wrong:

```sh
k3sctl restore --config k3sctl.yaml --from /var/lib/rancher/k3s/server/db/snapshots/etcd-snapshot-xxx
```

**7. Tear down (when done)**

```sh
k3sctl reset --config k3sctl.yaml
```

***

## Next steps

* [Configuration](./configuration.md): full reference for every option
* [Commands](./commands.md): all CLI commands and flags
* [Installation](./installation.md): other ways to install `k3sctl`
