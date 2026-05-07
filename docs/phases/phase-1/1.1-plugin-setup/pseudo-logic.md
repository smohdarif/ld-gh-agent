# Pseudo Logic — Plugin Setup & Configuration

## Plugin Initialization Flow

```
WHEN GitHub Copilot Extensions platform loads the plugin:
  1. READ plugin.json
  2. REGISTER plugin with name "launchdarkly-flag-automation"
  3. DISCOVER agents from "agents/" directory
     - Load main.agent.md → register as "launchdarkly-agent"
     - Load flag-creator.agent.md → register as "flag-creator"
  4. DISCOVER skills from "skills/" directory
     - Load create-flag/SKILL.md → register as "create-flag"
     - Load list-flags/SKILL.md → register as "list-flags"
     - Load suggest-flags/SKILL.md → register as "suggest-flags"
  5. LOAD hooks from hooks.json
     - Register "before:agent:run" → scripts/validate-ld-context.sh
```

## Pre-Run Hook Flow

```
WHEN any agent is about to run:
  1. TRIGGER "before:agent:run" event
  2. EXECUTE scripts/validate-ld-context.sh
  3. IF $GITHUB_COPILOT_OIDC_MCP_TOKEN is set:
       → PASS (exit 0)
     ELSE IF $LD_API_KEY is set:
       → PASS (exit 0)
     ELSE:
       → PRINT error to stderr
       → FAIL (exit 1) — agent run is aborted
```

## MCP Server Connection (per agent)

```
WHEN an agent starts and has mcp-servers declared:
  1. READ mcp-servers config from agent front matter
  2. FOR each MCP server:
     a. IF oidc: true
        → Use OIDC token exchange to get access token
        → Inject token as Authorization: Bearer header
     b. ELSE IF headers are configured
        → Use static headers (e.g., API key)
  3. CONNECT to MCP server URL via HTTP
  4. DISCOVER available tools from MCP server
  5. MAKE tools available to the agent
```
