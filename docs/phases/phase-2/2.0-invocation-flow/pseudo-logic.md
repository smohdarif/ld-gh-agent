# Pseudo Logic — Invocation Flow

## Complete End-to-End Flow

```
// ═══════════════════════════════════════════
// STAGE 1: EVENT DETECTION (GitHub Platform)
// ═══════════════════════════════════════════

WHEN developer performs action in GitHub:
  event = detect_event(action)
  // event.type = "@-mention" | "assignment" | "push" | "pr"
  // event.context = { repo, pr_number, issue_number, comment, user }

  plugin = match_plugin(event, "launchdarkly-agent")
  IF plugin not found:
    IGNORE event
    RETURN


// ═══════════════════════════════════════════
// STAGE 2: PRE-RUN HOOKS (GitHub Platform)
// ═══════════════════════════════════════════

  hooks = load(plugin.hooks)  // hooks.json
  FOR each hook WHERE hook.event == "before:agent:run":
    result = execute(hook.command)  // scripts/validate-ld-context.sh
    IF result.exit_code != 0:
      ABORT agent run
      POST comment: "Agent could not start: authentication not configured"
      RETURN


// ═══════════════════════════════════════════
// STAGE 3: OIDC TOKEN EXCHANGE (GitHub Platform)
// ═══════════════════════════════════════════

  agent_config = load("agents/main.agent.md")
  FOR each mcp_server in agent_config.mcp_servers:
    IF mcp_server.oidc == true:
      jwt = sign_jwt(
        sub: user_id:{event.user.id},
        iss: "https://github.com",
        aud: discover_audience(mcp_server.url),
        exp: now() + 5_minutes
      )
      access_token = exchange_token(mcp_server.url, jwt)
      mcp_server.auth_header = "Bearer " + access_token


// ═══════════════════════════════════════════
// STAGE 4: CONTEXT INJECTION (GitHub Platform)
// ═══════════════════════════════════════════

  context = gather_context(event)
  // IF PR: { diff, title, description, branch, files, comments, labels }
  // IF issue: { title, body, labels, comments, assignees }

  available_tools = agent_config.tools  // ["view", "edit"]
  available_mcp_tools = discover_tools(mcp_server)  // LaunchDarkly tools
  available_sub_agents = discover_agents(plugin) - main_agent  // ["flag-creator"]


// ═══════════════════════════════════════════
// STAGE 5: MAIN AGENT EXECUTION (LLM)
// ═══════════════════════════════════════════

  main_agent = start_agent(
    system_prompt: agent_config.body,  // markdown instructions
    context: context,
    tools: available_tools + available_mcp_tools + available_sub_agents
  )

  // LLM autonomously:
  //   1. Reads context
  //   2. Identifies flag opportunities
  //   3. Derives flag parameters
  //   4. Calls list-flags (MCP tool) to check duplicates
  //   5. Calls flag-creator sub-agent if needed


// ═══════════════════════════════════════════
// STAGE 6: SUB-AGENT EXECUTION (LLM)
// ═══════════════════════════════════════════

  WHEN main_agent invokes "flag-creator":
    sub_agent_config = load("agents/flag-creator.agent.md")
    sub_agent = start_agent(
      system_prompt: sub_agent_config.body,
      input: { project_key, flag_key, flag_name, ... },  // from main agent
      tools: sub_agent_config.mcp_tools  // LaunchDarkly MCP only
    )

    // Sub-agent LLM autonomously:
    //   1. Calls create_feature_flag (MCP tool)
    //   2. Calls get_feature_flag (MCP tool) to verify
    //   3. Returns structured result

    result = sub_agent.result
    RETURN result TO main_agent


// ═══════════════════════════════════════════
// STAGE 7: REPORT BACK (LLM + Platform)
// ═══════════════════════════════════════════

  main_agent composes summary comment from result
  platform.post_comment(event.pr_number, comment)


// ═══════════════════════════════════════════
// STAGE 8: CLEANUP (GitHub Platform)
// ═══════════════════════════════════════════

  FOR each mcp_server WHERE mcp_server.oidc == true:
    revoke_token(mcp_server.url, access_token)

  mark_job_complete()
```
