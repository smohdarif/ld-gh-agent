# Pseudo Logic — OIDC Authentication

## Full OIDC Token Exchange Flow

```
WHEN agent starts and oidc: true is configured:

  1. GITHUB reads OIDC config for the MCP server
     - Discover endpoints from MCP server URL:
       a. Check {url}/.well-known/oauth-authorization-server
       b. Check {url}/.well-known/oauth-protected-resource
       c. Check {url}/.well-known/openid-configuration
     - Extract token exchange endpoint and audience

  2. GITHUB signs a JWT:
     - Algorithm: RS256
     - Expiry: 5 minutes
     - Claims:
       sub    = user_id:{github_user_id}   (or installation_id if no user)
       iss    = "https://github.com"
       aud    = {discovered audience or MCP server URL}
       exp    = now() + 5 minutes
       preferred_username = {github_login}
       user_id = {github_user_id}

  3. GITHUB sends token exchange request:
     POST {token_endpoint}
     grant_type = urn:ietf:params:oauth:grant-type:jwt-bearer
     assertion  = {signed JWT}

  4. LAUNCHDARKLY token endpoint:
     a. FETCH GitHub's public keys from /.well-known/jwks.json
     b. VERIFY JWT signature using public keys
     c. VALIDATE claims (iss, aud, exp)
     d. LOOKUP user mapping from sub/preferred_username/user_id
     e. DETERMINE permissions for this user
     f. GENERATE access_token scoped to user's LaunchDarkly permissions
     g. RETURN { access_token, token_type: "Bearer", expires_in }

  5. GITHUB injects the access token:
     - For remote servers: Authorization: Bearer {access_token} header
     - For local servers: environment variable

  6. AGENT executes using MCP tools with the injected token

  7. WHEN agent job completes:
     a. GITHUB signs a fresh JWT for revocation
     b. GITHUB calls revocation endpoint:
        POST {revoke_endpoint}
        token = {access_token}
     c. LAUNCHDARKLY invalidates the access token
```

## Pre-Run Validation Logic

```
FUNCTION validate_ld_context():
  IF env GITHUB_COPILOT_OIDC_MCP_TOKEN is set and non-empty:
    RETURN success (exit 0)
  ELSE IF env LD_API_KEY is set and non-empty:
    RETURN success (exit 0)
  ELSE:
    PRINT "ERROR: No LaunchDarkly authentication token available." to stderr
    RETURN failure (exit 1)
    → Agent run is aborted by the platform
```

## Fallback: API Key Flow

```
WHEN oidc: true but OIDC token exchange fails:
  IF LD_API_KEY is set:
    USE LD_API_KEY as Authorization: Bearer header
  ELSE:
    ABORT agent run
```
