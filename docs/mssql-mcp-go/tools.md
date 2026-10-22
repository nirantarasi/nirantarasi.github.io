---
url: /docs/mssql-mcp-go/tools.md
description: All tools exposed by mssql-mcp-go, grouped by what they help you do.
---

# Tools

Every tool below issues read-only catalog queries or validated `SELECT` /
`WITH` statements. Mutating SQL is rejected before execution.

## Core query

| Tool | Description |
|---|---|
| `execute_query` | Run a read-only `SELECT` / `WITH` query |
| `health_check` | Check connectivity and connection-pool stats |

## Database discovery

| Tool | Description |
|---|---|
| `list_databases` | List accessible databases |
| `list_schemas` | List schemas in the current database |
| `list_tables` | List tables with row and column counts |
| `describe_table` | Columns, keys, and indexes of a table |
| `list_columns` | Columns with types and nullability |
| `describe_column` | Detailed metadata for a single column |
| `table_row_count` | Row count from catalog metadata |

## Relationships and keys

| Tool | Description |
|---|---|
| `table_relationships` | Outbound and inbound foreign keys |
| `list_foreign_keys` | Foreign-key constraints, optionally by schema |
| `list_primary_keys` | Primary-key constraints, optionally by schema |
| `foreign_key_graph` | Full foreign-key graph as nodes and edges |
| `list_unique_constraints` | Unique constraints by schema |
| `list_check_constraints` | Check constraints by schema |
| `list_default_constraints` | Default constraints by schema |

## Stored procedures

| Tool | Description |
|---|---|
| `list_procedures` | List stored procedures |
| `get_procedure_definition` | Full T-SQL definition and parameters |
| `search_procedures` | Search definitions by keyword |
| `analyze_stored_procedure` | Static analysis: tables, views, transactions, and more |
| `extract_tables_used` | Tables referenced by a procedure |
| `extract_views_used` | Views referenced by a procedure |
| `extract_functions_used` | Functions referenced by a procedure |
| `extract_called_procedures` | Procedures called with `EXEC` |
| `extract_transaction_usage` | Transaction-control usage |
| `extract_dynamic_sql` | Dynamic SQL detection |
| `extract_cursor_usage` | Cursor declarations |
| `extract_temp_tables` | Temp-table usage |
| `extract_error_handling` | Error-handling constructs |
| `extract_validation_rules` | Validation rules |
| `extract_business_rules` | Business-rule comments |

## Views

| Tool | Description |
|---|---|
| `list_views` | List all views |
| `get_view_definition` | Definition of a view |
| `analyze_view` | Columns and referenced objects |
| `trace_view_sources` | Resolve underlying base tables |
| `extract_view_dependencies` | Objects a view depends on |

## Functions

| Tool | Description |
|---|---|
| `list_functions` | Scalar, inline, and table-valued functions |
| `get_function_definition` | Definition and parameters |
| `analyze_function` | Static analysis |
| `trace_function_usage` | Objects referencing the function |
| `extract_function_dependencies` | Objects a function depends on |

## Triggers

| Tool | Description |
|---|---|
| `list_triggers` | DML triggers with their tables |
| `get_trigger_definition` | Definition of a trigger |
| `analyze_trigger` | Static analysis |

## Dependencies and search

| Tool | Description |
|---|---|
| `find_object_dependencies` | Objects an object depends on |
| `find_object_references` | Objects that reference an object |
| `find_table_usage` | All objects referencing a table |
| `search_objects_by_keyword` | Search names and definitions |
| `generate_erd` | Entity-relationship description |
| `dependency_graph` | Recursive dependency graph |
| `find_column_usage` | Modules referencing a column |
| `find_unused_tables` | Tables with no references |
| `find_unused_columns` | Columns with no tracked references |
| `find_orphan_tables` | Tables with no foreign-key connections |
| `find_circular_dependencies` | Cycles in the foreign-key graph |
| `search_tables` | Tables matching a keyword |
| `search_columns` | Columns matching a keyword |
| `search_views` | Views matching a keyword |
| `search_functions` | Functions matching a keyword |

## Data profiling

| Tool | Description |
|---|---|
| `table_statistics` | Row count, column count, and storage stats |
| `column_statistics` | Min, max, distinct, and null statistics |
| `column_distinct_values` | Top distinct values with frequencies |
| `column_null_analysis` | Null count and percentage |
| `data_distribution` | Value-frequency distribution |
| `sample_table_data` | Sample rows from a table |
| `find_duplicate_rows` | Duplicate groups by columns |
| `find_candidate_keys` | Single-column candidate-key detection |

## Performance analysis

| Tool | Description |
|---|---|
| `explain_query` | Estimated execution plan |
| `estimate_query_cost` | Estimated subtree cost |
| `analyze_execution_plan` | Plan with an operator summary |
| `find_missing_indexes` | Optimizer-recommended indexes |
| `find_unused_indexes` | Indexes that are never read |
| `find_fragmented_indexes` | High-fragmentation indexes |
| `index_usage_report` | Read and write stats for all indexes |
| `slow_query_analysis` | Most expensive cached queries |
| `query_performance_report` | Aggregate query stats |

## Security analysis

| Tool | Description |
|---|---|
| `list_logins` | Server-level logins |
| `list_users` | Database-level users |
| `list_roles` | Database roles |
| `list_permissions` | Database permission grants |
| `user_permissions` | Permissions for a specific user |
| `role_memberships` | Role-membership graph |
| `object_permissions` | Permissions on a specific object |

## Migration assessment

| Tool | Description |
|---|---|
| `database_overview` | High-level object counts |
| `database_health_report` | Health indicators |
| `database_complexity_report` | Structural complexity metrics |
| `analyze_database_for_migration` | Comprehensive migration assessment |
| `migration_complexity_score` | Weighted complexity score |
| `migration_priority_order` | Dependency-based migration order |
| `high_risk_objects` | Modules with risky constructs |
| `large_table_analysis` | Largest tables by storage |
| `legacy_object_analysis` | Stale or legacy objects |
| `deprecated_feature_detection` | Deprecated-feature scan |

## Monitoring

| Tool | Description |
|---|---|
| `connection_pool_stats` | Connection-pool statistics |
| `database_size_report` | Data and log file sizes |
| `table_growth_report` | Largest tables by size |
