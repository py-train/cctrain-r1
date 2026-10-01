## Hook Events

| Event | When Triggered | Matcher Input | Can Block | Common Use |
|-------|---------------|---------------|-----------|------------|
| **SessionStart** | Session begins/resumes/clear/compact | startup/resume/clear/compact/fork | No | Environment setup |
| **Setup** | Initial environment setup (one-time per session) | (none) | No | Provision tooling, install deps |
| **InstructionsLoaded** | After CLAUDE.md or rules file loaded | (none) | No | Modify/filter instructions |
| **UserPromptSubmit** | User submits prompt | (none) | Yes | Validate prompts |
| **UserPromptExpansion** | User prompt is expanded (e.g., `@` mentions, slash commands resolved) | (none) | Yes | Transform or inspect expanded prompt |
| **PreToolUse** | Before tool execution | Tool name | Yes (allow/deny/ask/defer) | Validate, modify inputs |
| **PermissionRequest** | Permission dialog shown | Tool name | Yes | Auto-approve/deny |
| **PermissionDenied** | User denies a permission prompt | Tool name | No | Logging, analytics, policy enforcement |
| **PostToolUse** | After tool succeeds | Tool name | No | Add context, feedback |
| **PostToolUseFailure** | Tool execution fails | Tool name | No | Error handling, logging |
| **PostToolBatch** | After a batch of tool uses completes | (none) | No | Aggregate reporting, batched validation |
| **Notification** | Notification sent | Notification type | No | Custom notifications |
| **MessageDisplay** | While assistant message text is displayed | (none) | No | Transform or hide displayed message text (v2.1.152) |
| **SubagentStart** | Subagent spawned | Agent type name | No | Subagent setup |
| **SubagentStop** | Subagent finishes | Agent type name | Yes | Subagent validation |
| **Stop** | Claude finishes responding | (none) | Yes | Task completion check |
| **StopFailure** | API error ends turn | (none) | No | Error recovery, logging |
| **TeammateIdle** | Agent team teammate idle | (none) | Yes | Teammate coordination |
| **TaskCompleted** | Task marked complete | (none) | Yes | Post-task actions |
| **TaskCreated** | Task created via TaskCreate | (none) | No | Task tracking, logging |
| **ConfigChange** | Config file changes | (none) | Yes (except policy) | React to config updates |
| **CwdChanged** | Working directory changes | (none) | No | Directory-specific setup |
| **DirectoryAdded** | New working directory registered mid-session via `/add-dir` or the SDK `register_repo_root` control request (v2.1.219) | (none) | No | Set up tooling for a newly added directory |
| **FileChanged** | Watched file changes | (none) | No | File monitoring, rebuild |
| **PreCompact** | Before context compaction | manual/auto | No | Pre-compact actions |
| **PostCompact** | After compaction completes | (none) | No | Post-compact actions |
| **PreModelSwitch** | Before Claude Code applies a requested model switch | Canonical name of the model being switched to (from `to_model`) | Yes | Gate or veto model changes |
| **PostModelSwitch** | After the session's model changes, including changes Claude Code makes itself (such as restoring the model on resume) | Canonical name of the model switched to (from `to_model`) | No | Log or react to model changes |
| **WorktreeCreate** | Worktree being created | (none) | Yes (path return) | Worktree initialization |
| **WorktreeRemove** | Worktree being removed | (none) | No | Worktree cleanup |
| **Elicitation** | MCP server requests user input | (none) | Yes | Input validation |
| **ElicitationResult** | User responds to elicitation | (none) | Yes | Response processing |
| **SessionEnd** | Session terminates | (none) | No | Cleanup, final logging |

`PreModelSwitch` and `PostModelSwitch` require v2.1.251 or later. Both receive `from_model` and `to_model`; the matcher is evaluated against the canonical name derived from `to_model` (e.g. `claude-opus-5`, `.*opus.*`). Their `command`, `http`, and `mcp_tool` timeout default is lowered to 30 seconds.

