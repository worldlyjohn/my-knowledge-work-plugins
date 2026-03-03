# MCP Re-enable Template

When your company re-enables MCP connectors, copy this template into `data/.mcp.json` and replace each URL with your approved endpoint.

> Keep only the servers your org has enabled. Remove anything you cannot authenticate to yet.

```json
{
  "mcpServers": {
    "snowflake": {
      "type": "http",
      "url": "https://YOUR-SNOWFLAKE-MCP-ENDPOINT"
    },
    "databricks": {
      "type": "http",
      "url": "https://YOUR-DATABRICKS-MCP-ENDPOINT"
    },
    "bigquery": {
      "type": "http",
      "url": "https://YOUR-BIGQUERY-MCP-ENDPOINT"
    },
    "hex": {
      "type": "http",
      "url": "https://YOUR-HEX-MCP-ENDPOINT"
    },
    "amplitude": {
      "type": "http",
      "url": "https://YOUR-AMPLITUDE-MCP-ENDPOINT"
    },
    "atlassian": {
      "type": "http",
      "url": "https://YOUR-ATLASSIAN-MCP-ENDPOINT"
    }
  }
}
```

After updating:
1. Restart Claude/Claude Code.
2. Open `/mcp` to verify server status.
3. Authenticate any server that reports `needs authentication`.
