---
url: /docs/k3sctl.md
description: >-
  Declarative lifecycle management for K3s clusters over SSH — install, upgrade,
  reconfigure, back up, and reset from one YAML file.
---

# k3sctl

`k3sctl` is a declarative lifecycle management tool for
[K3s](https://k3s.io) clusters, created and inspired by
[k0sctl](https://github.com/k0sproject/k0sctl). Point it at a YAML file
describing your hosts and it installs, upgrades, reconfigures, backs up, and
resets the cluster over SSH.

```sh
k3sctl init --name demo --address 10.0.0.1 --agent --agent-address 10.0.0.2 > k3sctl.yaml
$EDITOR k3sctl.yaml
k3sctl apply --config k3sctl.yaml
k3sctl kubeconfig > kubeconfig.yaml
```

## What it does

* Declarative cluster configuration with servers and agents in one file.
* Installs and upgrades K3s binaries from GitHub releases, a URL, a local file,
  or an existing installation on the host.
* Generates server and agent configuration plus systemd units.
* Supports labels, taints, extra install flags, environment variables, and
  arbitrary file uploads.
* Applies additional Kubernetes manifests bundled in the configuration file.
* Creates etcd snapshots and restores them, with etcd member validation.
* Fetches a ready-to-use admin kubeconfig.
* Idempotent `apply`: safe to re-run, and only changes what drifted.
* Dry-run mode with `--dry-run`.

::: warning
Linux hosts only. `k3sctl` drives the cluster over SSH, so each host must be
reachable and must provide `sudo` or a root login.
:::

## How it works

1. `k3sctl` reads the `Cluster` document from your config file.
2. It connects to each host over SSH and detects the OS and architecture.
3. It uploads or downloads the requested K3s version, writes the server or agent
   configuration and the systemd unit, then starts K3s.
4. Once the API server answers, it applies any additional manifests found in the
   same file with `kubectl apply`.
5. `k3sctl kubeconfig` reads the admin kubeconfig back so you can use it
   locally.

Because the desired state is the file, re-running `apply` after editing it is
the normal way to change a cluster.

## Requirements

* Linux hosts for servers and agents.
* SSH access, with a key or the agent available to `k3sctl`.
* Root or passwordless `sudo` on each host.

## Next steps

* [Installation](./installation.md): release binary, `go install`, or build from
  source.
* [Configuration](./configuration.md): every host, K3s, and drain option.
* [Commands](./commands.md): the full CLI and the flags accepted by `apply`.
