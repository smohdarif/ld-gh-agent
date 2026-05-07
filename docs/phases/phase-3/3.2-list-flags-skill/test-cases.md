# Test Cases — List Flags Skill

## TC-3.2.1: List all flags in a project

| Field | Value |
|---|---|
| **Input** | project_key="default" |
| **Expected** | Returns list of up to 20 flags with key, name, and date |

## TC-3.2.2: Search by flag key

| Field | Value |
|---|---|
| **Input** | project_key="default", search="checkout" |
| **Expected** | Returns only flags with "checkout" in key or name |

## TC-3.2.3: Filter by tag

| Field | Value |
|---|---|
| **Input** | project_key="default", tag="repo:my-org/my-repo" |
| **Expected** | Returns only flags tagged with the specified repo |

## TC-3.2.4: No matching flags

| Field | Value |
|---|---|
| **Input** | project_key="default", search="nonexistent-flag-xyz" |
| **Expected** | Returns "No existing flags found matching your criteria." |

## TC-3.2.5: Exact key match highlighted

| Field | Value |
|---|---|
| **Input** | project_key="default", search="new-checkout-flow" (flag exists) |
| **Expected** | Flag is highlighted as an existing match |
