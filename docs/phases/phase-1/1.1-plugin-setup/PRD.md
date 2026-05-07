# PRD — Plugin Setup & Configuration

## Problem Statement

Engineering teams manually create LaunchDarkly feature flags when building new features, which is error-prone, inconsistent, and disconnected from the GitHub workflow where the actual code changes happen.

## Goals

1. Automate feature flag creation directly from GitHub PRs and issues
2. Ensure consistent flag naming, tagging, and documentation
3. Prevent duplicate flags from being created
4. Provide zero-configuration authentication via OIDC
5. Report flag creation status back to the developer in the PR/issue

## Target Users

- Software engineers who use both GitHub and LaunchDarkly
- Teams practicing trunk-based development with feature flags
- Engineering leads who want consistent flag hygiene

## Requirements

| ID | Requirement | Priority |
|---|---|---|
| R1 | Plugin must be installable via GitHub Copilot Extensions marketplace | Must |
| R2 | Plugin must connect to LaunchDarkly via MCP server | Must |
| R3 | Authentication must work via OIDC without manual API key setup | Must |
| R4 | Fallback to API key for local development | Should |
| R5 | Pre-run validation must abort if no auth is available | Must |
| R6 | Plugin must support multiple LaunchDarkly projects | Must |

## Success Metrics

- Reduction in time from code change to flag creation
- Consistent flag naming across the organization
- Zero duplicate flags created by the automation
