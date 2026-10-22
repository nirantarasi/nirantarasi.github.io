---
url: /docs/mssql-mcp-go/installation.md
description: Install mssql-mcp-go with Homebrew, a release binary, Docker, or from source.
---

# Installation

## Homebrew (macOS / Linux)

```sh
brew install vinaycharlie01/tap/mssql-mcp-go
```

## Download a binary

Pre-built binaries for Linux, macOS, and Windows (`amd64` / `arm64`) are
published with every [GitHub release](https://github.com/vinaycharlie01/mssql-mcp-go/releases).

```sh
# Example: the latest release for macOS arm64
curl -LO https://github.com/vinaycharlie01/mssql-mcp-go/releases/latest/download/mssql-mcp-go_darwin_arm64.tar.gz
tar xzf mssql-mcp-go_darwin_arm64.tar.gz
sudo mv mssql-mcp-go /usr/local/bin/
```

## Docker

```sh
docker pull ghcr.io/vinaycharlie01/mssql-mcp-go:latest

docker run --rm \
  -e SQLSERVER_HOST=host.docker.internal \
  -e SQLSERVER_PORT=1433 \
  -e SQLSERVER_USER=readonly_user \
  -e SQLSERVER_PASSWORD=secret \
  -e SQLSERVER_DATABASE=MyDatabase \
  ghcr.io/vinaycharlie01/mssql-mcp-go:latest
```

## Build from source

Requires Go 1.25 or newer.

```sh
git clone https://github.com/vinaycharlie01/mssql-mcp-go.git
cd mssql-mcp-go
go build -o dist/mssql-mcp-go ./cmd/server
```

The binary is written to `dist/mssql-mcp-go`.

## Configure and run

The server reads `config.yaml` or environment variables. To start from the
example file:

```sh
cp config.yaml.example config.yaml
# edit config.yaml with your database credentials
mssql-mcp-go
```

The default transport is stdio, so the process waits for an MCP client on
standard input. See [Configuration](./configuration.md) for every option and
the Claude Desktop and Cline snippets.
