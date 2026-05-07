# Test Cases — Plugin Setup & Configuration

## TC-1.1.1: Plugin manifest is valid

| Field | Value |
|---|---|
| **Precondition** | `plugin.json` exists in repo root |
| **Action** | Platform reads and parses `plugin.json` |
| **Expected** | Plugin is registered with name `launchdarkly-flag-automation` |

## TC-1.1.2: Agent discovery

| Field | Value |
|---|---|
| **Precondition** | `agents/` directory contains `.agent.md` files |
| **Action** | Platform scans `agents/` directory |
| **Expected** | Both `launchdarkly-agent` and `flag-creator` agents are discovered and registered |

## TC-1.1.3: Skill discovery

| Field | Value |
|---|---|
| **Precondition** | `skills/` directory contains subdirectories with `SKILL.md` |
| **Action** | Platform scans `skills/` directory |
| **Expected** | Three skills registered: `create-flag`, `list-flags`, `suggest-flags` |

## TC-1.1.4: Pre-run hook — OIDC token present

| Field | Value |
|---|---|
| **Precondition** | `GITHUB_COPILOT_OIDC_MCP_TOKEN` is set |
| **Action** | `validate-ld-context.sh` runs before agent |
| **Expected** | Script exits 0, agent proceeds |

## TC-1.1.5: Pre-run hook — API key present

| Field | Value |
|---|---|
| **Precondition** | `LD_API_KEY` is set, OIDC token is not |
| **Action** | `validate-ld-context.sh` runs before agent |
| **Expected** | Script exits 0, agent proceeds |

## TC-1.1.6: Pre-run hook — no auth available

| Field | Value |
|---|---|
| **Precondition** | Neither `GITHUB_COPILOT_OIDC_MCP_TOKEN` nor `LD_API_KEY` is set |
| **Action** | `validate-ld-context.sh` runs before agent |
| **Expected** | Script exits 1 with error message, agent run is aborted |

## TC-1.1.7: Hooks file references valid script

| Field | Value |
|---|---|
| **Precondition** | `hooks.json` references `scripts/validate-ld-context.sh` |
| **Action** | Platform reads `hooks.json` |
| **Expected** | Hook is registered for `before:agent:run` event, script path is valid and executable |
