## Modular Rules System

Create organized, path-specific rules using the `.claude/rules/` directory structure. Rules can be defined at both the project level and user level:

```
your-project/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       ├── security.md
│       └── api/                  # Subdirectories supported
│           ├── conventions.md
│           └── validation.md

~/.claude/
├── CLAUDE.md
└── rules/                        # User-level rules (all projects)
    ├── personal-style.md
    └── preferred-patterns.md
```

Rules are discovered recursively within the `rules/` directory, including any subdirectories. User-level rules at `~/.claude/rules/` are loaded before project-level rules, allowing personal defaults that projects can override.

### Path-Specific Rules with YAML Frontmatter

Define rules that apply only to specific file paths:

```markdown
---
paths: src/api/**/*.ts
---

# API Development Rules

- All API endpoints must include input validation
- Use Zod for schema validation
- Document all parameters and response types
- Include error handling for all operations
```

**Glob Pattern Examples:**

- `**/*.ts` - All TypeScript files
- `src/**/*` - All files under src/
- `src/**/*.{ts,tsx}` - Multiple extensions
- `{src,lib}/**/*.ts, tests/**/*.test.ts` - Multiple patterns

### Subdirectories and Symlinks

Rules in `.claude/rules/` support two organizational features:

- **Subdirectories**: Rules are discovered recursively, so you can organize them into topic-based folders (e.g., `rules/api/`, `rules/testing/`, `rules/security/`)
- **Symlinks**: Symlinks are supported for sharing rules across multiple projects. For example, you can symlink a shared rule file from a central location into each project's `.claude/rules/` directory
