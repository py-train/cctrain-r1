## Forked Subagents

Forked subagents (`context: fork`) inherit the parent agent's full conversation context at the moment of forking, rather than starting with a clean slate. This is useful for exploring alternative paths without losing the work done so far.

> **Availability**: GA in v2.1.117. **Since v2.1.232, fork mode is on by default in interactive sessions** — on every build, first-party or not. It stays off by default in non-interactive mode (`claude -p`) and in the Agent SDK. On Claude Code older than v2.1.232, or to turn it on where it is off by default, set `CLAUDE_CODE_FORK_SUBAGENT=1`.

> **Fork-mode subagents run in the background.** Where fork mode is on — as it is by default in an interactive session — Claude Code runs the subagent in the background, forked and non-forked subagents alike.

### Configuration

```yaml
---
name: alternative-explorer
description: Explore an alternative implementation path while preserving parent context
context: fork
tools: Read, Edit, Bash, Grep, Glob
---

You are a forked subagent. You inherit the parent's full conversation and
may explore an alternative approach. Return your findings and the parent
will decide whether to adopt them.
```

### Enabling Fork Mode Explicitly

Interactive sessions on v2.1.232+ need no flag. Use this on older versions, in headless
runs, or in the Agent SDK:

```bash
export CLAUDE_CODE_FORK_SUBAGENT=1
claude
```

### When to Use Fork vs Clean Context

| Scenario | `context: fork` | Clean context (default) |
|----------|-----------------|-------------------------|
| Explore alternative implementations | Yes | No (would lose context) |
| Long research with existing context | Yes | No |
| Independent specialized task | No | Yes |
| Avoiding context pollution | No | Yes |

---

## Restrict Spawnable Subagents

You can control which subagents a given subagent is allowed to spawn by using the `Agent(agent_type)` syntax in the `tools` field. This provides a way to allowlist specific subagents for delegation.

> **Note**: In v2.1.63, the `Task` tool was renamed to `Agent`. Existing `Task(...)` references still work as aliases.

### Example

```yaml
---
name: coordinator
description: Coordinates work between specialized agents
tools: Agent(worker, researcher), Read, Bash
---

You are a coordinator agent. You can delegate work to the "worker" and
"researcher" subagents only. Use Read and Bash for your own exploration.
```

In this example, the `coordinator` subagent can only spawn the `worker` and `researcher` subagents. It cannot spawn any other subagents, even if they are defined elsewhere.

---