> **`TaskCreated` and `TaskCompleted` need the todo tools enabled (v2.1.233).** These two
> events fire from the todo/task-tracking tools (`TaskCreate`/`Get`/`Update`/`List`,
> `TodoWrite`), which are **no longer available on Opus 4.8, Sonnet 5, Fable 5, Mythos 5,
> and newer models**. On those models the hooks are still valid configuration but simply
> never fire — you get no output and no error. Set `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` to
> bring the tools, and therefore the events, back.

> **PostToolUse duration (v2.1.119):** `PostToolUse` and `PostToolUseFailure` hook inputs now include `duration_ms` — see the [PostToolUse](#posttooluse) section for details.

### PreToolUse

Runs after Claude creates tool parameters and before processing. Use this to validate or modify tool inputs.

**Configuration:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py"
          }
        ]
      }
    ]
  }
}
```

**Common matchers:** `Task`, `Bash`, `Glob`, `Grep`, `Read`, `Edit`, `Write`, `WebFetch`, `WebSearch`

**Output control:**
- `permissionDecision`: `"allow"`, `"deny"`, `"ask"`, or `"defer"`
  - `"allow"` skips the permission prompt (except for tools that require user interaction, and connector tools your organization set to `ask`)
  - `"deny"` prevents the tool call
  - `"ask"` prompts the user to confirm
  - `"defer"` exits gracefully so the tool can be resumed later; `permissionDecisionReason`, `updatedInput` and `additionalContext` are all ignored for this value
  - Deny and ask rules are still evaluated regardless of what the hook returns. When multiple `PreToolUse` hooks disagree, precedence is `deny` > `defer` > `ask` > `allow`
- `permissionDecisionReason`: Explanation for decision. Shown to the user (not Claude) for `"allow"` and `"ask"`; shown to Claude for `"deny"`; ignored for `"defer"`
- `updatedInput`: Modified tool input parameters

### PostToolUse

Runs immediately after tool completion. Use for verification, logging, or providing context back to Claude.

**Configuration:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py"
          }
        ]
      }
    ]
  }
}
```

**Output control:**
- `"block"` decision prompts Claude with feedback
- `additionalContext`: Context added for Claude

**Additional input fields (v2.1.119):**

| Field | Type | Description |
|-------|------|-------------|
| `duration_ms` | number | Tool execution time in milliseconds. Excludes time spent in permission prompts and PreToolUse hook execution. Available on both `PostToolUse` and `PostToolUseFailure` hooks. |

#### Recoverable blocks (`continueOnBlock`, v2.1.139)

By default, a `PostToolUse` hook that returns `"decision": "block"` aborts the current turn. Set `"continueOnBlock": true` on the hook to instead surface the rejection back to Claude as a `tool_result`, so the model can read the feedback and retry or adjust.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/policy-check.py",
            "continueOnBlock": true
          }
        ]
      }
    ]
  }
}
```

Use this when the hook's `reason` is something Claude can act on (e.g., "this file is read-only; write somewhere else"); leave it off when a block must halt the turn entirely.

### UserPromptSubmit

Runs when user submits a prompt, before Claude processes it.

**Configuration:**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

**Output control:**
- `decision`: `"block"` to prevent processing
- `reason`: Explanation if blocked
- `additionalContext`: Context added to prompt

### Stop and SubagentStop

Run when Claude finishes responding (Stop) or a subagent completes (SubagentStop). Supports prompt-based evaluation for intelligent task completion checking.

**Additional input field:** Both `Stop` and `SubagentStop` hooks receive a `last_assistant_message` field in their JSON input, containing the final message from Claude or the subagent before stopping. This is useful for evaluating task completion.

**Configuration:**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude completed all requested tasks.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

> **Safety cap on consecutive blocks (v2.1.143)**: If a `Stop` hook returns `"decision": "block"` (or sets `continue: false`) **8 times in a row** for the same turn, Claude Code short-circuits the loop and ends the session with a warning. Override the threshold with the env var `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP=<integer>` (set to `0` to disable the cap entirely). This prevents a buggy Stop hook from looping the session forever.

**Return field (v2.1.163):** A `Stop` or `SubagentStop` hook can return `hookSpecificOutput.additionalContext` to give Claude feedback and **continue the turn without surfacing an error label**. Previously, influencing the model from a Stop hook was awkward; now the hook can inject context cleanly, avoiding the error-label behavior of older feedback paths (such as `"decision": "block"`).

```json
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "Reminder: run the test suite before declaring done."
  }
}
```

### SubagentStart

Runs when a subagent begins execution. The matcher input is the agent type name, allowing hooks to target specific subagent types.

**Configuration:**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "code-review",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/subagent-init.sh"
          }
        ]
      }
    ]
  }
}
```

