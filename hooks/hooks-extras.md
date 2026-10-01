## Environment Variables

| Variable | Availability | Description |
|----------|-------------|-------------|
| `CLAUDE_PROJECT_DIR` | All hooks | Absolute path to project root |
| `CLAUDE_ENV_FILE` | SessionStart, CwdChanged, FileChanged | File path for persisting env vars |
| `CLAUDE_CODE_REMOTE` | All hooks | `"true"` if running in remote environments |
| `${CLAUDE_PLUGIN_ROOT}` | Plugin hooks | Path to plugin directory |
| `${CLAUDE_PLUGIN_DATA}` | Plugin hooks | Path to plugin data directory |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hooks | Configurable timeout in milliseconds for SessionEnd hooks (overrides default) |
| `CLAUDE_CODE_SESSION_ID` | Bash tool subprocesses (v2.1.132+) | Session UUID; matches the `session_id` field in hook input JSON. Use to correlate bash logs with hook telemetry. |
| `CLAUDE_EFFORT` | Bash tool subprocesses (v2.1.133+) | Active effort level (`low`/`medium`/`high`/`xhigh`/`max`); matches `effort.level` in hook input JSON. |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | Process-wide (v2.1.143+) | Max consecutive Stop-hook blocks before the session ends with a warning (default `8`). Set to `0` to disable the cap. |


## Key fields

| Field | Description | Example |
|-------|-------------|---------|
| `matcher` | Pattern to match tool names (case-sensitive) | `"Write"`, `"Edit\|Write"`, `"*"` |
| `hooks` | Array of hook definitions | `[{ "type": "command", ... }]` |
| `type` | Hook type: `"command"` (bash), `"prompt"` (LLM), `"http"` (webhook), `"mcp_tool"` (MCP tool invocation, v2.1.118+), or `"agent"` (subagent) | `"command"` |
| `command` | Shell command to execute | `"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh"` |
| `timeout` | Optional timeout in seconds. Defaults: 600 for command/http/mcp_tool, 30 for prompt, 60 for agent. | `30` |
| `once` | If `true`, run the hook only once per session | `true` |
| `async` | If `true`, runs in the background without blocking | `true` |
| `asyncRewake` | If `true`, runs in the background and wakes Claude on exit code 2. Implies `async`. | `true` |
| `shell` | Accepts `"bash"` or `"powershell"`. Defaults to `"bash"`, or to `"powershell"` on Windows when Git Bash isn't installed. | `"bash"` |
| `statusMessage` | Custom spinner message displayed while the hook runs | `"Formatting…"` |

> **Note**: Some events lower the default timeout. `UserPromptSubmit` lowers the `command`, `http`, and `mcp_tool` default to 30 seconds, and `MessageDisplay` lowers it to 10 seconds. `SessionEnd` hooks share a 1.5-second budget; if your settings set a longer per-hook `timeout`, Claude Code raises that budget to match, up to 60 seconds.


> **Matchers are matched exactly (v2.1.195+).** A hyphenated identifier (for example an MCP tool name containing a hyphen) no longer accidentally substring-matches a different tool. Comma-separated matchers like `"Write,Edit"` fire on any tool in the list — earlier builds silently never fired them.

**InstructionsLoaded matcher values:**

| Matcher Value | Description |
|---------------|-------------|
| `session_start` | Instructions loaded at session startup |
| `nested_traversal` | Instructions loaded during nested directory traversal |
| `path_glob_match` | Instructions loaded via path glob pattern matching |


> **v2.1.214 update**: A single-segment `dir/**` pattern in a hook `if` condition (like `Edit(src/**)`) now matches only `<cwd>/dir` — not that directory at any depth in the tree. Previously `src/**` also matched `foo/src/**`. Use `**/dir/**` if you need any-depth matching. **Important**: this narrowing applies only to hook `if:` conditions and allow-rule auto-approval — deny/ask permission rules still match `dir/**` at any depth.

**Workspace trust required (v2.1.218):** Frontmatter hooks in a **project** subagent now require workspace trust acceptance for the folder the agent file came from before they run. Before v2.1.218, these hooks could run from folders you hadn't trusted. See the [subagents documentation](https://code.claude.com/docs/en/sub-agents#hooks-in-subagent-frontmatter) for which scopes are exempt.

## PermissionRequest Event

Handles permission requests with custom output format:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": {},
      "message": "Custom message",
      "interrupt": false
    }
  }
}
```

> **Scope (v2.1.121+):** `hookSpecificOutput.updatedToolOutput` is now honored for **all** tools, not just MCP tools. A `PostToolUse` hook on `Bash`, `Edit`, `Read`, etc. can rewrite the tool's output before Claude sees it — useful for redacting secrets, normalizing diffs, or filtering noisy command output. Example (strip ANSI color codes from a `Bash` output):
>
> ```json
> {
>   "hookSpecificOutput": {
>     "hookEventName": "PostToolUse",
>     "updatedToolOutput": "<plain-text output with ANSI escapes removed>"
>   }
> }
> ```

> **`retry` (PermissionDenied)**: Use JSON `hookSpecificOutput.retry: true` to tell the model it may retry the denied tool call.

> **Deprecated `PreToolUse` decision form**: For `PreToolUse`, the top-level `decision` and `reason` fields are **deprecated** — use `hookSpecificOutput.permissionDecision` (`allow` / `deny` / `ask` / `defer`) and `permissionDecisionReason` instead. Precedence among decisions is `deny` > `defer` > `ask` > `allow`. Note also that `suppressOutput` is accepted but **has no effect**.

#### `terminalSequence` (v2.1.141)

Hooks can emit raw OSC (operating system command) escape sequences by setting `terminalSequence` in the JSON output. The host writes the sequence to its controlling terminal when the hook returns — useful for desktop notifications, window-title updates, and terminal bells without requiring a TTY of your own.

| Field | Type | Description |
|-------|------|-------------|
| `terminalSequence` | string | Raw escape sequence (typically OSC 9 / OSC 0 / OSC 777). Written to the host terminal verbatim. |

Example — fire an OSC 9 desktop notification when a long task completes:

```json
{
  "terminalSequence": "]9;Task complete"
}
```


Configure it on a `Stop` hook so the notification fires when Claude finishes a turn. Sequence support is terminal-dependent; Kitty/iTerm2/Windows Terminal honor OSC 9.

