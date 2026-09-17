### Optional Frontmatter Fields

```yaml
---
name: my-skill
description: What this skill does and when to use it
argument-hint: "[filename] [format]"        # Hint for autocomplete
disable-model-invocation: true              # Only user can invoke
user-invocable: false                       # Hide from slash menu
allowed-tools: Read, Grep, Glob             # Restrict tool access
disallowed-tools: Write, Edit               # Remove specific tools while active (v2.1.152)
model: opus                                 # Specific model to use
effort: high                                # Effort level override (low, medium, high, xhigh, max)
context: fork                               # Run in isolated subagent
agent: Explore                              # Which agent type (with context: fork)
background: false                           # Fork skills run in background (default true); false = foreground
shell: bash                                 # Shell for commands: bash (default) or powershell
hooks:                                      # Skill-scoped hooks
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
paths: "src/api/**/*.ts"               # Glob patterns limiting when skill activates
---
```

| Field | Description |
|-------|-------------|
| `name` | Lowercase letters, numbers, hyphens only (max 64 chars). Cannot contain "anthropic" or "claude". |
| `description` | What the Skill does AND when to use it. The combined `description` + `when_to_use` text is truncated at 1,536 chars in the skill listing (configurable via `skillListingMaxDescChars`). Critical for auto-invocation matching. |
| `when_to_use` | Additional context for when Claude should invoke the skill. Appended to `description` in the skill listing and counts toward the 1,536-character cap. |
| `argument-hint` | Hint shown in the `/` autocomplete menu (e.g., `"[filename] [format]"`). |
| `disable-model-invocation` | `true` = only the user can invoke via `/name`. Claude will never auto-invoke. |
| `user-invocable` | `false` = hidden from the `/` menu. Only Claude can invoke it automatically. |
| `allowed-tools` | Comma-separated list of tools the skill may use without permission prompts. |
| `disallowed-tools` | Comma-separated list of tools to remove while the skill is active (complements `allowed-tools`). Added v2.1.152. |
| `model` | Model override while the skill is active (e.g., `opus`, `sonnet`). |
| `effort` | Effort level override while the skill is active: `low`, `medium`, `high`, `xhigh`, or `max` — all five are supported on Opus 5, Sonnet 5, Opus 4.8, and Opus 4.7. The default effort is `high` on every model that supports effort, except Opus 4.7 which defaults to `xhigh`. |
| `context` | `fork` to run the skill in a forked subagent context with its own context window. |
| `agent` | Subagent type when `context: fork` (e.g., `Explore`, `Plan`, `general-purpose`). |
| `background` | Only meaningful with `context: fork`. Defaults to `true` for `context: fork` skills, so they run in the background; set `false` to run them in the foreground. Added v2.1.218. |
| `shell` | Shell used for `` !`command` `` substitutions and scripts: `bash` (default) or `powershell`. |
| `hooks` | Hooks scoped to this skill's lifecycle (same format as global hooks). |
| `paths` | Glob patterns that limit when the skill is auto-activated. Comma-separated string or YAML list. Same format as path-specific rules. |
| `arguments` | Declares the arguments the skill accepts, for autocomplete and argument substitution. |
| `metadata` | Free-form key/value map for your own bookkeeping (e.g. `version`, `author`). Claude Code passes it through. |
| `license` | License identifier for the skill (e.g. `MIT`). |
| `compatibility` | Free-text compatibility statement, up to 500 characters. Claude Code accepts it but does not act on it. |

> **Note**: Only `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools` are valid for skills uploaded to claude.ai or created through the Skills API. The other fields in this table are Claude Code-specific.

Since v2.1.218, boolean frontmatter fields also accept `yes`/`no`, `on`/`off`, and `1`/`0` (case-insensitive) in addition to `true`/`false`.
