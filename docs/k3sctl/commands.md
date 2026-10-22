---
url: /docs/k3sctl/commands.md
description: The k3sctl CLI — apply, reset, kubeconfig, backup, config, init, and more.
---

# Commands

| Command | Description |
|---|---|
| `k3sctl apply` | Install or update the cluster |
| `k3sctl reset` | Remove K3s from the hosts |
| `k3sctl kubeconfig` | Print the admin kubeconfig |
| `k3sctl backup` | Create an etcd snapshot |
| `k3sctl config edit` | Edit the cluster configuration |
| `k3sctl config status` | Show the current cluster status |
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
| `--concurrency` | Number of hosts to operate on concurrently |
| `--concurrent-uploads` | Number of concurrent file uploads |
| `--disable-downgrade-check` | Allow installing an older K3s version |

## Common workflows

### Create a cluster

```sh
k3sctl apply --config k3sctl.yaml
k3sctl kubeconfig > kubeconfig.yaml
export KUBECONFIG="$PWD/kubeconfig.yaml"
kubectl get nodes
```

### Check before applying

```sh
k3sctl apply --config k3sctl.yaml --dry-run
```

### Back up and restore etcd

```sh
# Create a snapshot on the server
k3sctl backup --config k3sctl.yaml

# Restore from a snapshot while applying
k3sctl apply --config k3sctl.yaml --restore-from /path/to/snapshot.db
```

### Inspect and edit configuration

```sh
k3sctl config status
k3sctl config edit
```

### Generate a starter file

```sh
k3sctl init --name demo --address 10.0.0.1 --agent --agent-address 10.0.0.2 > k3sctl.yaml
```

### Tear the cluster down

```sh
k3sctl reset --config k3sctl.yaml
```

## version and completion

```sh
k3sctl version
k3sctl completion bash
```
