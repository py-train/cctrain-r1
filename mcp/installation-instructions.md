## Installation Instructions

### Prerequisites
- Node.js and npm installed
- Claude Code CLI installed
- API tokens/credentials for external services

### Step-by-Step Setup

1. **Add your first MCP server** using the CLI (example: GitHub):
```bash
claude mcp add --transport stdio github -- npx @modelcontextprotocol/server-github
```

   Or create a `.mcp.json` file in your project root:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

2. **Set environment variables:**
```bash
export GITHUB_TOKEN="your_github_personal_access_token"
```

3. **Test the connection:**
```bash
claude /mcp
```

4. **Use MCP tools:**
```bash
/mcp__github__list_prs
/mcp__github__create_issue "Title" "Description"
```

### Installation for Specific Services

**GitHub MCP:**
```bash
npm install -g @modelcontextprotocol/server-github
```

**Database MCP:**
```bash
npm install -g @modelcontextprotocol/server-database
```

**Filesystem MCP:**
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Slack MCP:**
```bash
npm install -g @modelcontextprotocol/server-slack
```
