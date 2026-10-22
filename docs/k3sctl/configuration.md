---
url: /docs/k3sctl/configuration.md
description: >-
  The k3sctl Cluster file — hosts, roles, SSH access, K3s settings, drain
  options, and additional manifests.
---

# Configuration

A `k3sctl` configuration is a single YAML document with
`apiVersion: k3sctl.k3sproject.io/v1beta1` and `kind: Cluster`.

```yaml
apiVersion: k3sctl.k3sproject.io/v1beta1
kind: Cluster
metadata:
  name: demo
spec:
  hosts:
    - role: server
      ssh:
        address: 10.0.0.1
        user: root
        port: 22
        keyPath: ~/.ssh/id_ed25519
      privateAddress: 10.0.0.1        # advertises this IP to the cluster
      installFlags:
        - --disable=traefik
      labels:
        node-role.example.com/edge: "true"
      taints:
        - dedicated=edge:NoSchedule
      environment:
        ETCD_UNSUPPORTED_ARCH: arm     # rendered into the systemd unit
      files:
        - src: ./configs/registries.yaml
          dst: /etc/rancher/k3s/registries.yaml
          perm: "0600"
      hooks:
        apply:
          before:
            - echo preparing
          after:
            - echo done

    - role: agent
      ssh:
        address: 10.0.0.2
        user: root
        port: 22
        keyPath: ~/.ssh/id_ed25519

  k3s:
    version: v1.30.0+k3s1
    config:                            # merged into config.yaml on every host
      write-kubeconfig-mode: "0644"
      disable:
        - servicelb
    extraArgs:
      - --kubelet-arg=eviction-hard=memory.available<100Mi

  options:
    drain:
      enabled: true
      timeout: 5m
      gracePeriod: -1
      ignoreDaemonSets: true
      deleteEmptyDirData: true
```

## Hosts

`spec.hosts` is a list. Each entry describes one machine and the role it plays.

| Key | Description |
|---|---|
| `role` | `server` (default) or `agent` |
| `ssh` / `openSSH` / `localhost` | How to reach the host |
| `privateAddress` | Address used for the node IP and API advertisements |
| `installFlags` | Extra `k3s server` / `k3s agent` command-line flags |
| `labels`, `taints` | Convenience shortcuts for `--node-label` and `--node-taint` |
| `environment` | Environment variables written to the host and the systemd unit |
| `files` | Files to upload, from a local path or a URL |
| `hooks` | Commands to run before or after `apply` |
| `dataDir` | Overrides the K3s data directory |
| `uploadBinary` | Upload a locally built binary instead of downloading one |
| `k3sBinaryPath` | Use an existing binary already on the host |
| `noFlannel` | Disable the flannel CNI |
| `os` | Override OS detection |

### SSH access

The `ssh` block takes an `address`, `user`, `port`, and `keyPath`. Use
`openSSH` to reuse your OpenSSH client configuration, or `localhost` when the
host is the machine running `k3sctl`.

### Files and hooks

`files` entries copy a local path or download a URL to `dst` on the host. Use
`perm` to set the mode. `hooks.apply.before` and `hooks.apply.after` run shell
commands around the apply step, which is useful for pre-flight checks and
post-install verification.

## K3s

`spec.k3s` controls the K3s installation itself.

| Key | Description |
|---|---|
| `version` | K3s version to install, for example `v1.30.0+k3s1` |
| `config` | Mapping merged into `config.yaml` on every host |
| `extraArgs` | Extra arguments passed to the K3s service |

Binary sources can also be a URL, a local file, or an existing installation on
the host; see the host-level `uploadBinary` and `k3sBinaryPath` options above.

## Drain options

`spec.options.drain` controls how nodes are drained during upgrades and
reconfiguration.

```yaml
spec:
  options:
    drain:
      enabled: true
      timeout: 5m
      gracePeriod: -1
      ignoreDaemonSets: true
      deleteEmptyDirData: true
```

## Additional manifests

Any other YAML documents in the configuration file are applied to the cluster
after installation. Separate them with `---`:

```yaml
# ... Cluster document ...
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: example
data:
  key: value
```

Each document must be a valid Kubernetes manifest. `k3sctl` passes them to
`kubectl apply` on the server node.
