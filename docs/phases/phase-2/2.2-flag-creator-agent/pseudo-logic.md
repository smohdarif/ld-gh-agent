# Pseudo Logic — Flag Creator Sub-Agent

## Flag Creation Flow

```
WHEN invoked by main agent with parameters:
  INPUT: project_key, flag_key, flag_name, description, tags, flag_type, temporary

  1. CREATE flag:
     CALL MCP tool: create_feature_flag
       project_key = {project_key}
       key         = {flag_key}
       name        = {flag_name}
       description = {description}
       tags        = {tags}
       kind        = {flag_type} (default: "boolean")
       temporary   = {temporary}

     IF create fails with "conflict" (flag already exists):
       RETURN {
         success: false,
         flag_key: {flag_key},
         flag_url: "https://app.launchdarkly.com/projects/{project_key}/flags/{flag_key}",
         environments: [],
         error: "Flag already exists"
       }

     IF create fails with "not found" (invalid project):
       RETURN {
         success: false,
         flag_key: {flag_key},
         flag_url: null,
         environments: [],
         error: "Invalid project key: {project_key}"
       }

     IF create fails with other error:
       RETURN {
         success: false,
         flag_key: {flag_key},
         flag_url: null,
         environments: [],
         error: {error_message}
       }

  2. VERIFY flag was created:
     CALL MCP tool: get_feature_flag
       project_key = {project_key}
       flag_key    = {flag_key}

     IF flag exists:
       EXTRACT environments from response
       CONSTRUCT flag_url = "https://app.launchdarkly.com/projects/{project_key}/flags/{flag_key}"
       RETURN {
         success: true,
         flag_key: {flag_key},
         flag_url: {flag_url},
         environments: {environments},
         error: null
       }

     IF flag not found (creation didn't persist):
       RETURN {
         success: false,
         flag_key: {flag_key},
         flag_url: null,
         environments: [],
         error: "Flag creation appeared to succeed but flag was not found on verification"
       }
```
