# DLD — List Flags Skill

## Skill Definition

File: `skills/list-flags/SKILL.md`

### Front Matter

```yaml
name: list-flags
description: List existing LaunchDarkly feature flags in a project
disable-model-invocation: false
```

### MCP Tools Used

| Tool | Parameters | Purpose |
|---|---|---|
| `list_feature_flags` | project_key, search, tag, limit | Query existing flags |

### Output Format

**Flags found:**
```
Existing flags in project 'default':
- `new-checkout-flow` — New Checkout Flow (created 2026-01-15)
- `dark-mode-toggle` — Dark Mode Toggle (created 2026-02-01)
```

**No flags found:**
```
No existing flags found matching your criteria.
```
