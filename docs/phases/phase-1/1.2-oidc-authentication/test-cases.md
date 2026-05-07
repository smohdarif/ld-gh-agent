# Test Cases — OIDC Authentication

## TC-1.2.1: OIDC token exchange succeeds

| Field | Value |
|---|---|
| **Precondition** | Agent is running in GitHub Copilot Extensions with OIDC configured |
| **Action** | Agent starts, GitHub performs token exchange with LaunchDarkly |
| **Expected** | Access token is returned and injected as Authorization header |

## TC-1.2.2: JWT contains correct claims

| Field | Value |
|---|---|
| **Precondition** | User triggers agent via @-mention in a PR |
| **Action** | GitHub signs JWT for token exchange |
| **Expected** | JWT contains `sub`, `preferred_username`, `user_id`, `iss`, `aud`, `exp` claims |

## TC-1.2.3: JWT expires after 5 minutes

| Field | Value |
|---|---|
| **Precondition** | JWT is signed by GitHub |
| **Action** | Check `exp` claim |
| **Expected** | `exp` is approximately 5 minutes from `iat` |

## TC-1.2.4: Token is revoked after job completes

| Field | Value |
|---|---|
| **Precondition** | Agent has completed its work |
| **Action** | GitHub calls LaunchDarkly's revocation endpoint |
| **Expected** | Access token is invalidated, subsequent use returns 401 |

## TC-1.2.5: Invalid JWT is rejected

| Field | Value |
|---|---|
| **Precondition** | JWT signature is invalid or expired |
| **Action** | GitHub sends JWT to LaunchDarkly token endpoint |
| **Expected** | Token exchange fails, authorization error is returned |

## TC-1.2.6: Fallback to API key when OIDC unavailable

| Field | Value |
|---|---|
| **Precondition** | OIDC token not available, `LD_API_KEY` is set |
| **Action** | Agent starts |
| **Expected** | Agent uses `LD_API_KEY` for authentication, MCP tools work |

## TC-1.2.7: Agent aborts when no auth available

| Field | Value |
|---|---|
| **Precondition** | Neither OIDC token nor `LD_API_KEY` is set |
| **Action** | Pre-run hook executes |
| **Expected** | `validate-ld-context.sh` exits 1, agent run is aborted |

## TC-1.2.8: User identity is mapped to LaunchDarkly permissions

| Field | Value |
|---|---|
| **Precondition** | User's GitHub account is linked to LaunchDarkly account |
| **Action** | OIDC token exchange completes |
| **Expected** | Access token is scoped to the user's LaunchDarkly permissions (e.g., can only access projects the user has access to) |
