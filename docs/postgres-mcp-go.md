---
url: /docs/postgres-mcp-go.md
description: >-
  A read-only Model Context Protocol server for PostgreSQL, written in Go, with
  70+ tools for schema, data, performance, security, and migration analysis.
---

# postgres-mcp-go

`postgres-mcp-go` is a **read-only** [Model Context Protocol](https://modelcontextprotocol.io)
(MCP) server for PostgreSQL, written in Go. It gives AI assistants safe,
structured access to your databases for schema exploration, data profiling,
query analysis, migration planning, and more.

## Why read-only

Every query is validated before execution. `INSERT`, `UPDATE`, `DELETE`,
`DROP`, `ALTER`, `CREATE`, `TRUNCATE`, and other mutating statements are
rejected before they reach the database. Stored procedures and functions are
analyzed statically and never executed.

That makes the server safe to hand to an assistant that needs to understand a
production database without being able to change it.

## Features

* **Read-only by design** — mutating SQL is blocked by a validator.
* **70+ tools** covering schema discovery, data profiling, performance
  analysis, security auditing, migration assessment, and monitoring.
* **Connection pooling** with configurable limits and automatic cleanup.
* **Stdio transport** for Claude Desktop, Cline, and other MCP clients.
* **Single binary** for Linux, macOS, and Windows, plus a multi-arch container
  image.

## Quick start

Set the connection with environment variables (they always override the config
file), then run the server:

```sh
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5432
export POSTGRES_USER=readonly_user
export POSTGRES_PASSWORD=secret
export POSTGRES_DB=mydb

postgres-mcp-go
```

The server starts on stdio by default and waits for an MCP client to connect.
See [Configuration](./configuration.md) for `config.yaml` and client setup, and
[Tools](./tools.md) for the full tool list.

## Architecture

```
postgres-mcp-go/
  cmd/server/            Entry point
  internal/
    adapters/mcp/        MCP protocol, tool registration
    postgres/            Service layer (queries, analysis)
    db/                  Connection pool, query execution
    infrastructure/      Config, logging
  pkg/version/           Build version info
```

The server registers all tools with the MCP SDK and forwards tool calls to the
PostgreSQL service layer. Every statement passes through the read-only validator
first.

## Next steps

* [Installation](./installation.md)
* [Configuration](./configuration.md)
* [Tools](./tools.md)
