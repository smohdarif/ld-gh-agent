# HLD — Create Flag Skill

## Overview

The `create-flag` skill is a focused task definition that creates a new feature flag in LaunchDarkly. It wraps the `create_feature_flag` MCP tool with a structured workflow: create, verify, and return a formatted result.

## Flow

```
Agent invokes create-flag skill
        │
        ▼
  Call create_feature_flag (MCP)
        │
        ▼
  Call get_feature_flag (MCP) — verify
        │
        ▼
  Return formatted result:
    "Created flag: **{name}** (`{key}`)
     Project: {project_key}
     URL: https://app.launchdarkly.com/..."
```

## Parameters

| Parameter | Required | Default | Description |
|---|---|---|---|
| `project_key` | Yes | — | LaunchDarkly project key |
| `flag_key` | Yes | — | Unique flag key (lowercase, hyphens only) |
| `flag_name` | Yes | — | Human-readable name |
| `description` | Yes | — | What the flag gates |
| `tags` | No | — | Comma-separated tags |
| `temporary` | No | `true` | Short-lived (`true`) or permanent (`false`) |
| `flag_type` | No | `boolean` | `boolean`, `string`, `number`, or `json` |

## What It Creates

- A flag definition in the **off** state
- Default variations (`true`/`false` for boolean)
- No targeting rules, rollouts, or segments