### SessionStart

Runs when session starts or resumes. Can persist environment variables.

**Matchers:** `startup`, `resume`, `clear`, `compact`, `fork`

> **v2.1.214 update**: A forked session now reports source `"fork"` — previously it reported `"resume"`.

**Special feature:** Use `CLAUDE_ENV_FILE` to persist environment variables (also available in `CwdChanged` and `FileChanged` hooks):

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=development' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

**Session-scoped outputs (v2.1.152):** A `SessionStart` hook can return JSON to re-scan skills and set the session title:

```json
{
  "reloadSkills": true,
  "hookSpecificOutput": {
    "sessionTitle": "Payments migration"
  }
}
```

Top-level `reloadSkills: true` triggers a skill re-scan in the same session (the same action as the `/reload-skills` command), making skills the hook just installed available immediately. `hookSpecificOutput.sessionTitle` sets the session's display title on startup and resume.

### SessionEnd

Runs when session ends to perform cleanup or final logging. Cannot block termination.

**Reason field values:**
- `clear` - User cleared the session
- `logout` - User logged out
- `prompt_input_exit` - User exited via prompt input
- `other` - Other reason

**Configuration:**
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-cleanup.sh\""
          }
        ]
      }
    ]
  }
}
```

### Notification Event

Updated matchers for notification events:
- `permission_prompt` - Permission request notification
- `idle_prompt` - Idle state notification
- `auth_success` - Authentication success
- `elicitation_dialog` - Dialog shown to user
- `agent_needs_input` - Background agent needs input (v2.1.198)
- `agent_completed` - Background agent finished (v2.1.198)

### PreModelSwitch

Runs **before** Claude Code applies a requested model switch — for example when you run `/model`, or when a component asks for a different model. Requires v2.1.251 or later.

**Matchers:** the canonical name of the model being switched to, derived from `to_model`. Match an exact model (`claude-opus-5`) or a family with a regex (`.*opus.*`).

**Input fields:** in addition to the common fields, the hook receives `from_model` (the model in use before the switch) and `to_model` (the model requested).

**Can block:** yes. Exit code `2` blocks the switch and shows stderr as an error, so the session keeps its current model. Use this to gate or veto model changes — for example, to keep a cost-sensitive project off the most expensive model.

**Timeout:** this event lowers the `command`, `http`, and `mcp_tool` default timeout to 30 seconds.

**Configuration:**
```json
{
  "hooks": {
    "PreModelSwitch": [
      {
        "matcher": ".*opus.*",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/gate-model-switch.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# gate-model-switch.sh - refuse a switch to Opus on this project
input=$(cat)
to_model=$(echo "$input" | jq -r '.to_model')

if [[ "$to_model" == *opus* ]]; then
  echo "This project is budgeted for Sonnet; staying on the current model." >&2
  exit 2
fi

exit 0
```

### PostModelSwitch

Runs **after** the session's model has changed. It also fires for changes Claude Code makes itself — such as restoring the previously selected model when you resume a session — not only for switches you request. Requires v2.1.251 or later.

**Matchers:** same as `PreModelSwitch` — the canonical name derived from `to_model`.

**Input fields:** `from_model` and `to_model`, alongside the common fields.

**Can block:** no. The switch has already happened; the hook can only observe and react.

**Timeout:** this event lowers the `command`, `http`, and `mcp_tool` default timeout to 30 seconds.

**Configuration:**
```json
{
  "hooks": {
    "PostModelSwitch": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/log-model-switch.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# log-model-switch.sh - append every model change to a session log
input=$(cat)
from=$(echo "$input" | jq -r '.from_model')
to=$(echo "$input" | jq -r '.to_model')

echo "$(date -Iseconds) $from -> $to" >> ~/.claude/model-switches.log
exit 0
```

