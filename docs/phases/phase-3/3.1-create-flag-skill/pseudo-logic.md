# Pseudo Logic — Create Flag Skill

```
FUNCTION create_flag(project_key, flag_key, flag_name, description, tags?, temporary?, flag_type?):

  1. CALL MCP create_feature_flag:
       project_key = project_key
       key         = flag_key
       name        = flag_name
       description = description
       tags        = tags (if provided)
       kind        = flag_type (default: "boolean")
       temporary   = temporary (default: true)

  2. IF creation fails (flag already exists):
       RETURN "Flag `{flag_key}` already exists.\nURL: https://app.launchdarkly.com/projects/{project_key}/flags/{flag_key}"

  3. CALL MCP get_feature_flag:
       project_key = project_key
       flag_key    = flag_key

  4. IF flag found:
       RETURN "Created flag: **{flag_name}** (`{flag_key}`)\nProject: {project_key}\nURL: https://app.launchdarkly.com/projects/{project_key}/flags/{flag_key}"

  5. ELSE:
       RETURN "Error: Flag creation could not be verified."
```
