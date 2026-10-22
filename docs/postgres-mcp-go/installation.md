---
url: /docs/postgres-mcp-go/installation.md
description: >-
  Install postgres-mcp-go with Homebrew, a release binary, Docker, or from
  source.
---

# Installation

## Homebrew (macOS / Linux)

```sh
brew install vinaycharlie01/tap/postgres-mcp-go
```

## Download a binary

Pre-built binaries for Linux, macOS, and Windows (`amd64` / `arm64`) are
published with every [GitHub release](https://github.com/vinaycharlie01/postgres-mcp-go/releases).

```sh
# Example: the latest release for macOS arm64
curl -LO https://github.com/vinaycharlie01/postgres-mcp-go/releases/latest/download/postgres-mcp-go_darwin_arm64.tar.gz
tar xzf postgres-mcp-go_darwin_arm64.tar.gz
sudo mv postgres-mcp-go /usr/local/bin/
```

## Docker

```sh
docker pull ghcr.io/vinaycharlie01/postgres-mcp-go:latest

docker run --rm \
  -e POSTGRES_HOST=host.docker.internal \
  -e POSTGRES_PORT=5432 \
  -e POSTGRES_USER=readonly_user \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  ghcr.io/vinaycharlie01/postgres-mcp-go:latest
```

## Build from source

Requires Go 1.27 or newer.

```sh
git clone https://github.com/vinaycharlie01/postgres-mcp-go.git
cd postgres-mcp-go
go build -o dist/postgres-mcp-go ./cmd/server
```

The binary is written to `dist/postgres-mcp-go`.

## Configure and run

The server reads `config.yaml` or environment variables. To start from the
example file:

```sh
cp config.yaml.example config.yaml
# edit config.yaml with your database credentials
postgres-mcp-go
```

The default transport is stdio, so the process waits for an MCP client on
standard input. See [Configuration](./configuration.md) for every option and
the Claude Desktop and Cline snippets.
