# Test Cases — Create Flag Skill

## TC-3.1.1: Create boolean flag with all required params

| Field | Value |
|---|---|
| **Input** | project_key="default", flag_key="new-feature", flag_name="New Feature", description="Gates new feature" |
| **Expected** | Flag created, output shows name, key, project, and URL |

## TC-3.1.2: Create flag with optional tags

| Field | Value |
|---|---|
| **Input** | All required params + tags="repo:org/repo,pr:42" |
| **Expected** | Flag created with both tags applied |

## TC-3.1.3: Create temporary flag

| Field | Value |
|---|---|
| **Input** | All required params + temporary=true |
| **Expected** | Flag marked as temporary in LaunchDarkly |

## TC-3.1.4: Create permanent flag

| Field | Value |
|---|---|
| **Input** | All required params + temporary=false |
| **Expected** | Flag marked as permanent in LaunchDarkly |

## TC-3.1.5: Flag already exists

| Field | Value |
|---|---|
| **Input** | flag_key that already exists in the project |
| **Expected** | Output states flag already exists with its URL, no error thrown |

## TC-3.1.6: Verify after creation

| Field | Value |
|---|---|
| **Input** | Valid creation params |
| **Expected** | get_feature_flag is called after create_feature_flag to confirm |
