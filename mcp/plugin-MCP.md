## Plugin-Provided MCP Servers

Plugins can bundle their own MCP servers, making them available automatically when the plugin is installed. Plugin-provided MCP servers can be defined in two ways:

1. **Standalone `.mcp.json`** -- Place a `.mcp.json` file in the plugin root directory
2. **Inline in `plugin.json`** -- Define MCP servers directly within the plugin manifest

Use the `${CLAUDE_PLUGIN_ROOT}` variable to reference paths relative to the plugin's installation directory:

```json
{
  "mcpServers": {
    "plugin-tools": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/dist/mcp-server.js"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```
