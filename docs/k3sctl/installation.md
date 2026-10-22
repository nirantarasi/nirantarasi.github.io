---
url: /docs/k3sctl/installation.md
description: Install k3sctl from a release binary, with Go, or build it from source.
---

# Installation

`k3sctl` ships as a single static binary. Pick the method that fits your
workflow.

## Release binary

Download the binary for your platform and put it on your `PATH`:

```sh
curl -sSfL -o k3sctl \
  https://github.com/vinaycharlie01/k3sctl/releases/latest/download/k3sctl_$(uname -s | tr '[:upper:]' '[:lower:]')_$(uname -m)
chmod +x k3sctl
sudo install -m 755 k3sctl /usr/local/bin/k3sctl
```

Release assets are published for Linux and macOS on `amd64`, `arm64`, and
`arm`.

## With Go

If you already have a Go toolchain:

```sh
go install github.com/vinaycharlie01/k3sctl@latest
```

This installs `k3sctl` into `$(go env GOPATH)/bin`.

## Build from source

```sh
git clone https://github.com/vinaycharlie01/k3sctl.git
cd k3sctl
go build -o k3sctl .
./k3sctl version
```

## Verify the installation

```sh
k3sctl version
k3sctl --help
```

## Shell completion

`k3sctl` can print a completion script for your shell:

```sh
# Bash
source <(k3sctl completion bash)

# Zsh
k3sctl completion zsh > "${fpath[1]}/_k3sctl"
```

Run `k3sctl completion --help` for the shells your build supports.

## Next steps

Generate a starter configuration, edit it, and apply it:

```sh
k3sctl init --name demo --address 10.0.0.1 --agent --agent-address 10.0.0.2 > k3sctl.yaml
k3sctl apply --config k3sctl.yaml
```

See [Configuration](./configuration.md) for the full file reference and
[Commands](./commands.md) for every subcommand.
