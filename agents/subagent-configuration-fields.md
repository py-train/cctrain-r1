# SubAgent Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase letters and hyphens). Lookup is normalized (case- and separator-insensitive — see below), but a name containing `:` is **rejected** as of v2.1.218: `:` is reserved for plugin namespacing |
| `description` | Yes | Natural language description of purpose. Include "use PROACTIVELY" to encourage automatic invocation |
| `tools` | No | Comma-separated list of specific tools. Omit to inherit all tools. Supports `Agent(agent_name)` syntax to restrict spawnable subagents |
| `disallowedTools` | No | Comma-separated list of tools the subagent must not use |
| `model` | No | Model to use: `sonnet`, `opus`, `haiku`, full model ID, or `inherit`. Defaults to configured subagent model |
| `permissionMode` | No | `manual` (renamed from `default` in v2.1.200 — `default` is still accepted as the older name), `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan`, `auto`. As of v2.1.212, the Task tool's `mode` invocation parameter is deprecated and ignored — subagents inherit the parent session's permission mode by default unless overridden here |
| `maxTurns` | No | Maximum number of agentic turns the subagent can take |
| `skills` | No | Comma-separated list of skills to preload. Injects full skill content into the subagent's context at startup. **v2.1.133+:** subagents also discover project, user, and plugin skills via the Skill tool — same catalog as the main session, no longer limited to their own embedded set. |
| `mcpServers` | No | MCP servers to make available to the subagent |
| `hooks` | No | Component-scoped hooks (PreToolUse, PostToolUse, Stop) |
| `memory` | No | Persistent memory directory scope: `user`, `project`, or `local` |
| `background` | No | Subagents already run in the background by default (v2.1.198). Set to `true` to *force* background always and prevent inline execution |
| `effort` | No | Reasoning effort level: `low`, `medium`, `high`, `xhigh`, or `max`. Overrides the session effort level; available levels depend on the model |
| `isolation` | No | Set to `worktree` to give the subagent its own git worktree |
| `initialPrompt` | No | Auto-submitted first turn when the subagent runs as the main agent |
| `color` | No | Display color for the subagent in the task list and transcript. Accepts `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, or `cyan` |
| `experimental` | No | Experimental settings block (v2.1.248+). `experimental.cacheTtl` sets the cache TTL for this subagent — `"5m"` or `"1h"` |

## Subagent Model Environment Variables

Two environment variables affect which model a subagent runs on:

| Variable | Version | Description |
|----------|---------|-------------|
| `CLAUDE_CODE_SUBAGENT_MODEL` | — | Sets the model used for subagents |
| `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` | v2.1.257+ | Set to `1` to force the subagent model over a subagent's frontmatter `model:` |

> **Precedence changed in v2.1.251**: before that release, `CLAUDE_CODE_SUBAGENT_MODEL` came first and overrode agent frontmatter — including `model: inherit`. From v2.1.251 on, a subagent's own `model:` frontmatter wins. Set `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (v2.1.257+) when you want the environment variable to override frontmatter again, for example to pin an entire evaluation run to one model.

## Main-Thread Agent Frontmatter Honoring (v2.1.117+/v2.1.119+)

When an agent is invoked as the main-thread agent (via `claude --agent <name>` or `--print` mode), these frontmatter fields are honored:

| Field | Version | Notes |
|-------|---------|-------|
| `mcpServers` | v2.1.117+ | Loaded when agent is invoked as main-thread agent via `claude --agent <name>` |
| `permissionMode` | v2.1.119+ | Honored for built-in agents via `--agent <name>` |
| `tools` / `disallowedTools` | v2.1.119+ | Honored in `--print` mode (non-interactive/scripted usage) |
