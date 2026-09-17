### Restricting Claude's Skill Access

Three ways to control which skills Claude can invoke:

**Disable all skills** in `/permissions`:
```
# Add to deny rules:
Skill
```

**Allow or deny specific skills**:
```
# Allow only specific skills
Skill(commit)
Skill(review-pr *)

# Deny specific skills
Skill(deploy *)
```

**Hide individual skills** by adding `disable-model-invocation: true` to their frontmatter.

### Controlling Skill Override Behavior (`skillOverrides`)

When a project skill and a user skill share the same name, project wins by default. The `skillOverrides` setting (v2.1.129+) lets you tune this. Add it to `~/.claude/settings.json` or project `.claude/settings.json`:

```json
{
  "skillOverrides": "name-only"
}
```

Accepted values:

| Value | Behavior |
|-------|----------|
| `"on"` (default) | A repo skill can override a user skill of the same name. |
| `"off"` | Disable overriding entirely — user skills always win. |
| `"name-only"` | Match overrides only on skill name (ignore description / source). |
| `"user-invocable-only"` | Only user-invocable skills can be overridden — model-invoked skills always come from their original location. |

Useful when team policy says "user-defined skills must always take precedence" (`"off"`) or "only allow narrow name-based overrides" (`"name-only"`).

