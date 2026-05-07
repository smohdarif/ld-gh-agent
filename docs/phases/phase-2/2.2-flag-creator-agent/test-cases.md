# Test Cases — Flag Creator Sub-Agent

## TC-2.2.1: Successful flag creation

| Field | Value |
|---|---|
| **Precondition** | Valid project_key, unique flag_key |
| **Action** | Sub-agent calls create_feature_flag then get_feature_flag |
| **Expected** | Returns `success: true` with flag URL and environments list |

## TC-2.2.2: Flag already exists

| Field | Value |
|---|---|
| **Precondition** | Flag with the given key already exists in the project |
| **Action** | Sub-agent calls create_feature_flag |
| **Expected** | Returns `success: false` with error "Flag already exists" and existing flag URL |

## TC-2.2.3: Invalid project key

| Field | Value |
|---|---|
| **Precondition** | Project key does not exist in LaunchDarkly |
| **Action** | Sub-agent calls create_feature_flag |
| **Expected** | Returns `success: false` with clear error about invalid project key |

## TC-2.2.4: Boolean flag with correct defaults

| Field | Value |
|---|---|
| **Precondition** | flag_type is "boolean" (or not specified) |
| **Action** | Sub-agent creates the flag |
| **Expected** | Flag is created with `true`/`false` variations, off by default |

## TC-2.2.5: Flag created with tags

| Field | Value |
|---|---|
| **Precondition** | Tags `["repo:my-org/my-repo", "pr:42"]` provided |
| **Action** | Sub-agent creates the flag |
| **Expected** | Flag in LaunchDarkly has both tags applied |

## TC-2.2.6: Temporary flag

| Field | Value |
|---|---|
| **Precondition** | `temporary: true` is set |
| **Action** | Sub-agent creates the flag |
| **Expected** | Flag is marked as temporary in LaunchDarkly |

## TC-2.2.7: Verification after creation

| Field | Value |
|---|---|
| **Precondition** | Flag creation API returns success |
| **Action** | Sub-agent calls get_feature_flag |
| **Expected** | Flag is found, environments list is extracted and returned |
