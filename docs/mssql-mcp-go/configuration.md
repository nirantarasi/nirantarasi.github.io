---
url: /docs/mssql-mcp-go/configuration.md
description: >-
  Configure mssql-mcp-go with config.yaml or environment variables, and connect
  it to Claude Desktop, Cline, or Docker.
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
  port: 1433
  user: "readonly_user"
  password: ""              # prefer SQLSERVER_PASSWORD
  database: "MyDatabase"
  encrypt: "true"           # true, false, or disable
  trust_server_certificate: false
  connection_timeout: 30

# Always Encrypted: client-side column decryption with a local PFX.
encryption:
  enabled: false
  key_path: ""              # absolute path to the .pfx file
  key_password: ""          # prefer SQLSERVER_ALWAYS_ENCRYPTED_KEY_PASSWORD

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

mcp:
  transport: "stdio"        # stdio or http
  port: 9090
```

## Environment variables

| Variable | Description |
|---|---|
| `SQLSERVER_HOST` | Database host |
| `SQLSERVER_PORT` | Database port (default `1433`) |
| `SQLSERVER_USER` | Database user |
| `SQLSERVER_PASSWORD` | Database password |
| `SQLSERVER_DATABASE` | Database name |
| `SQLSERVER_ENCRYPT` | Encryption mode (default `true`) |
| `SQLSERVER_TRUST_SERVER_CERTIFICATE` | Trust self-signed certificates |
| `SQLSERVER_CONNECTION_TIMEOUT` | Connection timeout in seconds |
| `SQLSERVER_ALWAYS_ENCRYPTED_KEY_PATH` | Path to the PFX certificate |
| `SQLSERVER_ALWAYS_ENCRYPTED_KEY_PASSWORD` | PFX certificate password |
| `QUERY_MAX_LENGTH` | Maximum query length |
| `QUERY_MAX_ROWS` | Maximum result rows |
| `QUERY_TIMEOUT_SECONDS` | Per-query timeout |
| `LOG_LEVEL` | Log level |
| `LOG_FORMAT` | Log format |
| `CONFIG_PATH` | Path to `config.yaml` |

## MCP client integration

### Claude Desktop

Add the server to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mssql": {
      "command": "mssql-mcp-go",
      "env": {
        "SQLSERVER_HOST": "localhost",
        "SQLSERVER_PORT": "1433",
        "SQLSERVER_USER": "readonly_user",
        "SQLSERVER_PASSWORD": "secret",
        "SQLSERVER_DATABASE": "MyDatabase",
        "SQLSERVER_ENCRYPT": "disable"
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
    "mssql": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "SQLSERVER_HOST=host.docker.internal",
        "-e", "SQLSERVER_PORT=1433",
        "-e", "SQLSERVER_USER=readonly_user",
        "-e", "SQLSERVER_PASSWORD=secret",
        "-e", "SQLSERVER_DATABASE=MyDatabase",
        "ghcr.io/vinaycharlie01/mssql-mcp-go:latest"
      ]
    }
  }
}
```

### Cline (VS Code)

Add the server to your Cline MCP settings:

```json
{
  "mssql": {
    "command": "mssql-mcp-go",
    "env": {
      "SQLSERVER_HOST": "localhost",
      "SQLSERVER_PORT": "1433",
      "SQLSERVER_USER": "readonly_user",
      "SQLSERVER_PASSWORD": "secret",
      "SQLSERVER_DATABASE": "MyDatabase"
    }
  }
}
```

## Security notes

* Use a login that only has read permissions on the databases the assistant
  should see.
* Prefer environment variables over committing credentials to `config.yaml`.
* TLS is on by default (`encrypt: true`). Set `trust_server_certificate: false`
  unless you are connecting to a server with a self-signed certificate.
* Always Encrypted columns are decrypted client-side with the PFX you provide;
  the key never leaves the machine running the server.
