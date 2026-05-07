# Test Cases — Skills Design Rationale

## TC-3.0.1: All skills are discoverable

| Field | Value |
|---|---|
| **Precondition** | Plugin is loaded |
| **Action** | Platform scans `skills/` directory |
| **Expected** | Three skills discovered: `create-flag`, `list-flags`, `suggest-flags` |

## TC-3.0.2: Skill can be invoked by main agent

| Field | Value |
|---|---|
| **Precondition** | Main agent is running |
| **Action** | Main agent decides to use `list-flags` skill |
| **Expected** | Skill instructions are loaded, agent follows them, output matches skill's format |

## TC-3.0.3: Skill uses invoking agent's MCP tools

| Field | Value |
|---|---|
| **Precondition** | Main agent invokes `create-flag` skill |
| **Action** | Skill instructs to call `create_feature_flag` MCP tool |
| **Expected** | The main agent's MCP connection is used (not a separate one) |

## TC-3.0.4: Skill output follows defined format

| Field | Value |
|---|---|
| **Precondition** | `list-flags` skill is invoked with project_key="default" |
| **Action** | Skill completes |
| **Expected** | Output matches: "Existing flags in project 'default':\n- `key` — Name (created date)" |

## TC-3.0.5: Sub-agent and skill produce equivalent results

| Field | Value |
|---|---|
| **Precondition** | Same flag creation parameters |
| **Action** | Create flag via (a) create-flag skill directly, (b) flag-creator sub-agent |
| **Expected** | Same flag is created in LaunchDarkly in both cases |

## TC-3.0.6: Skill with no MCP tools (suggest-flags)

| Field | Value |
|---|---|
| **Precondition** | Main agent invokes `suggest-flags` with a PR diff |
| **Action** | Skill analyzes the diff (LLM only, no MCP calls) |
| **Expected** | Returns structured flag recommendations without calling any external API |

## TC-3.0.7: Hypothetical second agent can reuse skills

| Field | Value |
|---|---|
| **Precondition** | A second agent is added to the plugin |
| **Action** | New agent invokes `list-flags` skill |
| **Expected** | Skill works identically, instructions loaded into new agent's context |
