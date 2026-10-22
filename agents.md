---
url: /agents.md
description: >-
  A compact map of Nirantara's documentation, plus the details that are easiest
  to get wrong when generating configuration for k3sctl, the MCP servers, or the
  omniroute-operator.
---

# Documentation for coding agents

Nirantara maintains four open-source projects. The documentation below is
organised per project, and every page is also available as raw Markdown by
appending `.md` to its URL. The curated index is at [/llms.txt](/llms.txt) and
the full corpus at [/llms-full.txt](/llms-full.txt).

## Where to look

### k3sctl

Declarative K3s cluster lifecycle over SSH.

* [Overview](./docs/k3sctl/index.md)
* [Installation](./docs/k3sctl/installation.md)
* [Configuration](./docs/k3sctl/configuration.md): the `Cluster` file, hosts,
  K3s settings, and drain options.
* [Commands](./docs/k3sctl/commands.md): `apply`, `reset`, `kubeconfig`,
  `backup`, and the flags each accepts.

### mssql-mcp-go

Read-only MCP server for Microsoft SQL Server.

* [Overview](./docs/mssql-mcp-go/index.md)
* [Installation](./docs/mssql-mcp-go/installation.md)
* [Configuration](./docs/mssql-mcp-go/configuration.md): `config.yaml`,
  `SQLSERVER_*` variables, and client snippets.
* [Tools](./docs/mssql-mcp-go/tools.md)

### postgres-mcp-go

Read-only MCP server for PostgreSQL.

* [Overview](./docs/postgres-mcp-go/index.md)
* [Installation](./docs/postgres-mcp-go/installation.md)
* [Configuration](./docs/postgres-mcp-go/configuration.md): `config.yaml`,
  `POSTGRES_*` variables, and client snippets.
* [Tools](./docs/postgres-mcp-go/tools.md)

### omniroute-operator

Kubernetes operator for OmniRoute.

* [Overview](./docs/omniroute-operator/index.md)
* [Installation](./docs/omniroute-operator/installation.md)
* [OmniRoute CR](./docs/omniroute-operator/omniroute-cr.md): every field in
  `gateway.omniroute.ai/v1alpha1`.
* [Local development](./docs/omniroute-operator/local-development.md)

## Details that are easy to get wrong

### k3sctl

1. Hosts must be Linux. `k3sctl` installs K3s and systemd units over SSH.
2. The configuration is `apiVersion: k3sctl.k3sproject.io/v1beta1`,
   `kind: Cluster`.
3. `spec.hosts[]` entries choose a `role` of `server` or `agent`. Servers are
   the default.
4. Extra YAML documents in the same file are applied after installation with
   `kubectl apply`. Separate them with `---`, and make each one a valid
   Kubernetes manifest.
5. `apply` is idempotent and only changes what drifted. `--dry-run` shows the
   plan without changing the cluster.
6. `reset` removes K3s from the hosts. It is not the same as deleting a
   namespace.

### MCP servers

1. Both servers are read-only. Mutating SQL is rejected before execution, and
   stored procedures are analysed, never run.
2. Environment variables override `config.yaml`; never assume the file wins.
3. The default transport is `stdio`, so an MCP client launches the binary rather
   than connecting to a port.
4. `mssql-mcp-go` uses `SQLSERVER_*` variables and `encrypt`/`ssl` options that
   are SQL Server specific. `postgres-mcp-go` uses `POSTGRES_*` variables and
   `ssl_mode` plus `search_path`.
5. The tool names and groups are the same across both servers, but the SQL
   dialect differs. Do not copy a T-SQL snippet into a PostgreSQL answer.

### omniroute-operator

1. The group and version are `gateway.omniroute.ai/v1alpha1`; the kind is
   `OmniRoute`.
2. `spec.replicas` is effectively capped at `1` because OmniRoute stores state
   in SQLite with a single writer.
3. Credentials come from an existing Secret with `JWT_SECRET`,
   `API_KEY_SECRET`, and `INITIAL_PASSWORD`.
4. `spec.ingress.enabled` defaults to `false`. Enable it explicitly and set
   `className`, `host`, and TLS if the instance must be public.
5. `spec.persistence.enabled` creates a PVC. Disabling it makes the instance
   ephemeral.
6. `NODE_OPTIONS` is derived from the memory limit; do not hardcode it in
   `spec.env` unless you need to override the derived value.
