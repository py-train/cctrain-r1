## Settings File Hierarchy

Claude Code settings (including `autoMemoryDirectory`, `claudeMdExcludes`, and other configuration) resolve by precedence — unlike CLAUDE.md files above, settings genuinely override rather than concatenate. When the same setting appears in multiple scopes, the higher level wins:

| Level | Location | Scope |
|-------|----------|-------|
| 1 (Highest) | Managed — `managed-settings.json`, plist/registry, or server-managed | Organization-wide enforcement; cannot be overridden |
| 2 | Command line arguments | Temporary session overrides |
| 3 | `.claude/settings.local.json` | Local overrides (git-ignored) |
| 4 | `.claude/settings.json` | Project-level (committed to git) |
| 5 (Lowest) | `~/.claude/settings.json` | User preferences |

Managed settings also support a drop-in directory, `managed-settings.d/`, alongside `managed-settings.json`: the base file merges first, then `*.json` files in the drop-in directory merge on top in alphabetical order (scalars override, arrays concatenate and de-duplicate, objects deep-merge). This lets separate teams deploy independent policy fragments without editing a shared file. Note this is a **settings.json** mechanism, not a CLAUDE.md one — it doesn't apply to the CLAUDE.md file locations described above.

Permission rules (`allow`/`ask`/`deny`) behave differently from other settings: they merge across scopes instead of the higher level replacing the lower one.
