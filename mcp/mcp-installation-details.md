## Other MCP Installation Details

#### `CLAUDE_PROJECT_DIR` for stdio servers (v2.1.139+)

Every MCP stdio server is spawned with `CLAUDE_PROJECT_DIR=<absolute path to repo root>` already set in its environment — the same convention used for hooks. Plugin and project `.mcp.json` files can reference `${CLAUDE_PROJECT_DIR}` in the `command`, `args`, and `env` values, and the substitution happens before `execve()`:

```json
{
  "mcpServers": {
    "repo-tools": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PROJECT_DIR}/.claude/mcp/repo-tools.js"],
      "env": {
        "REPO_ROOT": "${CLAUDE_PROJECT_DIR}"
      }
    }
  }
}
```

Use this when your stdio server needs to read files relative to the project root regardless of where Claude Code was launched.

stdio MCP servers also receive `CLAUDE_CODE_SESSION_ID` (matching the value passed to hooks and Bash), including when the session is resumed with `--resume` (v2.1.163+).

### SSE Transport (Deprecated)

Server-Sent Events transport is deprecated in favor of `http` but still supported:

```bash
claude mcp add --transport sse legacy-server https://example.com/sse
```

### WebSocket Transport (`ws`)

WebSocket servers hold a persistent bidirectional connection, which suits remote MCP servers that push events to Claude unprompted. Use HTTP instead when your server only responds to requests, since HTTP supports OAuth and the `claude mcp add --transport` flag, while WebSocket supports neither.

Because `--transport` does not accept `ws`, configure it in `.mcp.json` or through `claude mcp add-json`:

```json
{
  "type": "ws",
  "url": "wss://mcp.example.com/socket",
  "headers": {
    "Authorization": "Bearer YOUR_TOKEN"
  }
}
```

The `type: "ws"` entry accepts the same `url`, `headers`, `headersHelper`, `timeout`, and `alwaysLoad` fields as `http`. Authentication is **header-only** — there is no OAuth flow for WebSocket servers.

> **Note**: WebSocket servers don't appear in `claude mcp list` output. Use `claude mcp get <name>` or the `/mcp` panel to check them.

Like HTTP and SSE, WebSocket connections use a 5-minute idle window; stdio and WebSocket have no per-request timer. A `url` entry with no `type` is an error naming `"http"`, `"sse"`, and `"ws"` as the valid values.

### Session Working Directories (roots/list)

MCP servers can discover the session's working directories: the launch directory plus all `--add-dir`/`additionalDirectories` entries are returned via the MCP `roots/list` request, and a `notifications/roots/list_changed` notification is sent whenever the set changes (v2.1.203). The idle timeout now also applies to stdio servers (30 minutes), with a per-server `timeout` acting as an idle floor (v2.1.203).

### Windows-Specific Note

On native Windows (not WSL), use `cmd /c` for npx commands:

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### OAuth 2.0 Authentication

Claude Code supports OAuth 2.0 for MCP servers that require it. When connecting to an OAuth-enabled server, Claude Code handles the entire authentication flow:

```bash
# Connect to an OAuth-enabled MCP server (interactive flow)
claude mcp add --transport http my-service https://my-service.example.com/mcp

# Pre-configure OAuth credentials for non-interactive setup
claude mcp add --transport http my-service https://my-service.example.com/mcp \
  --client-id "your-client-id" \
  --client-secret "your-client-secret" \
  --callback-port 8080
```

| Feature | Description |
|---------|-------------|
| **Interactive OAuth** | Use `/mcp` to trigger the browser-based OAuth flow |
| **Pre-configured OAuth clients** | Built-in OAuth clients for common services like Notion, Stripe, and others (v2.1.30+) |
| **Pre-configured credentials** | `--client-id`, `--client-secret`, `--callback-port` flags for automated setup |
| **Token storage** | Tokens are stored securely in your system keychain |
| **Step-up auth** | Supports step-up authentication for privileged operations |
| **Discovery caching** | OAuth discovery metadata is cached for faster reconnections |
| **Metadata override** | `oauth.authServerMetadataUrl` in `.mcp.json` to override default OAuth metadata discovery |

#### Overriding OAuth Metadata Discovery

If your MCP server returns errors on the standard OAuth metadata endpoint (`/.well-known/oauth-authorization-server`) but exposes a working OIDC endpoint, you can tell Claude Code to fetch OAuth metadata from a specific URL. Set `authServerMetadataUrl` in the `oauth` object of your server config:

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

The URL must use `https://`. This option requires Claude Code v2.1.64 or later.

#### Authentication Startup Notice and Dynamic-Header Refresh (v2.1.193)

- **Startup auth notice (v2.1.193+)**: At startup, Claude Code surfaces a notice listing any MCP servers that still need authentication, so a server that needs a login isn't left silently non-working.
- **`headersHelper` auto-refresh (v2.1.193+)**: If you supply custom auth via a `headersHelper`, the helper is re-invoked automatically when a server returns HTTP 401 or 403. Credentials refresh on the fly without a manual reconnect. See [Use dynamic headers for custom authentication](https://code.claude.com/docs/en/mcp).

> **Warning** (v2.1.238): A `headersHelper` in a project `.mcp.json`, and inline MCP servers in project or `--add-dir` agent files, now require that folder's trust dialog to have been accepted — including under `claude -p`. Such helpers also run **without inherited credential environment variables**; user-, managed-, and claude.ai-scope helpers run from the Claude config directory instead. A project config that relied on inherited credentials or on running untrusted will stop working until you accept the trust dialog and supply credentials another way.

### Claude.ai MCP Connectors

MCP servers configured in your Claude.ai account are automatically available in Claude Code. This means any MCP connections you set up through the Claude.ai web interface will be accessible without additional configuration.

Claude.ai MCP connectors are also available in `--print` mode (v2.1.83+), enabling non-interactive and scripted usage.

> **Startup note (v2.1.117+):** Concurrent connect is the default when both local and claude.ai MCP servers are configured (previously serial), reducing startup latency when multiple servers are in use.

To disable Claude.ai MCP servers in Claude Code, set the `ENABLE_CLAUDEAI_MCP_SERVERS` environment variable to `false`:

```bash
ENABLE_CLAUDEAI_MCP_SERVERS=false claude
```

> **Note:** This feature is only available for users logged in with Claude.ai accounts.
