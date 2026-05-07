# HLD — OIDC Authentication

## Overview

The plugin authenticates with LaunchDarkly's MCP server using GitHub's 3rd Party Token Support for MCPs — a workload identity federation pattern. No long-lived secrets are stored.

## Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌────────────────────────┐
│ Copilot (runtime) │     │ GitHub (sweagentd)│     │ LaunchDarkly MCP       │
│                  │     │                  │     │ Token Endpoint         │
└────────┬─────────┘     └────────┬─────────┘     └────────────┬───────────┘
         │                        │                             │
         │  MCP servers need      │                             │
         │  access tokens         │                             │
         │───────────────────────>│                             │
         │                        │  Sign JWT (RS256, 5min)     │
         │                        │──┐                          │
         │                        │<─┘                          │
         │                        │  RFC 7523 bearer assertion  │
         │                        │────────────────────────────>│
         │                        │                             │  Verify via JWKS
         │                        │           access_token      │
         │                        │<────────────────────────────│
         │  Inject as Auth header │                             │
         │<───────────────────────│                             │
         │       ... agent work ...                             │
         │                        │  Revoke token (RFC 7009)    │
         │                        │────────────────────────────>│
```

## Key Design Decisions

1. **OIDC over static API keys** — Eliminates secret management, tokens are short-lived and auto-revoked.
2. **Workload identity federation** — Same pattern as GitHub Actions OIDC. GitHub is the identity provider.
3. **LaunchDarkly owns the token endpoint** — This plugin is purely a client; it doesn't implement any token exchange logic.
4. **Fallback to API key** — For local development where OIDC isn't available.

## Token Lifecycle

| Stage | What Happens |
|---|---|
| **Issuance** | GitHub signs a JWT with RS256, 5-minute expiry |
| **Exchange** | JWT is sent to LaunchDarkly's token endpoint, exchanged for an access token |
| **Usage** | Access token is injected as `Authorization: Bearer` header for all MCP tool calls |
| **Revocation** | When the agent job completes, GitHub calls the revocation endpoint (RFC 7009) |
