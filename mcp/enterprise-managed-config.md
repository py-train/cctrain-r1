## Managed MCP Configuration (Enterprise)

For enterprise deployments, IT administrators can enforce MCP server policies through the `managed-mcp.json` configuration file. This file provides exclusive control over which MCP servers are permitted or blocked organization-wide.

**Location:**
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `~/.config/ClaudeCode/managed-mcp.json`
- Windows: `%APPDATA%\ClaudeCode\managed-mcp.json`

**Features:**
- `allowedMcpServers` -- whitelist of permitted servers
- `deniedMcpServers` -- blocklist of prohibited servers
- `allowAllClaudeAiMcps` -- managed setting that permits loading claude.ai cloud MCP connectors organization-wide (v2.1.149+)
- Supports matching by server name, command, and URL patterns
- Organization-wide MCP policies enforced before user configuration
- Prevents unauthorized server connections

**Example configuration:**

```json
{
  "allowedMcpServers": [
    {
      "serverName": "github",
      "serverUrl": "https://api.github.com/mcp"
    },
    {
      "serverName": "company-internal",
      "serverCommand": "company-mcp-server"
    }
  ],
  "deniedMcpServers": [
    {
      "serverName": "untrusted-*"
    },
    {
      "serverUrl": "http://*"
    }
  ]
}
```

> **Note:** When both `allowedMcpServers` and `deniedMcpServers` match a server, the deny rule takes precedence.
