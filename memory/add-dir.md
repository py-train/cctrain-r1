## Additional Directories with `--add-dir`

The `--add-dir` flag allows Claude Code to load CLAUDE.md files from additional directories beyond the current working directory. This is useful for monorepos or multi-project setups where context from other directories is relevant.

To enable this feature, set the environment variable:

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1
```

Then launch Claude Code with the flag:

```bash
claude --add-dir /path/to/other/project
```

Claude will load CLAUDE.md from the specified additional directory alongside the memory files from your current working directory.
