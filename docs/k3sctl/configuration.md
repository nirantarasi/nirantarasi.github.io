---
url: /docs/k3sctl/configuration.md
description: >-
  The k3sctl Cluster file — hosts, roles, SSH access, K3s settings, networking,
  security, TLS, datastore, component args, runtime, embedded registry, image
  credential providers, local storage, etcd snapshots and tuning, registries,
  health checks, token files, kubeconfig path, supervisor metrics, rootless
  mode, CRI endpoints, agent load balancer, node ID, and additional manifests.
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
      withNodeID: true                   # append machine ID to node name
      nodeExternalIP:                    # per-host external IPs
        - 203.0.113.10
      nodeInternalDNS:                   # per-host internal DNS names
        - agent1.internal.example.com
      nodeExternalDNS:                   # per-host external DNS names
        - agent1.example.com

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
        sessionToken: FwoGZXIv...          # STS session token
        bucketLookupType: auto             # auto, dns, or path
        proxy: http://proxy.example.com:8080
        configSecret: my-s3-secret
        timeout: 30s
        retention: 10                      # per-S3 retention override
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
      tokenFile: /etc/rancher/k3s/token     # read token from a file
      agentTokenFile: /etc/rancher/k3s/agent-token
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

    network:                           # cluster networking
      clusterCIDR:
        - 10.42.0.0/16
      serviceCIDR:
        - 10.43.0.0/16
      clusterDNS:
        - 10.43.0.10
      clusterDomain: cluster.local
      nodePortRange: "30000-32767"
      flannelBackend: vxlan             # none, vxlan, host-gw, wireguard-native
      flannelIPv6Masq: false
      flannelExternalIP: false
      flannelIface: eth1                # network interface for flannel
      flannelConf: /etc/flannel.json    # custom flannel config file
      flannelCNIConf: /etc/cni/net.d/10-flannel.conflist
      egressSelectorMode: agent         # agent, cluster, pod, disabled
      serviceLBNamespace: kube-system

    disable:                           # disable packaged components
      components:
        - traefik
        - servicelb
      scheduler: false
      cloudController: false
      kubeProxy: false
      networkPolicy: false
      helmController: false

    tls:                               # API server TLS SANs
      sans:
        - lb.example.com
        - 10.0.0.100
      sanSecurity: true

    kubeconfig:                        # kubeconfig output settings
      mode: "0644"
      group: admin
      name: my-cluster
      path: /output/kubeconfig.yaml    # custom kubeconfig output path

    datastore:                         # external datastore (MySQL/Postgres/etcd)
      endpoint: "postgres://user:pass@db:5432/k3s"
      caFile: /etc/ssl/ca.pem
      certFile: /etc/ssl/cert.pem
      keyFile: /etc/ssl/key.pem

    server:                            # API server listener settings
      advertiseAddress: 10.0.0.50
      advertisePort: 6443
      bindAddress: 0.0.0.0
      httpsPort: 6443
      supervisorMetrics: true          # expose supervisor metrics endpoint
      enablePprof: true                # enable pprof profiling endpoint

    security:                          # security hardening
      selinux: true
      protectKernelDefaults: true
      encryptionProvider: secretbox     # aescbc or secretbox
      rootless: true                   # run K3s in rootless mode
      preferBundledBin: true           # prefer bundled binaries over host ones

    componentArgs:                     # typed pass-through args for K3s components
      kubeAPIServer:
        - "--audit-log-path=/var/log/audit.log"
      kubeControllerManager:
        - "--node-cidr-mask-size=24"
      kubeScheduler: []
      kubeCloudControllerManager: []
      kubelet:
        - "--max-pods=110"
      kubeProxy:
        - "--proxy-mode=ipvs"
      etcd:
        - "--quota-backend-bytes=8589934592"
      helmController: []

    runtime:                           # container runtime settings
      systemDefaultRegistry: registry.example.com
      pauseImage: registry.example.com/pause:3.9
      snapshotter: stargz
      defaultRuntime: nvidia
      resolvConf: /etc/k3s-resolv.conf  # custom resolv.conf for containers
      containerRuntimeEndpoint: unix:///run/containerd/containerd.sock
      imageServiceEndpoint: unix:///run/containerd/containerd.sock

    embeddedRegistry:                  # built-in Spegel distributed registry
      enabled: true
      disableDefaultRegistryEndpoint: false  # disable the default registry endpoint

    imageCredentialProvider:           # kubelet image credential provider
      binDir: /var/lib/rancher/credentialprovider/bin
      config: /var/lib/rancher/credentialprovider/config.yaml

    agent:                             # agent-specific settings
      lbServerPort: 6444               # agent load balancer port (0-65535)

    localStorage:                      # local-path provisioner settings
      path: /opt/local-path-provisioner

    etcdAdvanced:                      # advanced etcd tuning
      exposeMetrics: true
      disableSnapshots: false
      snapshotName: my-cluster
      snapshotReconcileInterval: 5m

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
| `nodeExternalIP` | List of external IPs advertised for the node (passed as `--node-external-ip`) |
| `nodeInternalDNS` | List of internal DNS names for the node |
| `nodeExternalDNS` | List of external DNS names for the node |
| `withNodeID` | Append the machine ID to the node name for uniqueness |
| `noFlannel` | Disable the flannel CNI |
| `os` | Override OS detection |

