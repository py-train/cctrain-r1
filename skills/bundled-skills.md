## Bundled Skills

Claude Code ships with a set of built-in skills that are always available without installation (the most useful ones are listed below; see the [commands reference](https://code.claude.com/docs/en/commands) for the full set):

| Skill | Description |
|-------|-------------|
| `/batch <instruction>` | Orchestrate large-scale parallel changes across codebase using git worktrees |
| `/claude-api` | Load Claude API/SDK reference; auto-activates on `anthropic`/`@anthropic-ai/sdk` imports |
| `/dataviz` | Chart and dashboard design guidance with a runnable color-palette validator (v2.1.198) |
| `/debug [description]` | Troubleshoot current session by reading debug log |
| `/deep-research <topic>` | Run an in-depth research pass on a topic (explicit invocation only since v2.1.218 — Claude won't trigger this on its own) |
| `/fewer-permission-prompts` | Scan transcripts and propose a prioritized allowlist for common read-only tools |
| `/loop [interval] <prompt>` | Run prompt repeatedly on interval (e.g., `/loop 5m check the deploy`) |
| `/run` *(v2.1.145+)* | Launch this project's app to see a change running — looks for a project skill, otherwise falls back to built-in patterns per project type |
| `/run-skill-generator` *(v2.1.145+)* | Teach `/run`/`/verify` how to handle a specific project by generating a per-project skill |
| `/code-review [effort]` | Review the current diff for correctness bugs at a chosen effort level (e.g. `/code-review high`); pass `--comment` to post findings as inline PR comments. A distinct skill from `/simplify` (quality/reuse cleanups), which was split back out in v2.1.154. (explicit invocation only since v2.1.215 — Claude won't trigger this on its own) Since v2.1.218 it runs as a background subagent, so review work no longer fills your conversation and stacked slash commands stay its review target. |
| `/simplify` | Cleanup-only review — reuse, simplification, efficiency, altitude — and applies the fixes. Split back out from `/code-review` in v2.1.154 |
| `/verify` *(v2.1.145+)* | Build, run, and observe the app to confirm a fix works (not just that tests pass) (explicit invocation only since v2.1.215 — Claude won't trigger this on its own) |

These skills are available out-of-the-box and do not need to be installed or configured. They follow the same SKILL.md format as custom skills.
