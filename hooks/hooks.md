# Hooks

Hooks are automated scripts that execute in response to specific events during Claude Code sessions. They enable automation, validation, permission management, and custom workflows.

## Overview

Hooks are automated actions (shell commands, HTTP webhooks, LLM prompts, MCP tool calls, or subagent evaluations) that execute automatically when specific events occur in Claude Code. They receive JSON input and communicate results via exit codes and JSON output.

**Key features:**
- Event-driven automation
- JSON-based input/output
- Support for `command`, `http`, `mcp_tool`, `prompt`, and `agent` hook types
- Pattern matching for tool-specific hooks

## Configuration

Hooks are configured in settings files with a specific structure:

- `~/.claude/settings.json` - User settings (all projects)
- `.claude/settings.json` - Project settings (shareable, committed)
- `.claude/settings.local.json` - Local project settings (not committed)
- Managed policy - Organization-wide settings
- Plugin `hooks/hooks.json` - Plugin-scoped hooks
- Skill/Agent frontmatter - Component lifetime hooks

### Basic Configuration Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

[Key Fields](./hooks-extras.md#key-fields)


### Matcher Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| Exact string | Matches specific tool | `"Write"` |
| Regex pattern | Matches multiple tools | `"Edit\|Write"` |
| Comma-separated | Matches any listed tool (v2.1.191+) | `"Write,Edit"` |
| Wildcard | Matches all tools | `"*"` or `""` |
| MCP tools | Server and tool pattern | `"mcp__memory__.*"` |


### Narrowing with `if` conditions (tool-argument paths)

The `matcher` field selects a hook by **tool name** (`"Write"`, `"Edit|Write"`, `"*"`). To filter more narrowly by the tool's **arguments** — for example, to run a hook only when an edit touches `src/`, or to guard reads of secret files — add an `if` condition to an individual hook handler. This is distinct from the tool-name matcher: the `matcher` decides *which tool*, the `if` decides *which call*.

`if` uses [permission-rule syntax](https://code.claude.com/docs/en/permissions) (`ToolName(pattern)`), evaluated against the tool name **and** its arguments together. For `Read`/`Edit`/`Write`, the path pattern follows gitignore semantics, with the same anchors as permission rules: a bare name like `.env` matches at any depth, `src/**` is relative to the current directory, `/src/**` to the project root, `~/...` to your home directory, and `//...` is an absolute filesystem path.

The `if` field sits at the **hook-handler level** — a sibling of `type` and `command`, inside the `hooks` array — not on `matcher`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "if": "Edit(src/**)",
            "command": "./hooks/lint-src.sh"
          }
        ]
      },
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "if": "Read(.env)",
            "command": "./hooks/block-secret-read.sh"
          }
        ]
      }
    ]
  }
}
```

Examples of valid `if` patterns: `Edit(src/**)` (edits under `src/`), `Read(~/.ssh/**)` (reads of any SSH key), `Read(.env)` (any `.env` at or below the current directory), `Bash(git push *)` (only `git push` subcommands).

## Hook Types

Claude Code supports five hook types:

### Command Hooks

The default hook type. Executes a shell command and communicates via JSON stdin/stdout and exit codes.

```json
{
  "type": "command",
  "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py\"",
  "timeout": 60
}
```

#### Exec form (`args`)

> Added in v2.1.139.

Instead of the shell-form `"command": "..."`, a command hook can spawn a binary directly via `execve()` with an `args` array. There is no shell parsing, so path placeholders never need quoting and the configuration is immune to shell-injection bugs.

```json
{
  "type": "command",
  "args": ["python3", "$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py", "--strict"],
  "timeout": 60
}
```

The two forms are **mutually exclusive** — a hook with both `command` and `args` set is rejected at config load. Use `command` when you need pipes, redirection, `&&` chaining, or shell expansions; use `args` when you are calling one binary with arguments.

### HTTP Hooks

> Added in v2.1.63.

Remote webhook endpoints that receive the same JSON input as command hooks. HTTP hooks POST JSON to the URL and receive a JSON response. HTTP hooks are routed through the sandbox when sandboxing is enabled. Environment variable interpolation in URLs requires an explicit `allowedEnvVars` list for security.

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "http",
      "url": "https://my-webhook.example.com/hook",
      "matcher": "Write"
    }]
  }
}
```

**Key properties:**
- `"type": "http"` -- identifies this as an HTTP hook
- `"url"` -- the webhook endpoint URL
- Routed through sandbox when sandbox is enabled
- Requires explicit `allowedEnvVars` list for any environment variable interpolation in the URL

### Prompt Hooks

LLM-evaluated prompts where the hook content is a prompt that Claude evaluates. Primarily used with `Stop` and `SubagentStop` events for intelligent task completion checking.

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude completed all requested tasks.",
  "timeout": 30
}
```

The LLM evaluates the prompt and returns a structured decision (see [Prompt-Based Hooks](#prompt-based-hooks) for details).

### MCP Tool Hooks

> Added in v2.1.118.

The `mcp_tool` type invokes a configured MCP tool directly; configuration references the MCP server and tool name rather than a shell command or URL. This is useful when the validation or reaction logic already lives in an MCP server you have configured.

```json
{
  "matcher": "Edit",
  "hooks": [{
    "type": "mcp_tool",
    "server": "my-mcp-server",
    "tool": "validate_edit"
  }]
}
```

**Key properties:**
- `"type": "mcp_tool"` -- identifies this as an MCP tool hook
- `"server"` -- name of the configured MCP server
- `"tool"` -- the tool name on that server to invoke

The hook input (tool name, tool input, session context) is passed as the MCP tool's arguments.

### Agent Hooks

Subagent-based verification hooks that spawn a dedicated agent to evaluate conditions or perform complex checks. Unlike prompt hooks (single-turn LLM evaluation), agent hooks can use tools and perform multi-step reasoning.

> **Note**: Agent hooks are experimental and may change.

```json
{
  "type": "agent",
  "prompt": "Verify the code changes follow our architecture guidelines. Check the relevant design docs and compare.",
  "timeout": 120
}
```

**Key properties:**
- `"type": "agent"` -- identifies this as an agent hook
- `"prompt"` -- the task description for the subagent
- The agent can use tools (Read, Grep, Bash, etc.) to perform its evaluation
- Returns a structured decision similar to prompt hooks

## Hook Events

Claude Code supports **33 hook events**:

### [Hook Events Table](./hook-events.md#hook-events)

### [PreToolUse](./hook-events.md#pretooluse)

### [PostToolUse](./hook-events.md#posttooluse)

### [Stop and SubagentStop](./hook-events.md#stop-and-subagentstop)



## Component-Scoped Hooks

Hooks can be attached to specific components (skills, agents, commands) in their frontmatter:

**In SKILL.md, agent.md, or command.md:**

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true  # Only run once per session
---
```

**Supported events for component hooks:** `PreToolUse`, `PostToolUse`, `Stop`

This allows defining hooks directly in the component that uses them, keeping related code together.

### Hooks in Subagent Frontmatter

When a `Stop` hook is defined in a subagent's frontmatter, it is automatically converted to a `SubagentStop` hook scoped to that subagent. This ensures that the stop hook only fires when that specific subagent completes, rather than when the main session stops.

```yaml
---
name: code-review-agent
description: Automated code review subagent
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify the code review is thorough and complete."
  # The above Stop hook auto-converts to SubagentStop for this subagent
---
```


## Hook Input and Output

### JSON Input (via stdin)

All hooks receive JSON input via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_use_id": "toolu_01ABC123...",
  "agent_id": "agent-abc123",
  "agent_type": "main",
  "worktree": "/path/to/worktree",
  "effort": { "level": "medium" }
}
```

**Common fields:**

| Field | Description |
|-------|-------------|
| `session_id` | Unique session identifier |
| `transcript_path` | Path to the conversation transcript file |
| `cwd` | Current working directory |
| `prompt_id` | UUID of the prompt being processed; correlates with the OpenTelemetry `prompt.id` attribute (v2.1.196) |
| `hook_event_name` | Name of the event that triggered the hook |
| `agent_id` | Identifier of the agent running this hook |
| `agent_type` | Type of agent (`"main"`, subagent type name, etc.) |
| `worktree` | Path to the git worktree, if the agent is running in one |
| `effort.level` | (v2.1.133+) Active effort level: `low`, `medium`, `high`, `xhigh`, or `max` |

### Exit Codes

| Exit Code | Meaning | Behavior |
|-----------|---------|----------|
| **0** | Success | Continue, parse JSON stdout |
| **2** | Blocking error | Block operation, stderr shown as error |
| **Other** | Non-blocking error | Continue, stderr shown in verbose mode |

### JSON Output (stdout, exit code 0)

```json
{
  "continue": true,
  "stopReason": "Optional message if stopping",
  "suppressOutput": false,
  "systemMessage": "Optional warning message",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "File is in allowed directory",
    "updatedInput": {
      "file_path": "/modified/path.js"
    }
  }
}
```



## Prompt-Based Hooks

For `Stop` and `SubagentStop` events, you can use LLM-based evaluation:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if all tasks are complete. Return your decision.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**LLM Response Schema:**
```json
{
  "decision": "approve",
  "reason": "All tasks completed successfully",
  "continue": false,
  "stopReason": "Task complete"
}
```

## [Examples](./examples.md)



## Debugging

### Enable Debug Mode

Run Claude with debug flag for detailed hook logs:

```bash
claude --debug
```

### Verbose Mode

Use `Ctrl+O` in Claude Code to enable verbose mode and see hook execution progress.

### Test Hooks Independently

```bash
# Test with sample JSON input
echo '{"tool_name": "Bash", "tool_input": {"command": "ls -la"}}' | python3 .claude/hooks/validate-bash.py

# Check exit code
echo $?
```


## Hook Execution Details

| Aspect | Behavior |
|--------|----------|
| **Timeout** | 600 seconds default for command/http/mcp_tool (30 for prompt, 60 for agent); configurable per hook |
| **Parallelization** | All matching hooks run in parallel |
| **Deduplication** | Identical hook commands deduplicated |
| **Environment** | Runs in current directory with Claude Code's environment |

## Troubleshooting

### Hook Not Executing
- Verify JSON configuration syntax is correct
- Check matcher pattern matches the tool name
- Ensure script exists and is executable: `chmod +x script.sh`
- Run `claude --debug` to see hook execution logs
- Verify hook reads JSON from stdin (not command args)

### Hook Blocks Unexpectedly
- Test hook with sample JSON: `echo '{"tool_name": "Write", ...}' | ./hook.py`
- Check exit code: should be 0 for allow, 2 for block
- Check stderr output (shown on exit code 2)

### JSON Parsing Errors
- Always read from stdin, not command arguments
- Use proper JSON parsing (not string manipulation)
- Handle missing fields gracefully



## Additional Resources

- **[Official Hooks Documentation](https://code.claude.com/docs/en/hooks)** - Complete hooks reference
- **[CLI Reference](https://code.claude.com/docs/en/cli-reference)** - Command-line interface documentation
- **[Memory Guide](../02-memory/)** - Persistent context configuration

---

**Last Updated**: September 2, 2026
**Claude Code Version**: 2.1.257
**Sources**:
- https://code.claude.com/docs/en/hooks
- https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- https://code.claude.com/docs/en/sub-agents
**Compatible Models**: Claude Fable 5, Claude Opus 5, Claude Sonnet 5, Claude Sonnet 4.6, Claude Opus 4.8, Claude Haiku 4.5
