
### Agent Tools

> **Note on Glob/Grep (v2.1.113+):** On native macOS/Linux builds, Glob and Grep are provided as `bfs`/`ugrep` through the Bash tool rather than as separate tools. Windows and npm-JS builds still expose them as standalone tools. Authors can still reference Glob/Grep in `allowedTools`; the backend substitution is transparent.


> **Note**: Since v2.1.243, `--agents` no longer silently ignores invalid JSON or invalid agent definitions — Claude Code exits with a clear error, matching the behavior of `--mcp-config`.



## `claude agents` CLI Command

The `claude agents` command lists all configured agents grouped by source (built-in, user-level, project-level):

```bash
claude agents
```

This command:
- Shows all available agents from all sources
- Groups agents by their source location
- Indicates **overrides** when an agent at a higher priority level shadows one at a lower level (e.g., a project-level agent with the same name as a user-level agent)

### Listing Available Agents

> !! This is no longer true !!

Use the `claude agents` command to list all configured agents from all sources:

```bash
claude agents
```


### Key Behaviors

- **Nested spawning on by default, depth 3 (v2.1.219)** - Subagents can spawn their own subagents up to three layers below the main conversation. Set `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` to change the limit, or `1` to turn nesting off. At the depth limit, Claude Code withholds the `Agent` tool from every subagent except a fork. (History: v2.1.172–v2.1.216 nested by default up to 5 layers with no way to change it; v2.1.217 made nesting opt-in at depth 1; v2.1.219 set the default to 3.) Use the `Agent(agent_type)` restriction syntax (see [Restrict Spawnable Subagents](#restrict-spawnable-subagents)) to control which subagents a given subagent may spawn
- **Background permissions** - Background subagents auto-deny any permissions that are not pre-approved
- **Backgrounding** - Press `Ctrl+B` to background a currently running task
- **Transcripts** - Subagent transcripts are stored at `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`
- **Auto-compaction** - Subagent context auto-compacts at ~95% capacity (override with `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` environment variable)
- **Extended thinking inherited (v2.1.198)** - Subagents and context compaction now inherit the session's extended-thinking configuration (previously always disabled). There is no per-subagent thinking field

## Observability

> **Added in v2.1.139.**

API requests originating from a subagent carry two extra HTTP headers so traces and logs can be correlated back to the dispatching session:

| Header | Description |
|--------|-------------|
| `x-claude-code-agent-id` | UUID of the subagent making the request. |
| `x-claude-code-parent-agent-id` | UUID of the agent that dispatched this subagent (the main agent, or a higher-level subagent in a chain). |

The same identifiers are exposed on `claude_code.llm_request` OpenTelemetry spans as the attributes `claude.code.agent.id` and `claude.code.agent.parent_id`. Use them to:

- Attribute API spend to a specific subagent type rather than the parent session
- Reconstruct a chain of agent invocations after the fact (parent_id forms a tree)
- Alert on runaway subagents (e.g., one `agent.id` accounting for >50% of session spend)

See the OpenTelemetry section in [Advanced Features → Telemetry](../09-advanced-features/README.md) for end-to-end exporter setup.

## Plugin Subagent Security

Plugin-provided subagents have restricted frontmatter capabilities for security. The following fields are **not allowed** in plugin subagent definitions:

- `hooks` - Cannot define lifecycle hooks
- `mcpServers` - Cannot configure MCP servers
- `permissionMode` - Cannot override permission settings

This prevents plugins from escalating privileges or executing arbitrary commands through subagent hooks.

### Subagent Output Scanning (v2.1.210+)

As of v2.1.210, Claude Code scans each subagent's final report for text that imitates the harness's own output format — fake `<system-reminder>`-style tags, fabricated `Human:`/`Assistant:` turns, or mentions of permission-bypass flags and settings-file paths. This defends against prompt injection carried in subagent output, such as a subagent that fetched a malicious web page containing fake control tokens designed to manipulate the parent session.

When the scan flags something, Claude Code neutralizes it — inserting a backslash or an inline marker such as `[harness: subagent output matched instruction-shaped pattern(s): ...]` naming what triggered the scan — and the parent session is expected to treat the flagged text as a finding to relay, not an instruction to follow. The scan is on by default with no documented opt-out. It errs toward flagging: a legitimate subagent report that quotes a real flag name (e.g. `--dangerously-skip-permissions`) verbatim can trigger a marker even though nothing malicious occurred — a false positive is preferable to a missed injection.

### Subagent Concurrency and Depth Limits

> **The per-session spawn cap is gone.** Claude Code capped subagent spawns at 200 per session from v2.1.212, but **v2.1.224 removed that cap** — long-running sessions no longer refuse new agents, and the official subagents reference now states plainly that there is no limit on the total number of subagents Claude can spawn over a session. The `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` variable that overrode it is gone with it.

Two limits on subagent fan-out do still apply, both set by environment variable:

- `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` (v2.1.217) - Maximum number of subagents running **concurrently** at once. Default: 20.
- `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` (v2.1.217) - Maximum **nesting depth** for subagents spawning their own subagents. **Default: 3 since v2.1.219** (was 1 in v2.1.217–v2.1.218). Set this to `1` to disable nesting (see [Key Behaviors](#key-behaviors)).

```bash
export CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS=20
export CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH=5
```

---



