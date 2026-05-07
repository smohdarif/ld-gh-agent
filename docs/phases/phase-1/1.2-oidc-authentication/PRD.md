# PRD — OIDC Authentication

## Problem Statement

Background agentic workloads (like this plugin) need to authenticate with 3rd party services (LaunchDarkly) without user interaction. Traditional OAuth flows require a browser redirect, which isn't possible in an automated agent context. Static API keys introduce secret management overhead and security risks.

## Goals

1. Authenticate with LaunchDarkly without storing long-lived secrets
2. Scope access tokens to the user who triggered the agent
3. Automatically revoke tokens when the agent job completes
4. Fall back to API key for local development

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | OIDC token exchange must work without user interaction | Must |
| R2 | JWTs must be short-lived (5-minute expiry) | Must |
| R3 | Access tokens must be revoked when the job completes | Must |
| R4 | Token must contain identity claims (user, repo, org) | Must |
| R5 | LaunchDarkly must be able to scope permissions based on the user's identity | Should |
| R6 | Fallback to `LD_API_KEY` env var when OIDC is unavailable | Must |
| R7 | Pre-run validation must block agent if no auth is available | Must |

## Security Considerations

- No secrets stored in the repository
- Tokens are short-lived and auto-revoked
- GitHub is the identity provider — trusted by both sides
- LaunchDarkly controls what permissions the access token grants
