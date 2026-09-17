# Builtin Agents

Claude Code includes several built-in subagents that are always available:

| Agent | Model | Purpose |
|-------|-------|---------|
| **general-purpose** | Inherits | Complex, multi-step tasks |
| **Plan** | Inherits | Research for plan mode |
| **Explore** | Inherits (capped at Opus) | Read-only codebase exploration (quick/medium/very thorough) |
| **claude** | Inherits | Catch-all for tasks that don't fit a more specialized agent; has every tool available to subagents. Also the default agent for a dispatched background session |
| **statusline-setup** | Sonnet | Runs when you use `/statusline` to configure your status line |
| **claude-code-guide** | Haiku | Answers questions about Claude Code features |

---

## General-Purpose Subagent

| Property | Value |
|----------|-------|
| **Model** | Inherits from parent |
| **Tools** | All tools |
| **Purpose** | Complex research tasks, multi-step operations, code modifications |

**When used**: Tasks requiring both exploration and modification with complex reasoning.

## Plan Subagent

| Property | Value |
|----------|-------|
| **Model** | Inherits from parent |
| **Tools** | Read, Glob, Grep, Bash |
| **Purpose** | Used automatically in plan mode to research codebase |

**When used**: When Claude needs to understand the codebase before presenting a plan.

## Explore Subagent

| Property | Value |
|----------|-------|
| **Model** | Inherits the session model, capped at Opus (v2.1.198). Set `model: haiku` to keep it fast and cheap |
| **Mode** | Strictly read-only |
| **Tools** | Glob, Grep, Read, Bash (read-only commands only) |
| **Purpose** | Fast codebase searching and analysis |

**When used**: When searching/understanding code without making changes.

**Thoroughness Levels** - Specify the depth of exploration:
- **"quick"** - Fast searches with minimal exploration, good for finding specific patterns
- **"medium"** - Moderate exploration, balanced speed and thoroughness, default approach
- **"very thorough"** - Comprehensive analysis across multiple locations and naming conventions, may take longer

## Claude Subagent

| Property | Value |
|----------|-------|
| **Model** | Inherits from parent |
| **Tools** | Every tool available to subagents |
| **Purpose** | Catch-all agent for tasks that don't fit a more specialized agent |

**When used**: When a task doesn't match a more specialized built-in agent. It is also the default agent for a dispatched background session; which permission mode it starts in depends on how that session was started.

## Statusline Setup Subagent

| Property | Value |
|----------|-------|
| **Model** | Sonnet |
| **Tools** | Read, Write, Bash |
| **Purpose** | Configure the Claude Code status line display |

**When used**: When setting up or customizing the status line.

## Claude Code Guide Subagent (`claude-code-guide`)

| Property | Value |
|----------|-------|
| **Model** | Haiku (fast, low-latency) |
| **Tools** | Read-only |
| **Purpose** | Answer questions about Claude Code features and usage |

**When used**: When users ask questions about how Claude Code works or how to use specific features.
