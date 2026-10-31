---
url: /docs/k3sctl/configuration.md
description: >-
  The k3sctl Cluster file — hosts, roles, SSH access, K3s settings, etcd
  snapshots, registries, health checks, and additional manifests.
---

# Configuration

A `k3sctl` configuration is a single `Cluster` document with
`apiVersion: k3sctl.k3sproject.io/v1beta1`.

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

    etcdSnapshots:                     # automated etcd snapshot scheduling
      scheduled: true
      scheduleCron: "0 */12 * * *"
      retention: 5
      dir: /var/lib/rancher/k3s/server/db/snapshots
      compress: true
      s3:
        enabled: true
        endpoint: s3.amazonaws.com
        bucket: my-k3s-backups
        region: us-east-1
        folder: snapshots
        accessKey: AKIAIOSFODNN7EXAMPLE
        secretKey: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
      prune:                           # prune old snapshots on apply
        enabled: true
        retention: 10
        name: on-demand

    secretsEncrypt:                    # secrets encryption at rest
      enabled: true
      rotateKeys: true                 # dynamically rotate encryption keys
      reencrypt: true                  # re-encrypt all secrets with new keys

    certRotate:                        # certificate rotation on apply
      enabled: true
      rotateCA: true                   # rotate the root CA (full cluster restart)

    token:                             # cluster tokens
      serverToken: my-server-secret
      agentToken: my-agent-secret
      rotate: true                     # rotate token and restart all nodes

    registries:                        # private registry configuration
      mirrors:
        docker.io:
          endpoint:
            - https://registry.example.com
      configs:
        registry.example.com:
          auth:
            username: myuser
            password: mypass
          tls:
            caFile: /path/to/ca.pem
            insecureSkipVerify: false

    healthCheck:                       # post-apply health verification
      enabled: true
      timeout: 120s

    preflight:                         # pre-flight checks before apply
      enabled: true
      checkCgroups: true
      checkModules: true
      checkPorts: true
      checkDisk: true
      minDiskGB: 10

    airgap:                            # air-gap (offline) installation
      enabled: true
      imageBundle: /path/to/k3s-airgap-images-amd64.tar.zst

  options:
    drain:
      enabled: true
      timeout: 5m
      gracePeriod: -1
      ignoreDaemonSets: true
      deleteEmptyDirData: true
      disableEviction: false
    concurrency:
      limit: 30
      uploads: 5
    evictTaint:
      enabled: false
