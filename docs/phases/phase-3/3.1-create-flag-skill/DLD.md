# DLD — Create Flag Skill

## Skill Definition

File: `skills/create-flag/SKILL.md`

### Front Matter

```yaml
name: create-flag
description: Create a new LaunchDarkly feature flag in the specified project
disable-model-invocation: false
```

### MCP Tools Used

| Tool | Parameters | Purpose |
|---|---|---|
| `create_feature_flag` | project_key, key, name, description, tags, kind, temporary | Create the flag |
| `get_feature_flag` | project_key, flag_key | Verify the flag was created |

### Output Format

**Success:**
```
Created flag: **New Checkout Flow** (`new-checkout-flow`)
Project: default
URL: https://app.launchdarkly.com/projects/default/flags/new-checkout-flow
```

**Already exists:**
```
Flag `new-checkout-flow` already exists.
URL: https://app.launchdarkly.com/projects/default/flags/new-checkout-flow
```