### SSH access

The `ssh` block takes an `address`, `user`, `port`, and `keyPath`. Use
`openSSH` to reuse your OpenSSH client configuration, or `localhost: true`
when the host is the machine running `k3sctl` (no SSH needed):

```yaml
spec:
  hosts:
  - localhost: true
    role: server
```

Localhost mode is single-node only and cannot be combined with agent hosts.

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
| `network` | Cluster and service CIDRs, DNS, flannel backend, egress mode, NodePort range |
| `disable` | Disable packaged components and individual K3s subsystems |
| `tls` | Subject Alternative Names for the API server certificate |
| `kubeconfig` | Write-kubeconfig mode, group, and name |
| `datastore` | External datastore endpoint (MySQL/Postgres/etcd) with TLS |
| `server` | API server advertise address/port, bind address, HTTPS port |
| `security` | SELinux, protect-kernel-defaults, encryption provider |
| `componentArgs` | Typed pass-through args for kube-apiserver, kubelet, etcd, and others |
| `runtime` | System default registry, pause image, snapshotter, default runtime, resolv.conf |
| `embeddedRegistry` | Enable the built-in Spegel distributed container registry |
| `imageCredentialProvider` | Kubelet image credential provider binary directory and config |
| `localStorage` | Custom path for the local-path provisioner |
| `etcdAdvanced` | Advanced etcd settings: metrics, snapshots, reconcile interval |
| `agent` | Agent-specific settings: load balancer server port |

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
| `s3.sessionToken` | STS session token for temporary AWS credentials |
| `s3.bucketLookupType` | Bucket lookup style: `auto`, `dns`, or `path` (default `auto`) |
| `s3.proxy` | HTTP proxy URL for S3 requests |
| `s3.configSecret` | Name of a Kubernetes secret containing the S3 configuration |
| `s3.timeout` | Timeout for S3 operations, for example `"30s"` |
| `s3.retention` | Per-S3 snapshot retention count (overrides the top-level `retention`) |

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
| `tokenFile` | Path to a file containing the server token (mutually exclusive with `serverToken`) |
| `agentTokenFile` | Path to a file containing the agent token (mutually exclusive with `agentToken`) |
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

### network

Declarative cluster networking configuration. All fields are server-only.

| Key | Description |
|---|---|
| `clusterCIDR` | Pod CIDR ranges, for example `["10.42.0.0/16"]` or dual-stack `["10.42.0.0/16", "fd42::/48"]` |
| `serviceCIDR` | Service CIDR ranges, for example `["10.43.0.0/16"]` |
| `clusterDNS` | Cluster DNS service addresses, for example `["10.43.0.10"]` |
| `clusterDomain` | Cluster domain, for example `cluster.local` |
| `nodePortRange` | NodePort service port range, for example `"30000-32767"` |
| `flannelBackend` | Flannel backend: `none`, `vxlan`, `host-gw`, or `wireguard-native` |
| `flannelIPv6Masq` | Enable IPv6 masquerading for flannel |
| `flannelExternalIP` | Use node external IPs for flannel traffic |
| `flannelIface` | Network interface to use for flannel, for example `"eth1"` |
| `flannelConf` | Path to a custom flannel configuration file |
| `flannelCNIConf` | Path to a custom flannel CNI configuration file |
| `egressSelectorMode` | Egress selector mode: `agent`, `cluster`, `pod`, or `disabled` |
| `serviceLBNamespace` | Namespace for the built-in ServiceLB controller |

### disable

Disable packaged K3s components and subsystems. Server-only.

| Key | Description |
|---|---|
| `components` | List of packaged components to disable, for example `["traefik", "servicelb", "metrics-server", "coredns"]` |
| `scheduler` | Disable the kube-scheduler |
| `cloudController` | Disable the cloud controller manager |
| `kubeProxy` | Disable kube-proxy |
| `networkPolicy` | Disable the network policy controller |
| `helmController` | Disable the Helm controller |

### tls

TLS Subject Alternative Names for the API server certificate. Server-only.

| Key | Description |
|---|---|
| `sans` | Additional SANs for the API server TLS certificate, for example `["lb.example.com", "10.0.0.100"]` |
| `sanSecurity` | Enable TLS SAN security mode |

### kubeconfig

Controls how the admin kubeconfig file is written on server nodes. Server-only.

| Key | Description |
|---|---|
| `mode` | File permission mode for `kubeconfig`, for example `"0644"` |
| `group` | Group ownership for the kubeconfig file |
| `name` | Cluster name written into the kubeconfig |
| `path` | Custom output path for the kubeconfig file |

