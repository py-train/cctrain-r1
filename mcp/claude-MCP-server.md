## Claude as MCP Server (`claude mcp serve`)

Claude Code itself can act as an MCP server for other applications. This enables external tools, editors, and automation systems to leverage Claude's capabilities through the standard MCP protocol.

```bash
# Start Claude Code as an MCP server on stdio
claude mcp serve
```

Other applications can then connect to this server as they would any stdio-based MCP server. For example, to add Claude Code as an MCP server in another Claude Code instance:

```bash
claude mcp add --transport stdio claude-agent -- claude mcp serve
```

This is useful for building multi-agent workflows where one Claude instance orchestrates another.


- [Managed MCP Configuration (Enterprise)](./enterprise-managed-config.md)
- [Plugin-Provided MCP Servers](./plugin-MCP.md)


