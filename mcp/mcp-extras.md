## Subagent-Scoped MCP

MCP servers can be defined inline within agent frontmatter using the `mcpServers:` key, scoping them to a specific subagent rather than the entire project. This is useful when an agent needs access to a particular MCP server that other agents in the workflow do not require.

```yaml
---
mcpServers:
  my-tool:
    type: http
    url: https://my-tool.example.com/mcp
---

You are an agent with access to my-tool for specialized operations.
```

Subagent-scoped MCP servers are only available within that agent's execution context and are not shared with the parent or sibling agents.

## MCP Output Limits

Claude Code enforces limits on MCP tool output to prevent context overflow:

| Limit | Threshold | Behavior |
|-------|-----------|----------|
| **Warning** | 10,000 tokens | A warning is displayed that the output is large |
| **Default max** | 25,000 tokens | Output is truncated beyond this limit |
| **Disk persistence** | 50,000 characters | Tool results exceeding 50K characters are persisted to disk |

The maximum output limit is configurable via the `MAX_MCP_OUTPUT_TOKENS` environment variable:

```bash
# Increase the max output to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
```

## Auto-Backgrounding Long-Running Tool Calls (v2.1.212)

MCP tool calls that run longer than 2 minutes now move to the background automatically, so the session stays usable instead of blocking on a slow tool. The threshold is configurable, and the behavior can be tuned or disabled, via `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS`:

```bash
# Change the auto-background threshold to 5 minutes (300,000ms)
export CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS=300000
```

## Environment Variables

Store sensitive credentials in environment variables:

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxx"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export SLACK_TOKEN="xoxb-xxxxxxxxxxxxx"
```

Then reference them in MCP config:

```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```


## Server Deduplication

When the same MCP server is defined at multiple scopes (local, project, user), the local configuration takes precedence. This allows you to override project-level or user-level MCP settings with local customizations without conflicts.

## Recent Lifecycle Fixes (v2.1.136)

Two long-standing MCP lifecycle bugs were fixed in v2.1.136 — worth upgrading for if you run multi-server setups:

- **MCP servers persist across `/clear`**: Servers configured via `.mcp.json`, plugins, or claude.ai connectors no longer disappear after `/clear` in VS Code, JetBrains, or the Agent SDK. Earlier versions silently dropped them and required a restart.
- **OAuth refresh-token concurrent-refresh fix**: Multi-server OAuth setups no longer lose refresh tokens when several servers race to refresh simultaneously. This eliminates the "every morning I have to re-auth" pattern that affected setups with multiple OAuth-protected MCP servers.

## Dynamic Tool Updates

Claude Code supports MCP `list_changed` notifications. When an MCP server dynamically adds, removes, or modifies its available tools, Claude Code receives the update and adjusts its tool list automatically -- no reconnection or restart required.

## MCP Apps

MCP Apps is the first official MCP extension, enabling MCP tool calls to return interactive UI components that render directly in the chat interface. Instead of plain text responses, MCP servers can deliver rich dashboards, forms, data visualizations, and multi-step workflows -- all displayed inline without leaving the conversation.

## MCP Elicitation

MCP servers can request structured input from the user via interactive dialogs (v2.1.49+). This allows an MCP server to ask for additional information mid-workflow -- for example, prompting for a confirmation, selecting from a list of options, or filling in required fields -- adding interactivity to MCP server interactions.

## Tool Description and Instruction Cap

As of v2.1.84, Claude Code enforces a **2 KB cap** on tool descriptions and instructions per MCP server. This prevents individual servers from consuming excessive context with overly verbose tool definitions, reducing context bloat and keeping interactions efficient.

