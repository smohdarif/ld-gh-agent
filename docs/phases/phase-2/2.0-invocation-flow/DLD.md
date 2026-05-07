# DLD — Invocation Flow

## Detailed Step-by-Step

### Step 1: Event Detection

```
Developer types in PR #42 comment:
  "@launchdarkly-agent please create feature flags for this PR"

GitHub Copilot Extensions platform:
  - Parses comment for @-mention
  - Matches "launchdarkly-agent" to the registered plugin
  - Loads plugin.json to find agent and hook definitions
```

### Step 2: Pre-Run Hook Execution

```
Platform reads hooks.json:
  {
    "hooks": [
      {
        "event": "before:agent:run",
        "command": "scripts/validate-ld-context.sh"
      }
    ]
  }

Platform executes: bash scripts/validate-ld-context.sh

Script checks:
  IF $GITHUB_COPILOT_OIDC_MCP_TOKEN is set → exit 0 (pass)
  ELSE IF $LD_API_KEY is set → exit 0 (pass)
  ELSE → exit 1 (abort agent run)
```

### Step 3: OIDC Token Exchange

```
Platform reads main agent front matter:
  mcp-servers:
    launchdarkly:
      type: http
      url: https://mcp.launchdarkly.com/mcp/fm
      oidc: true

Platform performs token exchange:
  1. Signs JWT (RS256, 5min expiry) with claims:
     sub = user_id:{github_user_id}
     preferred_username = {github_login}
     iss = "https://github.com"
     aud = discovered from MCP server metadata

  2. POST to LaunchDarkly token endpoint:
     grant_type = urn:ietf:params:oauth:grant-type:jwt-bearer
     assertion = {signed JWT}

  3. Receives: { access_token: "ld_...", token_type: "Bearer" }

  4. Injects as Authorization: Bearer header for all MCP tool calls
```

### Step 4: Context Injection

```
Platform gathers context for PR #42:
  - PR title: "Add new checkout flow"
  - PR description: "Implements the new checkout experience..."
  - Branch: feature/new-checkout-flow
  - Diff: +150 lines across 4 files
  - Comments: the triggering @-mention
  - Labels: ["feature", "frontend"]

Platform makes available to agent:
  - Injected context (above)
  - Tools: ["view", "edit"] (read/modify repo files)
  - MCP tools: all LaunchDarkly tools (authenticated)
  - Sub-agents: ["flag-creator"] (available as callable tools)
```

### Step 5: Main Agent Execution

```
Main agent (LLM) receives system prompt from main.agent.md + context

LLM reasoning:
  1. "I've been @-mentioned in PR #42. Let me read the diff."
  2. Reads diff → sees new CheckoutV2 component with conditional rendering
  3. "This is a new user-facing feature. It should have a feature flag."
  4. Derives parameters:
     flag_key = "new-checkout-flow" (from branch name)
     flag_name = "Add new checkout flow" (from PR title)
     description = "Gates the new checkout experience"
     tags = ["repo:my-org/my-repo", "pr:42"]
     temporary = true (rollout flag)
  5. "I need to check for duplicates first."
  6. Invokes list-flags skill with project_key="default", search="new-checkout-flow"
  7. Result: no matching flags found
  8. "No duplicate. I'll create the flag."
  9. Invokes flag-creator sub-agent with structured params
```

### Step 6: Sub-Agent Execution

```
Platform starts flag-creator sub-agent:
  - System prompt from flag-creator.agent.md
  - Input: { project_key, flag_key, flag_name, description, tags, temporary }
  - MCP connection: same LaunchDarkly server (independently authenticated)

Sub-agent LLM reasoning:
  1. "I need to create flag 'new-checkout-flow' in project 'default'."
  2. Calls MCP tool: create_feature_flag
     → LaunchDarkly API creates the flag
     → Returns success
  3. "Let me verify it was created."
  4. Calls MCP tool: get_feature_flag
     → Returns flag details with environments: [production, staging, development]
  5. Returns to main agent:
     {
       success: true,
       flag_key: "new-checkout-flow",
       flag_url: "https://app.launchdarkly.com/projects/default/flags/new-checkout-flow",
       environments: ["production", "staging", "development"],
       error: null
     }
```

### Step 7: Report Back

```
Main agent receives result from flag-creator

LLM composes comment:
  "Created flag: **Add new checkout flow** (`new-checkout-flow`)
   Project: default
   Environments: production, staging, development
   URL: https://app.launchdarkly.com/projects/default/flags/new-checkout-flow

   Suggested next steps:
   - Wrap the CheckoutV2 component render (lines 42-67) in your flag evaluation
   - The flag is currently OFF in all environments
   - Turn it on in staging first to test"

Platform posts comment on PR #42
```

### Step 8: Cleanup

```
Platform:
  1. Signs fresh JWT for revocation
  2. POST to LaunchDarkly revocation endpoint:
     token = {access_token}
  3. LaunchDarkly invalidates the token
  4. Job marked complete
```
