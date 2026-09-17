## Excluding CLAUDE.md Files with `claudeMdExcludes`

In large monorepos, some CLAUDE.md files may be irrelevant to your current work. The `claudeMdExcludes` setting lets you skip specific CLAUDE.md files so they are not loaded into context:

```jsonc
// In ~/.claude/settings.json or .claude/settings.json
{
  "claudeMdExcludes": [
    "packages/legacy-app/CLAUDE.md",
    "vendors/**/CLAUDE.md"
  ]
}
```

Patterns are matched against paths relative to the project root. This is particularly useful for:

- Monorepos with many sub-projects, where only some are relevant
- Repositories that contain vendored or third-party CLAUDE.md files
- Reducing noise in Claude's context window by excluding stale or unrelated instructions
