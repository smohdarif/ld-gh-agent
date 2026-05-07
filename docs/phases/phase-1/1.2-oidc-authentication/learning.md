# Learning — OIDC Authentication

## Key Concepts

### Workload Identity Federation

A pattern where short-lived OIDC tokens are exchanged for 3rd party access tokens, eliminating the need for long-lived secrets (like API keys or PATs). GitHub is the identity provider (IdP), and LaunchDarkly is the relying party.

This is the same pattern used by:
- GitHub Actions OIDC → AWS, Azure, GCP
- GitHub Copilot Extensions → 3rd party MCP servers

### RFC 7523 — JWT Bearer Assertion

The standard protocol for exchanging a signed JWT for an access token. The client sends the JWT as an "assertion" in the token exchange request, and the server verifies the signature and returns an access token.

### RFC 7009 — Token Revocation

The standard protocol for explicitly revoking an access token. GitHub calls this when the agent job completes to minimize the token's active lifetime.

### JWT Claims

The signed JWT contains identity information:
- **`sub`** (subject) — Who is making the request. Determines authorization granularity.
- **`iss`** (issuer) — Who signed the token (GitHub).
- **`aud`** (audience) — Who the token is intended for (LaunchDarkly).
- **`exp`** (expiration) — When the token expires (5 minutes).

### User-on-Behalf-of Flow

When a user triggers the agent (e.g., by @-mentioning it in a PR), the JWT includes their GitHub identity. LaunchDarkly can map this to a LaunchDarkly account and scope the access token to that user's permissions.

### Account Linking

A one-time process where the GitHub user authorizes the connection between their GitHub identity and their LaunchDarkly account. After linking, future OIDC token exchanges can automatically return user-scoped access tokens.

### API Key vs SDK Key vs OIDC Token

| Credential | Purpose | Used Here? |
|---|---|---|
| **OIDC Token** | Exchanged for an access token to manage flags via MCP | Yes (primary) |
| **API Key (`LD_API_KEY`)** | Static credential for flag management API | Yes (fallback) |
| **SDK Key** | Used by application code at runtime to evaluate flags | No |

## Security Properties

- **Short-lived**: JWT expires in 5 minutes, access token is revoked on job completion
- **No secrets at rest**: Nothing stored in the repo or environment
- **Scoped**: Access token permissions determined by LaunchDarkly based on user identity
- **Auditable**: Each token exchange is tied to a specific user, agent, repo, and org
