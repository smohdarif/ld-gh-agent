# DLD — Main Agent (Orchestrator)

## Agent Definition

File: `agents/main.agent.md`

### Front Matter

```yaml
name: launchdarkly-agent
description: Automatically creates and manages LaunchDarkly feature flags based on GitHub activity
disable-model-invocation: false
tools: ["view", "edit"]
mcp-servers:
  launchdarkly:
    type: http
    url: https://mcp.launchdarkly.com/mcp/fm
    tools: ["*"]
    oidc: true
```

### Configuration Details

| Field | Value | Purpose |
|---|---|---|
| `name` | `launchdarkly-agent` | Agent identifier for @-mentions and platform routing |
| `disable-model-invocation` | `false` | LLM is enabled — the agent can reason and generate responses |
| `tools` | `["view", "edit"]` | Agent can view files and edit them (for reading PR diffs) |
| `mcp-servers` | LaunchDarkly MCP | Connects to LaunchDarkly for flag operations |

### Flag Detection Patterns

The agent looks for these patterns in code and issues:

| Pattern | Example |
|---|---|
| Conditional logic for new behavior | `if (featureEnabled)` |
| Flag-related code comments | `TODO: gate this`, `feature flag` |
| Gradual rollout descriptions | Issue: "Roll out new checkout to 10% of users" |
| A/B test setups | `experiment`, `variant` |
| User-facing behavior changes | New components, API changes |

### Flag Naming Rules

| Field | Convention |
|---|---|
| **Key** | Lowercase, hyphen-separated. Derived from branch name or feature. Strip issue numbers and special chars. |
| **Name** | PR or issue title |
| **Tags** | `repo:{org}/{repo}`, `pr:{number}` or `issue:{number}` |
| **Description** | Short summary of what the flag gates |

### Skills Used

| Skill | When |
|---|---|
| `list-flags` | Before creating, to check for existing flags |
| `create-flag` | To create the actual flag (via flag-creator sub-agent) |
| `suggest-flags` | When analyzing a diff/issue for flag opportunities |

### Sub-Agent Delegation

When creating a flag, the main agent invokes the `flag-creator` sub-agent with:

```
project_key: <inferred or asked>
flag_key: <derived from context>
flag_name: <PR/issue title>
description: <summary>
tags: [repo:org/repo, pr:123]
flag_type: boolean (default)
temporary: true/false
```