```

## Hosts

`spec.hosts` is a list. Each entry describes one machine and the role it plays.

| Key | Description |
|---|---|
| `role` | `server` (default) or `agent` |
| `ssh` / `openSSH` / `localhost` | How to reach the host |
| `privateAddress` | Address used for the node IP and API advertisements |
| `privateInterface` | Network interface whose address is used for node IP and API advertisements |
| `hostname` | Override the host name reported to the cluster |
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
| `etcdSnapshots` | Automated etcd snapshot scheduling, S3 upload, and pruning |
| `secretsEncrypt` | Secrets encryption at rest with key rotation and re-encryption |
| `certRotate` | Rotate TLS certificates and optionally the root CA |
| `token` | Explicit server/agent join tokens and optional rotation |
| `registries` | Private registry mirrors and authentication |
| `healthCheck` | Post-apply cluster health verification |
| `preflight` | Pre-flight checks run before apply |
| `airgap` | Air-gap (offline) installation settings |

Binary sources can also be a URL, a local file, or an existing installation on
the host; see the host-level `uploadBinary` and `k3sBinaryPath` options above.

### etcdSnapshots

| Key | Description |
|---|---|
| `scheduled` | Enable scheduled snapshots |
| `scheduleCron` | Cron expression for the snapshot schedule |
| `retention` | Number of snapshots to retain |
| `dir` | Local directory for snapshots |
| `compress` | Compress snapshots |
| `s3.enabled` | Store snapshots in S3 |
| `s3.endpoint` | S3 endpoint |
| `s3.endpointCa` | Path to a CA certificate for the S3 endpoint |
| `s3.skipSslVerify` | Skip TLS verification for S3 |
| `s3.accessKey` | S3 access key |
| `s3.secretKey` | S3 secret key |
| `s3.bucket` | S3 bucket name (required when S3 is enabled) |
| `s3.region` | S3 region |
| `s3.folder` | Folder within the bucket |
| `s3.insecure` | Use HTTP instead of HTTPS |

#### etcdSnapshots.prune

| Key | Description |
|---|---|
| `enabled` | Enable pruning of old etcd snapshots on apply |
| `retention` | Number of snapshots to retain (required when enabled, must be greater than 0) |
| `name` | Snapshot name filter for pruning, for example `on-demand` |

### secretsEncrypt

| Key | Description |
|---|---|
| `enabled` | Enable K3s secrets encryption at rest |
| `rotateKeys` | Dynamically rotate encryption keys |
| `reencrypt` | Re-encrypt all secrets with the new keys |

### certRotate

| Key | Description |
|---|---|
| `enabled` | Rotate K3s TLS certificates on each server during `apply` |
| `rotateCA` | Rotate the root Certificate Authority; triggers a full cluster restart |

### token

| Key | Description |
|---|---|
| `serverToken` | Token used for server-to-server communication |
| `agentToken` | Token used for agent-to-server communication |
| `rotate` | Rotate the cluster join token and restart all nodes |

### registries

| Key | Description |
|---|---|
| `mirrors.<registry>.endpoint` | List of mirror endpoint URLs for the given registry |
| `configs.<registry>.auth.username` | Registry authentication username |
| `configs.<registry>.auth.password` | Registry authentication password |
| `configs.<registry>.auth.token` | Registry authentication token |
| `configs.<registry>.tls.certFile` | Path to a client TLS certificate |
| `configs.<registry>.tls.keyFile` | Path to a client TLS key |
| `configs.<registry>.tls.caFile` | Path to a CA certificate for the registry |
| `configs.<registry>.tls.insecureSkipVerify` | Skip TLS verification for the registry |

The registries configuration is written to `/etc/rancher/k3s/registries.yaml`
on all nodes during apply.

### healthCheck

| Key | Description |
|---|---|
| `enabled` | Enable post-apply cluster health verification |
| `timeout` | Maximum wait time for health checks, for example `120s` or `5m` |

When enabled, the health check verifies the API server responds to `/healthz`
and all nodes reach `Ready` status before the apply completes.

### preflight

| Key | Description |
|---|---|
| `enabled` | Enable pre-flight checks |
| `checkCgroups` | Verify cgroup v1/v2 controllers are available |
| `checkModules` | Verify required kernel modules are loaded |
| `checkPorts` | Verify required ports are not in use |
| `checkDisk` | Verify minimum disk space is available |
| `minDiskGB` | Minimum disk space in GB |

### airgap

| Key | Description |
|---|---|
| `enabled` | Enable air-gap mode (skip downloading K3s images from the internet) |
| `imageBundle` | Local path to a K3s air-gap image bundle, for example `k3s-airgap-images-amd64.tar.zst` |

When enabled, the bundle is uploaded to `<dataDir>/agent/images/` on every host.

## Options

`spec.options` controls how `k3sctl` performs disruptive operations.

### Drain

`drain` controls how nodes are drained during upgrades and reconfiguration.

```yaml
spec:
  options:
    drain:
      enabled: true
      timeout: 5m
      gracePeriod: -1
      ignoreDaemonSets: true
      deleteEmptyDirData: true
      disableEviction: false
```

| Key | Description |
|---|---|
| `enabled` | Drain nodes before restarting them (default `true`) |
| `timeout` | Drain timeout (default `5m`) |
| `gracePeriod` | Grace period passed to `kubectl drain` (default `-1`) |
| `ignoreDaemonSets` | Ignore DaemonSet pods when draining (default `true`) |
| `deleteEmptyDirData` | Delete `emptyDir` data when draining (default `true`) |
| `disableEviction` | Force delete pods instead of evicting them (default `false`) |

### Concurrency

```yaml
spec:
  options:
    concurrency:
      limit: 30
      uploads: 5
```

| Key | Description |
|---|---|
| `limit` | Maximum number of hosts to process in parallel (default `30`) |
| `uploads` | Maximum number of concurrent file uploads (default `5`) |

### Evict taint

```yaml
spec:
  options:
    evictTaint:
      enabled: false
```

| Key | Description |
|---|---|
| `enabled` | Taint nodes before eviction during disruptive operations (default `false`) |

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
