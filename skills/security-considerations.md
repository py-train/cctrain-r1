## Security Considerations

**Only use Skills from trusted sources.** Skills provide Claude with capabilities through instructions and code—a malicious Skill can direct Claude to invoke tools or execute code in harmful ways.

**Key security considerations:**

- **Audit thoroughly**: Review all files in the Skill directory
- **External sources are risky**: Skills that fetch from external URLs can be compromised
- **Tool misuse**: Malicious Skills can invoke tools in harmful ways
- **Treat like installing software**: Only use Skills from trusted sources

### Disabling shell substitution in skills

Skills support the `` !`command` `` syntax to inject the output of shell commands into the prompt before Claude sees it. In security-sensitive environments (shared enterprise deployments, locked-down CI runners) you can disable this substitution entirely via the `disableSkillShellExecution` setting (added in **v2.1.91**):

```jsonc
// ~/.claude/settings.json or managed policy
{
  "disableSkillShellExecution": true
}
```

When `disableSkillShellExecution` is `true`, any `` !`command` `` markers in a skill are left as literal text instead of being executed — removing the skill-level shell-injection attack surface without disabling skills themselves. Consider combining this with an `allowedTools` allowlist for defense in depth.

### Hiding bundled skills (`disableBundledSkills`)

The `disableBundledSkills` setting (added in **v2.1.169**) hides the bundled skills, workflows, and commands that ship with Claude Code from the model. Use it when the built-in skills are noise for a given project, or to reduce the model's skill surface:

```jsonc
// ~/.claude/settings.json or project .claude/settings.json
{
  "disableBundledSkills": true
}
```

The equivalent environment-variable form is:

```bash
export CLAUDE_CODE_DISABLE_BUNDLED_SKILLS=1
```
