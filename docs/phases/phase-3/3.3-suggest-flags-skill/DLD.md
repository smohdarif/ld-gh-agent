# DLD — Suggest Flags Skill

## Skill Definition

File: `skills/suggest-flags/SKILL.md`

### Front Matter

```yaml
name: suggest-flags
description: Analyze a pull request diff or issue description and suggest feature flags
disable-model-invocation: false
```

### Input Types

| Input | Source |
|---|---|
| PR diff / code snippet | From the PR's changed files |
| Issue title + body | From the GitHub issue |
| Plain description | User-provided text describing the feature |

### Detection Patterns

| Pattern | Example | Signal |
|---|---|---|
| New conditional logic | `if (featureEnabled)`, `switch` | Direct flag candidate |
| Gate comments | `TODO: gate this`, `feature flag` | Explicit developer intent |
| New user-facing code | New components, new API endpoints | Rollout candidate |
| Database/API changes | Migrations, schema changes | Coordinated rollout |
| A/B test setup | `experiment`, `variant`, `% rollout` | Experiment flag |

### Output Format

**Flags suggested:**
```
Suggested flags:

1. **New Checkout Flow** (`new-checkout-flow`)
   Type: boolean | temporary: true
   Why: New checkout component is entirely new and user-facing
   Gate: Wrap the CheckoutV2 component render (lines 42-67)

2. **Payment Provider Switch** (`payment-provider-switch`)
   Type: boolean | temporary: false
   Why: Kill switch for new payment provider integration
   Gate: Wrap the paymentProvider selection logic
```

**No opportunities:**
```
No feature flag opportunities detected in the provided context.
```
