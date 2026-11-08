---
url: /docs/k3sctl/commands.md
description: The k3sctl CLI — apply, reset, kubeconfig, restore, config, init, and more.
---

# Commands

| Command | Description |
|---|---|
| `k3sctl apply` | Install or update the cluster |
| `k3sctl reset` | Remove K3s from the hosts |
| `k3sctl kubeconfig` | Print the admin kubeconfig |
| `k3sctl restore` | Restore the cluster from an etcd snapshot |
| `k3sctl config diff` | Show differences between desired config and actual cluster state |
| `k3sctl config validate` | Validate a cluster configuration file |
| `k3sctl init` | Print a configuration template |
| `k3sctl version` | Print version information |
| `k3sctl completion` | Print a shell completion script |

## Global flags

| Flag | Description |
|---|---|
| `--debug` | Enable debug logging |
| `--trace` | Enable trace logging |
| `--no-redact` | Do not redact sensitive values from output |

## apply

`apply` reconciles the cluster with the configuration file. It is idempotent:
re-running it only changes what drifted.

```sh
k3sctl apply --config k3sctl.yaml
```

| Flag | Description |
|---|---|
| `--dry-run` | Show what would change without changing it |
| `--no-wait` | Do not wait for the cluster to become ready |
| `--no-drain` | Skip node draining during the operation |
| `--restore-from` | Restore an etcd snapshot before applying |
| `--kubeconfig-out` | Write the admin kubeconfig to a file |
| `--kubeconfig-api-address` | Override the API address in the kubeconfig |
| `--kubeconfig-user` | Override the user in the kubeconfig |
| `--kubeconfig-cluster` | Override the cluster name in the kubeconfig |
| `--concurrency` | Number of hosts to operate on concurrently |
| `--concurrent-uploads` | Number of concurrent file uploads |
| `--disable-downgrade-check` | Allow installing an older K3s version |
| `--force` | Skip confirmation prompts |

## restore

`restore` stops K3s, resets etcd state, and restores the cluster from a
snapshot. It prompts for confirmation unless `--force` is given.

```sh
k3sctl restore --config k3sctl.yaml --from /path/to/snapshot.db
```

| Flag | Description |
|---|---|
| `--from` / `-f` | Path to the etcd snapshot to restore from (required) |
| `--force` | Skip the confirmation prompt |

## config

`config` groups configuration-only commands that do not change the cluster.

### diff

Preview what `apply` would change by comparing the desired configuration with
the actual cluster state.

```sh
k3sctl config diff --config k3sctl.yaml
```

### validate

Validate a configuration file without connecting to any host.

```sh
k3sctl config validate --config k3sctl.yaml
```

## Common workflows

### Generate a config

Single-node localhost (no SSH):

```sh
k3sctl init --localhost --name dev > k3sctl.yaml
```

Multi-node with a server and an agent:

```sh
k3sctl init --name demo --address 10.0.0.1 --agent --agent-address 10.0.0.2 > k3sctl.yaml
```

When `version` is left empty in the generated config, `k3sctl` automatically
resolves the latest stable K3s release.

### Create a cluster

```sh
k3sctl apply --config k3sctl.yaml
k3sctl kubeconfig --config k3sctl.yaml > kubeconfig.yaml
export KUBECONFIG="$PWD/kubeconfig.yaml"
kubectl get nodes
```

`k3sctl` handles `sudo` internally — do not prefix commands with `sudo`.

### Validate and preview before applying

```sh
k3sctl config validate --config k3sctl.yaml
k3sctl config diff --config k3sctl.yaml
k3sctl apply --config k3sctl.yaml --dry-run
```

### Upgrade K3s

Edit the `version` in `k3sctl.yaml`, then:

```sh
k3sctl config diff --config k3sctl.yaml
k3sctl apply --config k3sctl.yaml
```

`k3sctl` performs a rolling upgrade with drain/cordon.

### Restore etcd

```sh
k3sctl restore --config k3sctl.yaml --from /path/to/snapshot.db
```

Snapshots are scheduled with the `etcdSnapshots` configuration block; see
[Configuration](./configuration.md) for scheduling, S3 upload, and pruning.

### Tear the cluster down

```sh
k3sctl reset --config k3sctl.yaml
```

For step-by-step guides covering single-node, multi-node, and HA topologies,
see [Quick Start](./quick-start.md).

## version and completion

```sh
k3sctl version
k3sctl completion bash
k3sctl completion zsh
```
