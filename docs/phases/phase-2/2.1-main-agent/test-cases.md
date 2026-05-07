# Test Cases — Main Agent (Orchestrator)

## TC-2.1.1: Agent detects flag opportunity in PR diff

| Field | Value |
|---|---|
| **Precondition** | PR adds a new component with `if (featureEnabled)` logic |
| **Action** | Agent is @-mentioned in the PR |
| **Expected** | Agent identifies the conditional logic as a flag opportunity |

## TC-2.1.2: Agent detects flag opportunity from TODO comment

| Field | Value |
|---|---|
| **Precondition** | PR diff contains `// TODO: gate this behind a feature flag` |
| **Action** | Agent analyzes the PR diff |
| **Expected** | Agent identifies the TODO as a flag opportunity |

## TC-2.1.3: Agent detects no flag opportunities

| Field | Value |
|---|---|
| **Precondition** | PR is a documentation-only change |
| **Action** | Agent is @-mentioned in the PR |
| **Expected** | Agent responds "No feature flag opportunities detected" |

## TC-2.1.4: Agent checks for duplicates before creating

| Field | Value |
|---|---|
| **Precondition** | Agent has identified a flag opportunity with key `new-checkout-flow` |
| **Action** | Agent invokes `list-flags` skill |
| **Expected** | Agent queries LaunchDarkly for existing flags matching `new-checkout-flow` |

## TC-2.1.5: Agent skips creation when duplicate exists

| Field | Value |
|---|---|
| **Precondition** | Flag `new-checkout-flow` already exists in LaunchDarkly |
| **Action** | Agent checks for duplicates |
| **Expected** | Agent posts comment noting the flag already exists with its URL, does not create a new one |

## TC-2.1.6: Agent derives correct flag key from branch name

| Field | Value |
|---|---|
| **Precondition** | PR branch is `feature/new-checkout-flow-#123` |
| **Action** | Agent derives flag key |
| **Expected** | Flag key is `new-checkout-flow` (lowercase, hyphenated, issue number stripped) |

## TC-2.1.7: Agent tags flag with repo and PR number

| Field | Value |
|---|---|
| **Precondition** | PR #42 in repo `my-org/my-repo` |
| **Action** | Agent prepares flag parameters |
| **Expected** | Tags include `repo:my-org/my-repo` and `pr:42` |

## TC-2.1.8: Agent asks for project key when ambiguous

| Field | Value |
|---|---|
| **Precondition** | Agent cannot determine the LaunchDarkly project key from context |
| **Action** | Agent attempts to create a flag |
| **Expected** | Agent posts a clarifying question asking which project to use |

## TC-2.1.9: Agent posts summary comment after flag creation

| Field | Value |
|---|---|
| **Precondition** | Agent has successfully created a flag |
| **Action** | Flag creation completes |
| **Expected** | Agent posts comment with flag name, key, environments, dashboard link, and suggested next steps |

## TC-2.1.10: Agent handles flag creation failure

| Field | Value |
|---|---|
| **Precondition** | Flag creation fails (e.g., invalid project key) |
| **Action** | Flag-creator sub-agent returns error |
| **Expected** | Agent posts comment with the error and asks for clarification |
