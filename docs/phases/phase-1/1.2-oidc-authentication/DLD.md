# DLD — OIDC Authentication

## Detailed Design

### Agent Front Matter Configuration

Both agents declare the OIDC-enabled MCP server:

```yaml
mcp-servers:
  launchdarkly:
    type: http
    url: https://mcp.launchdarkly.com/mcp/fm
    tools: ["*"]
    oidc: true
```

| Field | Value | Purpose |
|---|---|---|
| `type` | `http` | Remote MCP server over HTTP |
| `url` | `https://mcp.launchdarkly.com/mcp/fm` | LaunchDarkly's hosted MCP endpoint |
| `tools` | `["*"]` | Grant access to all tools exposed by the server |
| `oidc` | `true` | Enable OIDC token exchange (GitHub handles automatically) |

### JWT Claims (Signed by GitHub)

| Claim | Example | Description |
|---|---|---|
| `sub` | `user_id:213455` | Subject — identifies who/what is making the request |
| `preferred_username` | `octocat` | GitHub login of the triggering user |
| `user_id` | `213455` | GitHub user ID |
| `iss` | `https://github.com` | Issuer — always GitHub |
| `aud` | `https://mcp.launchdarkly.com` | Audience — the MCP server |
| `exp` | `1714000000` | Expiration — 5 minutes from issuance |

### Subject Configuration Variants

| Scenario | `sub` claim format |
|---|---|
| User-triggered, `agent-only-subject: false` | `user_id:{user_id}` |
| Agent-only or no user present | `installation_id:{installation_id}` |
| GitHub built-in agents | `app:{client_id}:owner_id:org_id:{org_id}` |

### Token Exchange Request (RFC 7523)

```
POST /token HTTP/1.1
Host: mcp.launchdarkly.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
assertion=<signed OIDC JWT>
requested_token_type=urn:ietf:params:oauth:token-type:access_token
audience=<configured audience>
```

### Token Exchange Response

```json
{
  "access_token": "ld_mcp_abc123...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Token Revocation Request (RFC 7009)

```
POST /revoke HTTP/1.1
Host: mcp.launchdarkly.com
Content-Type: application/x-www-form-urlencoded

token=ld_mcp_abc123...
```

### Pre-Run Validation Script

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ -z "${GITHUB_COPILOT_OIDC_MCP_TOKEN:-}" ] && [ -z "${LD_API_KEY:-}" ]; then
  echo "ERROR: No LaunchDarkly authentication token available." >&2
  exit 1
fi

exit 0
```

### OIDC Discovery Flow (for remote servers)

When `oidc: true` is set, GitHub automatically:

1. Checks `{MCP_URL}/.well-known/oauth-authorization-server`
2. Checks `{MCP_URL}/.well-known/oauth-protected-resource`
3. Checks `{MCP_URL}/.well-known/openid-configuration`
4. Discovers the token exchange endpoint from the metadata
5. Determines the correct `audience` and `grant_type`
