---
url: /docs/postgres-mcp-go/configuration.md
description: >-
  Configure postgres-mcp-go with config.yaml or environment variables, and
  connect it to Claude Desktop, Cline, or Docker.
---

# Configuration

The server reads settings from a `config.yaml` file or from environment
variables. **Environment variables always override the file**, which makes
secrets easy to inject at runtime.

## config.yaml

Start from `config.yaml.example`:

```yaml
database:
  host: "localhost"
  port: 5432
  user: "readonly_user"
  password: ""              # prefer POSTGRES_PASSWORD
  database: "postgres"
  ssl_mode: "require"       # disable, allow, prefer, require, verify-ca, verify-full
  search_path: "public"
  connection_timeout: 30

query:
  max_length: 5000          # max characters per query
  max_rows: 1000            # max rows returned
  timeout: 30s              # per-query timeout

pool:
  max_open_conns: 10
  max_idle_conns: 5
  conn_max_lifetime: 1h

logging:
  level: "info"             # debug, info, warn, error
  format: "json"            # json, text
  file: ""                  # optional log file path

mcp:
  transport: "stdio"        # stdio or http
  port: 9090
```

## Environment variables

| Variable | Description |
|---|---|
| `POSTGRES_HOST` | Database host |
| `POSTGRES_PORT` | Database port (default `5432`) |
| `POSTGRES_USER` | Database user |
| `POSTGRES_PASSWORD` | Database password |
| `POSTGRES_DB` | Database name |
| `POSTGRES_SSL_MODE` | TLS mode (default `require`) |
| `POSTGRES_SEARCH_PATH` | Default schema (default `public`) |
| `POSTGRES_CONNECTION_TIMEOUT` | Connection timeout in seconds |
| `QUERY_MAX_LENGTH` | Maximum query length |
| `QUERY_MAX_ROWS` | Maximum result rows |
| `QUERY_TIMEOUT_SECONDS` | Per-query timeout |
| `LOG_LEVEL` | Log level |
| `LOG_FORMAT` | Log format |
| `LOG_FILE` | Log file path |
| `CONFIG_PATH` | Path to `config.yaml` |

## MCP client integration

### Claude Desktop

Add the server to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "postgres-mcp-go",
      "env": {
        "POSTGRES_HOST": "localhost",
        "POSTGRES_PORT": "5432",
        "POSTGRES_USER": "readonly_user",
        "POSTGRES_PASSWORD": "secret",
        "POSTGRES_DB": "mydb",
        "POSTGRES_SSL_MODE": "disable"
      }
    }
  }
}
```

If you installed with Homebrew, the binary is already on your `PATH`.
Otherwise, use its full path.

### Claude Desktop with Docker

```json
{
  "mcpServers": {
    "postgres": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "POSTGRES_HOST=host.docker.internal",
        "-e", "POSTGRES_PORT=5432",
        "-e", "POSTGRES_USER=readonly_user",
        "-e", "POSTGRES_PASSWORD=secret",
        "-e", "POSTGRES_DB=mydb",
        "ghcr.io/vinaycharlie01/postgres-mcp-go:latest"
      ]
    }
  }
}
```

### Cline (VS Code)

Add the server to your Cline MCP settings:

```json
{
  "postgres": {
    "command": "postgres-mcp-go",
    "env": {
      "POSTGRES_HOST": "localhost",
      "POSTGRES_PORT": "5432",
      "POSTGRES_USER": "readonly_user",
      "POSTGRES_PASSWORD": "secret",
      "POSTGRES_DB": "mydb"
    }
  }
}
```

## Security notes

* Use a role that only has read permissions on the databases the assistant
  should see.
* Prefer environment variables over committing credentials to `config.yaml`.
* TLS is on by default (`ssl_mode: require`). Use `verify-full` in production,
  and reserve `disable` for trusted local networks.
* Set `search_path` when the objects live outside `public`, so unqualified names
  resolve as expected.
