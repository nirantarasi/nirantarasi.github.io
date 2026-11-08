---
url: /docs/k3sctl/installation.md
description: Install k3sctl from a release binary, with Go, or build it from source.
---

# Installation

`k3sctl` ships as a single static binary. Pick the method that fits your
workflow.

## Homebrew (macOS / Linux)

```sh
brew install nirantarasi/tap/k3sctl
```

## Shell script (Linux / macOS)

```sh
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)
case "$ARCH" in
  x86_64)  ARCH="amd64" ;;
  aarch64) ARCH="arm64" ;;
  armv7l)  ARCH="arm" ;;
esac
VERSION=$(curl -sI https://github.com/nirantarasi/k3sctl/releases/latest | grep -i location | sed 's/.*tag\///' | tr -d '\r\n')
TMPDIR=$(mktemp -d)
curl -sSfL "https://github.com/nirantarasi/k3sctl/releases/download/${VERSION}/k3sctl_${VERSION#v}_${OS}_${ARCH}.tar.gz" | tar xz -C "$TMPDIR"
sudo install -m 755 "$TMPDIR/k3sctl" /usr/local/bin/k3sctl
rm -rf "$TMPDIR"
```

Release assets are published for Linux, macOS, and Windows on `amd64`,
`arm64`, and `arm`.

## Debian / Ubuntu

```sh
VERSION=$(curl -sI https://github.com/nirantarasi/k3sctl/releases/latest | grep -i location | sed 's/.*tag\///' | tr -d '\r\n')
curl -sSfL -O "https://github.com/nirantarasi/k3sctl/releases/download/${VERSION}/k3sctl_${VERSION#v}_linux_amd64.deb"
sudo dpkg -i k3sctl_*.deb
rm -f k3sctl_*.deb
```

## RHEL / Fedora

```sh
VERSION=$(curl -sI https://github.com/nirantarasi/k3sctl/releases/latest | grep -i location | sed 's/.*tag\///' | tr -d '\r\n')
curl -sSfL -O "https://github.com/nirantarasi/k3sctl/releases/download/${VERSION}/k3sctl_${VERSION#v}_linux_amd64.rpm"
sudo rpm -i k3sctl_*.rpm
rm -f k3sctl_*.rpm
```

## Alpine

```sh
VERSION=$(curl -sI https://github.com/nirantarasi/k3sctl/releases/latest | grep -i location | sed 's/.*tag\///' | tr -d '\r\n')
curl -sSfL -O "https://github.com/nirantarasi/k3sctl/releases/download/${VERSION}/k3sctl_${VERSION#v}_linux_amd64.apk"
sudo apk add --allow-untrusted k3sctl_*.apk
rm -f k3sctl_*.apk
```

## With Go

If you already have a Go toolchain:

```sh
go install github.com/nirantarasi/k3sctl@latest
```

This installs `k3sctl` into `$(go env GOPATH)/bin`.

## Build from source

```sh
git clone https://github.com/nirantarasi/k3sctl.git
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
