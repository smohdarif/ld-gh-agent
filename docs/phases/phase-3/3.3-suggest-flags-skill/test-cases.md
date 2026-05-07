# Test Cases — Suggest Flags Skill

## TC-3.3.1: Detect flag from conditional logic

| Field | Value |
|---|---|
| **Input** | PR diff with `if (newFeatureEnabled) { renderV2() }` |
| **Expected** | Suggests a boolean flag for the new feature |

## TC-3.3.2: Detect flag from TODO comment

| Field | Value |
|---|---|
| **Input** | PR diff with `// TODO: gate this behind a feature flag` |
| **Expected** | Suggests a flag matching the commented code |

## TC-3.3.3: Detect flag from issue description

| Field | Value |
|---|---|
| **Input** | Issue: "Roll out new payment flow to 10% of users initially" |
| **Expected** | Suggests a temporary boolean flag for the payment flow |

## TC-3.3.4: No opportunities in docs-only PR

| Field | Value |
|---|---|
| **Input** | PR diff with only README.md changes |
| **Expected** | Returns "No feature flag opportunities detected" |

## TC-3.3.5: Multiple flag suggestions

| Field | Value |
|---|---|
| **Input** | PR diff with a new component + a new API endpoint + a kill switch comment |
| **Expected** | Suggests 2-3 well-scoped flags (not one per line) |

## TC-3.3.6: Temporary flag classification

| Field | Value |
|---|---|
| **Input** | Issue describing a gradual rollout |
| **Expected** | Suggested flag has `temporary: true` |

## TC-3.3.7: Permanent flag classification

| Field | Value |
|---|---|
| **Input** | PR with comment "kill switch for external API" |
| **Expected** | Suggested flag has `temporary: false` |

## TC-3.3.8: Gate description included

| Field | Value |
|---|---|
| **Input** | PR diff with identifiable code sections |
| **Expected** | Each suggestion includes which code/lines to wrap |