### datastore

External datastore configuration for MySQL, PostgreSQL, or etcd. Server-only.
TLS fields require `endpoint` to be set.

| Key | Description |
|---|---|
| `endpoint` | Datastore connection string, for example `"postgres://user:pass@db:5432/k3s"` |
| `caFile` | Path to the CA certificate for the datastore TLS connection |
| `certFile` | Path to the client certificate for the datastore TLS connection |
| `keyFile` | Path to the client key for the datastore TLS connection |

### server

API server listener settings. Server-only.

| Key | Description |
|---|---|
| `advertiseAddress` | Address the API server advertises to cluster members |
| `advertisePort` | Port the API server advertises (default `6443`) |
| `bindAddress` | Address the API server binds to, for example `"0.0.0.0"` |
| `httpsPort` | HTTPS listen port for the API server |
| `supervisorMetrics` | Enable the supervisor metrics endpoint |
| `enablePprof` | Enable the pprof profiling endpoint on the supervisor |

### security

Security hardening options. Applied to both server and agent nodes.

| Key | Description |
|---|---|
| `selinux` | Enable SELinux support in K3s |
| `protectKernelDefaults` | Protect kernel defaults; K3s will error if required kernel parameters are not set |
| `encryptionProvider` | Secrets encryption provider: `aescbc` or `secretbox` |
| `rootless` | Run K3s in rootless mode (no root privileges required) |
| `preferBundledBin` | Prefer K3s bundled binaries (containerd, runc, etc.) over host-installed versions |

### componentArgs

Typed pass-through arguments for Kubernetes components. Server-only args are
applied only to server configs; `kubelet` and `kubeProxy` are applied to both
server and agent configs.

| Key | Applies to | Description |
|---|---|---|
| `kubeAPIServer` | Server | Extra arguments for kube-apiserver |
| `kubeControllerManager` | Server | Extra arguments for kube-controller-manager |
| `kubeScheduler` | Server | Extra arguments for kube-scheduler |
| `kubeCloudControllerManager` | Server | Extra arguments for kube-cloud-controller-manager |
| `kubelet` | Server + Agent | Extra arguments for kubelet |
| `kubeProxy` | Server + Agent | Extra arguments for kube-proxy |
| `etcd` | Server | Extra arguments for etcd |
| `helmController` | Server | Extra arguments for the Helm controller |

### runtime

Container runtime settings. Applied to both server and agent nodes.

| Key | Description |
|---|---|
| `systemDefaultRegistry` | Default registry for all system images, for example `"registry.example.com"` |
| `pauseImage` | Override the pause container image |
| `snapshotter` | Containerd snapshotter, for example `"stargz"` |
| `defaultRuntime` | Default container runtime, for example `"nvidia"` |
| `resolvConf` | Path to a custom `resolv.conf` for containers, for example `"/etc/k3s-resolv.conf"` |
| `containerRuntimeEndpoint` | CRI endpoint for the container runtime, for example `"unix:///run/containerd/containerd.sock"` |
| `imageServiceEndpoint` | CRI endpoint for the image service, for example `"unix:///run/containerd/containerd.sock"` |

### embeddedRegistry

Enable the built-in Spegel distributed container registry. Server-only.

| Key | Description |
|---|---|
| `enabled` | Enable the embedded Spegel distributed registry for peer-to-peer image sharing |
| `disableDefaultRegistryEndpoint` | Disable the default network endpoint for the embedded registry |

When enabled, K3s runs a built-in registry that distributes container images
across cluster nodes using peer-to-peer replication instead of pulling from an
external registry for every node.

### imageCredentialProvider

Kubelet image credential provider for authenticating with private container
registries. Applied to both server and agent nodes.

| Key | Description |
|---|---|
| `binDir` | Directory containing credential provider plugin binaries (required) |
| `config` | Path to the credential provider configuration file (required) |

Both `binDir` and `config` must be set together. K3s passes them as
`--image-credential-provider-bin-dir` and `--image-credential-provider-config`.

### localStorage

Local-path provisioner settings. Server-only.

| Key | Description |
|---|---|
| `path` | Storage path used by the built-in local-path provisioner, for example `"/opt/local-path-provisioner"` |

### etcdAdvanced

Advanced etcd tuning options. Server-only.

| Key | Description |
|---|---|
| `exposeMetrics` | Expose etcd metrics to the `/metrics` endpoint |
| `disableSnapshots` | Disable automatic etcd snapshots |
| `snapshotName` | Name prefix for etcd snapshots |
| `snapshotReconcileInterval` | Interval for snapshot reconciliation, for example `"5m"` |

### agent

Agent-specific settings. Applied only to agent nodes.

| Key | Description |
|---|---|
| `lbServerPort` | Port for the agent load balancer that fronts the control plane (0–65535) |

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
