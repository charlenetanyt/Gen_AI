# mcp_server/  (Owner: TBC)

MCP server exposing `list_tables`, `describe_table` and `run_read_query`. `run_read_query` calls `guardrails.check_sql` first when guardrails are enabled.
