## Security Considerations

### Disclaimer

**USE AT YOUR OWN RISK**: Hooks execute arbitrary shell commands. You are solely responsible for:
- Commands you configure
- File access/modification permissions
- Potential data loss or system damage
- Testing hooks in safe environments before production use

### Security Notes

- **Workspace trust required:** The `statusLine` and `fileSuggestion` hook output commands now require workspace trust acceptance before they take effect.
- **Status-line terminal size (v2.1.153):** Status-line command scripts now receive `COLUMNS` and `LINES` environment variables, so a script can adapt its output to the terminal width/height (e.g. `[ "$COLUMNS" -lt 80 ] && short_output`).
- **HTTP hooks and environment variables:** HTTP hooks require an explicit `allowedEnvVars` list to use environment variable interpolation in URLs. This prevents accidental leakage of sensitive environment variables to remote endpoints.
- **Managed settings hierarchy:** The `disableAllHooks` setting now respects the managed settings hierarchy, meaning organization-level settings can enforce hook disablement that individual users cannot override.
- **PowerShell auto-approve (v2.1.119):** PowerShell tool commands can be auto-approved in permission mode, matching Bash. This brings parity for Windows users running Claude Code with PowerShell-backed shell tools.
- **Bash bare env-var auto-approve closed (v2.1.145):** Before v2.1.145, a Bash command of the form `FOO=bar somecommand` (a bare variable assignment inline with a non-allowlisted command) could be auto-approved when only `FOO=bar` by itself was on the allowlist. v2.1.145 closes this — such commands now hit the permission prompt. Scripts relying on the implicit allow will start prompting; re-allow them explicitly via a `Bash(...)` permission rule that covers the full command, not just the variable assignment.

### Best Practices

| Do | Don't |
|-----|-------|
| Validate and sanitize all inputs | Trust input data blindly |
| Quote shell variables: `"$VAR"` | Use unquoted: `$VAR` |
| Block path traversal (`..`) | Allow arbitrary paths |
| Use absolute paths with `$CLAUDE_PROJECT_DIR` | Hardcode paths |
| Skip sensitive files (`.env`, `.git/`, keys) | Process all files |
| Test hooks in isolation first | Deploy untested hooks |
| Use explicit `allowedEnvVars` for HTTP hooks | Expose all env vars to webhooks |
