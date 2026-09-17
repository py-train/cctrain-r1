**Enhanced interactive mode:** Set `CLAUDE_CODE_NEW_INIT=1` to enable a multi-phase interactive flow that walks you through project setup step by step:

```bash
CLAUDE_CODE_NEW_INIT=1 claude
/init
```

`.claude/rules/*.md` files are a separate, related mechanism for topic- or path-scoped instructions.


Within the directory tree, Claude Code walks up from your working directory: `foo/CLAUDE.md` loads before `foo/bar/CLAUDE.md` if you launch from `foo/bar/`, so instructions closer to where you launched are read *last* — not "highest priority" in an override sense, just most recent in context. Within each directory, `CLAUDE.local.md` is appended after `CLAUDE.md`. CLAUDE.md and CLAUDE.local.md files in subdirectories *under* your working directory load on demand, when Claude reads files in those subdirectories, rather than at launch.

Organizations can also put managed CLAUDE.md content directly inside `managed-settings.json` via the `claudeMd` key, instead of deploying a separate file. This is honored only in managed/policy settings — setting `claudeMd` in user or project settings has no effect.

**`.claude/rules/*.md`** — modular, topic-specific instructions, optionally scoped to file paths via `paths` frontmatter. Rules without a `paths` field load unconditionally with the same priority as `.claude/CLAUDE.md`; path-scoped rules load on demand when Claude reads a matching file. User-level rules (`~/.claude/rules/`) load before project rules.


**Platform-specific configuration (v2.1.51+):**

Settings can also be configured via:
- **macOS**: Property list (plist) files
- **Windows**: Windows Registry

These platform-native mechanisms are read alongside JSON settings files and follow the same precedence rules.

> **Note (v2.1.119)**: `/config` changes now persist to `~/.claude/settings.json`. Values written via `/config` participate in the normal policy/local/project precedence chain described above — they are no longer session-only. Use `/config` for interactive edits and edit `settings.json` files directly for scripted or managed configuration.

### Retention and Cleanup Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `cleanupPeriodDays` | integer (days) | 30 | Retention window for on-disk artifacts. **As of v2.1.117**, it applies to all four of: checkpoints (`~/.claude/checkpoints/`), tasks (`~/.claude/tasks/`), shell-snapshots (`~/.claude/shell-snapshots/`), and backups (`~/.claude/backups/`). Files older than the window are pruned at startup. |

```jsonc
// ~/.claude/settings.json
{
  "cleanupPeriodDays": 14
}
```

### Attribution, Voice, and PR URL Settings

| Setting | Type | Description |
|---------|------|-------------|
| `attribution.commit` | boolean | Adds the `Co-Authored-By: Claude` trailer to commits Claude creates. Replaces the deprecated `includeCoAuthoredBy` flag. |
| `attribution.pr` | boolean | Adds Claude attribution to pull request descriptions. Replaces the deprecated `includeCoAuthoredBy` flag for PRs. |
| `attribution.sessionUrl` | boolean | Omit the claude.ai session link from commits and PRs created in web and Remote Control sessions (v2.1.183+). |
| `voice.enabled` | boolean | Enables push-to-talk voice dictation (`/voice`). Replaces the deprecated `voiceEnabled` flag. |
| `prUrlTemplate` | string | **New in v2.1.119.** Custom URL template for the footer PR badge; useful for GitLab, Bitbucket, or internal code-review platforms. Supports `{{owner}}`, `{{repo}}`, and `{{number}}` placeholders. |

```jsonc
// ~/.claude/settings.json
{
  "attribution": {
    "commit": false,
    "pr": true
  },
  "voice": {
    "enabled": true
  },
  "prUrlTemplate": "https://gitlab.internal/{{owner}}/{{repo}}/-/merge_requests/{{number}}"
}
```

#### Deprecated setting names

The following legacy setting keys still work but are deprecated. Prefer the replacements above.

| Deprecated key | Replacement | Notes |
|----------------|-------------|-------|
| `includeCoAuthoredBy` | `attribution.commit` / `attribution.pr` | The old single flag is split into separate commit and PR switches. Users on older installs can keep the legacy key; new projects should use the nested form. |
| `voiceEnabled` | `voice.enabled` | Grouped under the `voice` namespace alongside future voice-related options. |


### Version Requirement

Auto memory requires **Claude Code v2.1.59 or later**. If you are on an older version, upgrade first:

```bash
npm install -g @anthropic-ai/claude-code@latest
```

### Turning Auto Memory On or Off

Auto memory is **on by default**. The `autoMemoryEnabled` setting (default `true`) controls it; when `false`, Claude neither reads from nor writes to the auto memory directory. You can also toggle it with `/memory` during a session.

```json
{
  "autoMemoryEnabled": false
}
```

To disable it via environment instead, set `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. Setting it to `0` forces auto memory **on** even when `--bare` mode or `autoMemoryEnabled: false` would otherwise disable it.

### Custom Auto Memory Directory

By default, auto memory is stored in `~/.claude/projects/<project>/memory/`. You can change this location using the `autoMemoryDirectory` setting (available since **v2.1.74**):

```jsonc
// In ~/.claude/settings.json or .claude/settings.local.json (user/local settings only)
{
  "autoMemoryDirectory": "/path/to/custom/memory/directory"
}
```

> **Note**: `autoMemoryDirectory` can only be set in user-level (`~/.claude/settings.json`) or local settings (`.claude/settings.local.json`), not in project or managed policy settings.

This is useful when you want to:

- Store auto memory in a shared or synced location
- Separate auto memory from the default Claude configuration directory
- Use a project-specific path outside the default hierarchy

### Worktree and Repository Sharing

All worktrees and subdirectories within the same git repository share a single auto memory directory. This means switching between worktrees or working in different subdirectories of the same repo will read and write to the same memory files.

### Subagent Memory

Subagents (spawned via tools like Task or parallel execution) can have their own memory context. Use the `memory` frontmatter field in the subagent definition to specify which memory scopes to load:

```yaml
memory: user      # Load user-level memory only
memory: project   # Load project-level memory only
memory: local     # Load local memory only
```

This allows subagents to operate with focused context rather than inheriting the full memory hierarchy.

> **Note**: Subagents can also maintain their own auto memory. See the [official subagent memory documentation](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory) for details.

### Controlling Auto Memory

Auto memory can be controlled via the `CLAUDE_CODE_DISABLE_AUTO_MEMORY` environment variable:

| Value | Behavior |
|-------|----------|
| `0` | Force auto memory **on** |
| `1` | Force auto memory **off** |
| *(unset)* | Default behavior (auto memory enabled) |

```bash
# Disable auto memory for a session
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 claude

# Force auto memory on explicitly
CLAUDE_CODE_DISABLE_AUTO_MEMORY=0 claude
```
