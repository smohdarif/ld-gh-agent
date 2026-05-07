# DLD — Flag Creator Sub-Agent

## Agent Definition

File: `agents/flag-creator.agent.md`

### Front Matter

```yaml
name: flag-creator
description: Specialized sub-agent that handles the structured workflow for creating a single LaunchDarkly feature flag
disable-model-invocation: false
tools: []
mcp-servers:
  launchdarkly:
    type: http
    url: https://mcp.launchdarkly.com/mcp/fm
    tools: ["*"]
    oidc: true
```

| Field | Value | Purpose |
|---|---|---|
| `name` | `flag-creator` | Sub-agent identifier |
| `tools` | `[]` | No file system tools needed — only MCP tools |
| `mcp-servers` | LaunchDarkly MCP | Same MCP server as main agent |

### Input Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_key` | string | Yes | LaunchDarkly project to create the flag in |
| `flag_key` | string | Yes | Unique flag key (lowercase, hyphen-separated) |
| `flag_name` | string | Yes | Human-readable display name |
| `description` | string | Yes | What the flag gates |
| `tags` | list | No | Tags to apply |
| `flag_type` | string | No | `boolean` (default), `string`, `number`, `json` |
| `temporary` | boolean | No | `true` for experiments, `false` for kill switches |

### Output Structure

```json
{
  "success": true,
  "flag_key": "new-checkout-flow",
  "flag_url": "https://app.launchdarkly.com/projects/default/flags/new-checkout-flow",
  "environments": ["production", "staging", "development"],
  "error": null
}
```

### Error Cases

| Scenario | Response |
|---|---|
| Flag already exists | `success: false`, include existing flag URL |
| Invalid project key | `success: false`, clear error message |
| API failure | `success: false`, error message from API |

### MCP Tools Used

| Tool | When |
|---|---|
| `create_feature_flag` | To create the flag with provided parameters |
| `get_feature_flag` | To verify the flag was created successfully |
