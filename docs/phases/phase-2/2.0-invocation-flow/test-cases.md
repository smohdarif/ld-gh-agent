# Test Cases — Invocation Flow

## TC-2.0.1: @-mention triggers main agent

| Field | Value |
|---|---|
| **Precondition** | Plugin is installed, OIDC is configured |
| **Action** | Developer types `@launchdarkly-agent` in a PR comment |
| **Expected** | Platform detects @-mention, runs pre-hook, starts main agent with PR context |

## TC-2.0.2: Pre-hook aborts when no auth

| Field | Value |
|---|---|
| **Precondition** | Neither OIDC token nor API key is available |
| **Action** | Any event triggers the agent |
| **Expected** | `validate-ld-context.sh` exits 1, agent does not start |

## TC-2.0.3: OIDC token exchange completes before agent starts

| Field | Value |
|---|---|
| **Precondition** | OIDC is configured, pre-hook passes |
| **Action** | Platform starts the main agent |
| **Expected** | Access token is obtained and injected before the agent makes any MCP tool calls |

## TC-2.0.4: PR context is injected into main agent

| Field | Value |
|---|---|
| **Precondition** | Agent triggered from PR #42 |
| **Action** | Platform starts main agent |
| **Expected** | Agent has access to PR diff, title, description, branch, and comments |

## TC-2.0.5: Main agent can invoke flag-creator sub-agent

| Field | Value |
|---|---|
| **Precondition** | Main agent has identified a flag to create |
| **Action** | Main agent decides to call flag-creator |
| **Expected** | Platform routes the call, starts flag-creator with provided params, returns result |

## TC-2.0.6: Flag-creator result flows back to main agent

| Field | Value |
|---|---|
| **Precondition** | Flag-creator has completed (success or failure) |
| **Action** | Platform returns result to main agent |
| **Expected** | Main agent receives structured result and uses it to compose summary comment |

## TC-2.0.7: Summary comment is posted on PR

| Field | Value |
|---|---|
| **Precondition** | Main agent has composed a summary comment |
| **Action** | Main agent outputs the comment |
| **Expected** | Platform posts the comment on the originating PR/issue |

## TC-2.0.8: Token is revoked after job completion

| Field | Value |
|---|---|
| **Precondition** | Agent has finished all work |
| **Action** | Platform performs cleanup |
| **Expected** | OIDC access token is revoked via LaunchDarkly's revocation endpoint |

## TC-2.0.9: Issue assignment triggers main agent

| Field | Value |
|---|---|
| **Precondition** | Plugin is installed |
| **Action** | Developer assigns the agent to an issue |
| **Expected** | Platform detects assignment, starts main agent with issue context |

## TC-2.0.10: Full end-to-end flow

| Field | Value |
|---|---|
| **Precondition** | Plugin installed, OIDC configured, PR with new feature code |
| **Action** | Developer @-mentions agent in PR |
| **Expected** | Agent analyzes diff → checks duplicates → creates flag → posts comment with flag URL and next steps → token revoked |
