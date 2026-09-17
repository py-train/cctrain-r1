## Troubleshooting

### Start With the Error Output (v2.1.219+)

If a server won't connect, run `claude mcp list` (or `/mcp` inside a session) before changing anything. Claude Code now prints the **HTTP status code and the server's error text** next to the failing server, so you get `401 Unauthorized` or `404 Not Found` instead of a generic "failed to connect":

```bash
# Shows connection status plus HTTP status and error text for failures
claude mcp list
```

Read the status first — it tells you which fix applies:

- `401` / `403` → bad or expired credentials; re-authenticate with `claude mcp login <name>`
- `404` → wrong URL (a missing `/mcp` or `/sse` path suffix is the usual cause)
- `5xx` / timeout → the remote server is down; see [Connection Timeout](#connection-timeout)

### Hidden Whitespace in Config Values (v2.1.219+)

Claude Code warns when an MCP config value has **leading or trailing whitespace**. This is a common and hard-to-spot cause of auth failures: a token copy-pasted from a browser or a chat message often carries a trailing space or newline, which gets sent verbatim in the `Authorization` header and fails as an invalid credential.

If you see this warning, re-check the value in `.mcp.json` (or the env var it expands from) and trim it:

```bash
# Reveals hidden leading/trailing whitespace between the delimiters
printf '[%s]\n' "$GITHUB_TOKEN"
```

### Skipped Servers in Headless Runs (v2.1.219+)

Servers passed via `--mcp-config` that fail config validation are **skipped** rather than aborting the run, so a headless script can appear to work while missing half its tools. Claude Code now reports which ones were dropped:

- **Headless / `-p` runs**: the stream-json `init` event carries an `mcp_server_errors` field listing every skipped entry. Check it before trusting the run's output.
- **Interactive terminal runs**: the same problem is printed as a startup warning when the session begins.

```bash
# Inspect skipped --mcp-config entries in a headless run
claude -p "list my tools" --mcp-config ./servers.json \
  --output-format stream-json --verbose \
  | jq -r 'select(.type == "system" and .subtype == "init") | .mcp_server_errors'
```

### MCP Server Not Found
```bash
# Verify MCP server is installed
npm list -g @modelcontextprotocol/server-github

# Install if missing
npm install -g @modelcontextprotocol/server-github
```

### Authentication Failed
```bash
# Verify environment variable is set
echo $GITHUB_TOKEN

# Re-export if needed
export GITHUB_TOKEN="your_token"

# Verify token has correct permissions
# Check GitHub token scopes at: https://github.com/settings/tokens
```

### Connection Timeout
- Check network connectivity: `ping api.github.com`
- Verify API endpoint is accessible
- Check rate limits on API
- Try increasing timeout in config
- Check for firewall or proxy issues

### MCP Server Crashes
- Check MCP server logs: `~/.claude/logs/`
- Verify all environment variables are set
- Ensure proper file permissions
- Try reinstalling the MCP server package
- Check for conflicting processes on the same port
