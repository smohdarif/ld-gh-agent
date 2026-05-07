# LaunchDarkly Flag Automation — GitHub Copilot Extensions Plugin

A GitHub Copilot Extensions plugin that automatically creates and manages [LaunchDarkly](https://launchdarkly.com) feature flags directly from GitHub pull requests and issues.

## How It Works

1. **Trigger** — The agent is invoked via `@`-mentions in PRs, issues, or discussions. It can also be triggered by assignment to an issue or by push/PR events.
2. **Analyze** — The agent reads the PR diff, issue body, or comment to identify code that would benefit from a feature flag (new conditional logic, `TODO: gate this` comments, gradual rollouts, A/B tests, etc.).
3. **Check for duplicates** — Before creating anything, it lists existing flags in the target LaunchDarkly project to avoid duplicates.
4. **Create flags** — It creates the appropriate feature flag(s) in LaunchDarkly with correct naming, keys, tags, and descriptions derived from the PR/issue context.
5. **Report back** — It posts a comment on the PR or issue with the flag name, key, environments, a direct link to the LaunchDarkly dashboard, and suggested next steps.

## Why "Plugin"?

In the GitHub Copilot Extensions ecosystem, a **plugin** is the packaging format — it's the unit of distribution. The `plugin.json` manifest is what the platform reads to discover and register all agents, skills, and hooks. The term simply means "a package that plugs into the Copilot Extensions platform."

## Invocation Flow — Who Calls What

There is **no application code** connecting the components. The **GitHub Copilot Extensions platform** is the orchestrator — it detects events, invokes agents, routes calls between agents, and posts results.

### How the Main Agent Gets Invoked

The developer never calls the agent directly. The **platform** invokes it based on GitHub events:

| Developer Action | Platform Detects | Result |
|---|---|---|
| `@launchdarkly-agent` in a PR comment | @-mention event | Main agent starts with PR context |
| Assigns agent to an issue | Assignment event | Main agent starts with issue context |
| Opens/pushes a PR | Push/PR event (if configured) | Main agent starts with diff context |

Before starting, the platform runs the `before:agent:run` hook (`validate-ld-context.sh`) and performs the OIDC token exchange.

### How the Main Agent Calls the Flag-Creator Sub-Agent

Both agents are registered in the same plugin (discovered from the `agents/` directory). The platform makes sub-agents available as **callable tools** to the main agent. There is no import, function call, or API client — the main agent's LLM decides when to invoke the sub-agent based on its natural language instructions, and the platform handles the routing.

### Full Invocation Chain

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. DEVELOPER                                                     │
│    @launchdarkly-agent please create flags for this PR           │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. GITHUB COPILOT EXTENSIONS PLATFORM                            │
│    - Detects @-mention event                                     │
│    - Runs hook: validate-ld-context.sh ✓                         │
│    - Performs OIDC token exchange with LaunchDarkly               │
│    - Injects PR context (diff, title, branch, comments)          │
│    - Starts main agent with context + tools + MCP connection     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. MAIN AGENT (LLM reasoning)                                    │
│    - Reads PR diff → finds new feature code                      │
│    - Decides: "this needs a feature flag"                        │
│    - Derives: key=new-checkout-flow, name=New Checkout Flow      │
│    - Calls list-flags skill → no duplicate found                 │
│    - Calls flag-creator sub-agent with structured params         │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Platform routes the call
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. FLAG-CREATOR SUB-AGENT (LLM reasoning)                        │
│    - Receives: { project_key, flag_key, flag_name, ... }         │
│    - Calls MCP tool: create_feature_flag → LaunchDarkly API      │
│    - Calls MCP tool: get_feature_flag → verify                   │
│    - Returns: { success: true, flag_url: "...", environments }   │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Result returned to main agent
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. MAIN AGENT (continues)                                        │
│    - Receives result from flag-creator                           │
│    - Posts PR comment:                                           │
│      "Created flag: **New Checkout Flow** (`new-checkout-flow`)  │
│       URL: https://app.launchdarkly.com/...                      │
│       Next: wrap lines 42-67 in your flag evaluation"            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. PLATFORM                                                      │
│    - Posts the comment on the PR                                 │
│    - Revokes the OIDC token                                      │
│    - Job complete                                                │
└─────────────────────────────────────────────────────────────────┘
```

### How the Agent Accesses the Codebase

The agent does **not** call the GitHub REST API or use `git` commands directly. It accesses code through two channels:

1. **Platform-injected context** — When triggered, the platform automatically provides the PR diff, title, description, branch name, changed files, and comments.
2. **Platform tools** — The main agent declares `tools: ["view", "edit"]`, which are GitHub-provided tools that let it read and modify files in the repository. The flag-creator sub-agent has `tools: []` because it only needs LaunchDarkly MCP access.

## Project Structure

```
.
├── plugin.json                     # Plugin manifest
├── hooks.json                      # Pre-run hooks configuration
├── agents/
│   ├── main.agent.md               # Main orchestrator agent
│   └── flag-creator.agent.md       # Sub-agent for creating a single flag
├── skills/
│   ├── create-flag/SKILL.md        # Skill: create a feature flag
│   ├── list-flags/SKILL.md         # Skill: list existing flags (duplicate check)
│   └── suggest-flags/SKILL.md      # Skill: analyze a diff/issue and suggest flags
└── scripts/
    └── validate-ld-context.sh      # Pre-run auth validation script
```

### Agents

| Agent | Purpose |
|---|---|
| **`main.agent.md`** | The primary agent. Analyzes GitHub context, detects flag opportunities, delegates flag creation, and posts results back to the PR/issue. |
| **`flag-creator.agent.md`** | A specialized sub-agent invoked by the main agent to handle the structured workflow of creating a single LaunchDarkly feature flag and verifying it was created successfully. |

### Skills

Skills are a **GitHub Copilot Extensions** concept (not Claude Code). They are reusable task definitions written in markdown, discovered from the `skills/` directory. Skills don't have their own MCP connections or runtime — when an agent invokes a skill, the skill's instructions are loaded into the agent's LLM context, and the agent follows them using its own tools.

| Skill | Purpose |
|---|---|
| **`create-flag`** | Creates a new feature flag in LaunchDarkly with a given project key, flag key, name, description, tags, and type. |
| **`list-flags`** | Lists existing flags in a LaunchDarkly project, optionally filtered by tag or search query. Used before creation to prevent duplicates. |
| **`suggest-flags`** | Analyzes a PR diff or issue description and returns a structured list of recommended feature flags with keys, names, types, and rationale. |

### Why Skills AND a Sub-Agent?

There is intentional overlap between the `flag-creator` sub-agent and the `create-flag` skill — both describe how to call `create_feature_flag` and verify with `get_feature_flag`. This exists because they serve different invocation patterns:

| Aspect | Skill (e.g., `create-flag`) | Sub-Agent (`flag-creator`) |
|---|---|---|
| **Runtime** | Runs in the invoking agent's LLM context | Runs in its own separate LLM context |
| **MCP connection** | Uses the invoking agent's | Has its own |
| **Error handling** | Text output | Structured JSON contract (success/error) |
| **Overhead** | Low (no new LLM instance) | Higher (separate LLM call) |
| **Reusability** | Any agent can use it | Can only be invoked as a sub-agent |

The skills design pays off when more agents are added to the plugin. A hypothetical `flag-cleanup-agent` could reuse the `list-flags` skill without duplicating instructions. With just one main agent and one sub-agent (the current state), the benefit is primarily architectural cleanliness and future-proofing.

For a deeper analysis, see `docs/phases/phase-3/3.0-skills-design-rationale/`.

## MCP Server — How It Connects to LaunchDarkly

This plugin does **not** run or host any server itself. It is purely a client that connects to **LaunchDarkly's hosted MCP (Model Context Protocol) server** at:

```
https://mcp.launchdarkly.com/mcp/fm
```

This server is **owned, hosted, and maintained by LaunchDarkly**. It exposes LaunchDarkly API operations (create flag, list flags, get flag, etc.) as MCP tools that the agents can call.

### MCP Server Configuration

Both agents declare the MCP server connection in their front matter (`agents/main.agent.md` and `agents/flag-creator.agent.md`):

```yaml
mcp-servers:
  launchdarkly:
    type: http
    url: https://mcp.launchdarkly.com/mcp/fm
    tools: ["*"]
    oidc: true
```

- **`type: http`** — Connects to the MCP server over HTTP.
- **`url`** — The remote LaunchDarkly MCP server endpoint.
- **`tools: ["*"]`** — Grants the agent access to all tools exposed by the MCP server (e.g., `create_feature_flag`, `list_feature_flags`, `get_feature_flag`).
- **`oidc: true`** — Tells the agent framework to use OIDC-based authentication (see below).

## Authentication

### OIDC Authentication (Primary) — How the Token Exchange Works

The plugin uses **OIDC (OpenID Connect)** as its primary authentication method, based on GitHub's **3rd Party Token Support for MCPs** — a workload identity federation pattern (the same concept behind GitHub Actions OIDC).

Because `oidc: true` is set in the agent front matter, GitHub handles the entire token exchange automatically. **No API keys or secrets need to be stored.**

Here's the step-by-step flow:

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────────────┐
│ Copilot (runtime) │     │ GitHub (sweagentd)│     │ LaunchDarkly MCP Server  │
│                  │     │                  │     │ (token endpoint)         │
└────────┬─────────┘     └────────┬─────────┘     └────────────┬─────────────┘
         │                        │                             │
         │  "These MCP servers    │                             │
         │   need access tokens"  │                             │
         │───────────────────────>│                             │
         │                        │                             │
         │                        │  Read OIDC config for       │
         │                        │  MCP server                 │
         │                        │──┐                          │
         │                        │<─┘                          │
         │                        │                             │
         │                        │  Sign JWT (RS256, 5min exp) │
         │                        │──┐                          │
         │                        │<─┘                          │
         │                        │                             │
         │                        │  RFC 7523 bearer assertion  │
         │                        │  (assertion = signed JWT)   │
         │                        │────────────────────────────>│
         │                        │                             │
         │                        │                             │  Verify JWT via
         │                        │                             │  /.well-known/jwks.json
         │                        │                             │──┐
         │                        │                             │<─┘
         │                        │                             │
         │                        │           access_token      │
         │                        │<────────────────────────────│
         │                        │                             │
         │  Access token injected │                             │
         │  as Authorization hdr  │                             │
         │<───────────────────────│                             │
         │                        │                             │
         │           ... agent work proceeds ...                │
         │                        │                             │
         │                        │  Request token revocation   │
         │                        │────────────────────────────>│
         │                        │                             │  Revoke token
         │                        │                             │──┐
         │                        │                             │<─┘
```

**Step by step:**

1. **GitHub signs a short-lived JWT** (5-minute expiry, RS256) containing identity claims about the workload — the agent, repo, org, and user that triggered the request.
2. **GitHub sends the JWT** to LaunchDarkly's token exchange endpoint using the standard RFC 7523 bearer assertion flow.
3. **LaunchDarkly's endpoint verifies the JWT** using GitHub's public keys (fetched from `/.well-known/jwks.json`), validates standard claims (iss, aud, exp), and uses the `sub` claim and other claims to determine what access to grant.
4. **LaunchDarkly returns an access token** which is injected into the MCP server as a standard OAuth Bearer `Authorization` header for all tool calls.
5. **When the job completes**, GitHub calls LaunchDarkly's revocation endpoint (RFC 7009) to explicitly invalidate the token.

### What's in the JWT (Identity Claims)

The signed JWT contains claims that identify who and what is making the request:

| Claim | Description |
|---|---|
| `sub` | The subject — identifies the workload. Can be a `user_id:{id}`, `installation_id:{id}`, or `app:{client_id}:owner_id:...` depending on configuration. |
| `preferred_username` | The GitHub login of the user who triggered the agent (always present). |
| `user_id` | The GitHub user ID (always present). |
| `iss` | The issuer (GitHub). |
| `aud` | The audience (the MCP server / LaunchDarkly). |
| `exp` | Expiration time (5 minutes from issuance). |

LaunchDarkly's token endpoint uses these claims to determine **what permissions the returned access token has** — scoped to the user's LaunchDarkly account and permissions. The `sub` claim granularity determines authorization scope: a subject with only the agent identifier applies the same trust policy to all workloads, while including the user ID allows per-user permission scoping.

### User on-behalf-of Flow

For this plugin, the token exchange identifies *which user* triggered the work (via `preferred_username` and `user_id` claims). LaunchDarkly can use this to:
- Map the GitHub user to a LaunchDarkly account (via a one-time account linking process)
- Return an access token scoped to that user's LaunchDarkly permissions
- Ensure the agent can only create/list flags that the user themselves would have access to

### Token Security

- Tokens are **short-lived** — the JWT expires in 5 minutes, and the access token is revoked when the agent job completes.
- **No long-lived secrets** are stored in the repository or environment.
- GitHub **automatically revokes** tokens at the end of processing to minimize the window of exposure.

### API Key Authentication (Fallback)

As a fallback, the plugin also supports a static LaunchDarkly API key via the `LD_API_KEY` environment variable. If OIDC is not available (e.g., during local development or testing), you can set this variable and the MCP server headers can be configured to use it:

```bash
export LD_API_KEY="your-launchdarkly-api-key"
```

### Pre-Run Validation

Before the agent runs, a hook (`scripts/validate-ld-context.sh`) checks that at least one authentication method is available. If neither `GITHUB_COPILOT_OIDC_MCP_TOKEN` nor `LD_API_KEY` is set, the agent run is aborted with an error message.

## Which LaunchDarkly Project Does It Use?

The plugin does **not** hardcode a specific LaunchDarkly project. The **project key is determined at runtime** based on context:

- The agent infers the project key from the PR/issue context, repository configuration, or conversation with the user.
- The `create-flag` and `list-flags` skills both require a `project_key` parameter (e.g., `default`, `my-project`).
- If the agent **cannot determine** the correct project key from context alone, it will **ask a clarifying question** in the PR/issue comment thread rather than guessing.

This means the plugin can work across multiple LaunchDarkly projects — whichever project is relevant to the repository or team using it.

## API Key vs SDK Key — What This Plugin Uses

LaunchDarkly has two distinct types of keys, and it's important to understand which one this plugin uses:

| Key Type | Purpose | Used By This Plugin? |
|---|---|---|
| **API Key** (or OIDC token) | Management operations — create, update, delete, and list flags via the LaunchDarkly REST API / MCP server. | **Yes** — this is what the plugin uses to create and manage flags. |
| **SDK Key** | Runtime flag evaluation — your application code uses this to check flag values (e.g., `ldClient.variation('new-checkout-flow', user, false)`). Each environment in a project has its own SDK key. | **No** — this plugin does not handle SDK keys. |

### What this means in practice

1. **This plugin** authenticates with an API-level credential (OIDC token or `LD_API_KEY`) to talk to the LaunchDarkly MCP server and create/manage flags. It never evaluates flags.
2. **Your application code** still needs the LaunchDarkly SDK integrated with the appropriate **SDK key** for the target environment (e.g., `production`, `staging`) to actually evaluate the flags at runtime.
3. After the plugin creates a flag and posts the details on your PR, the developer is responsible for:
   - Adding the LaunchDarkly SDK to the application (if not already present)
   - Using the correct **SDK key** for the environment
   - Wrapping the relevant code with the flag evaluation (e.g., `if variation('flag-key') ...`)

The SDK key is found in your LaunchDarkly project under **Account Settings > Projects > {project} > Environments > {environment}**. It is not something this plugin creates or configures.

## What the Plugin Creates (and What It Doesn't)

### What it creates

The plugin creates a **flag definition** with the following metadata:

| Field | Source |
|---|---|
| `flag_key` | Derived from PR branch name or feature description |
| `flag_name` | PR or issue title |
| `description` | Summary of what the flag gates |
| `tags` | Repo name, PR/issue number |
| `flag_type` | `boolean` (default), `string`, `number`, or `json` |
| `temporary` | `true` for rollout/experiment flags, `false` for kill switches |

The flag is created in an **off** state across all environments in the project.

### What it does NOT create or configure

- **Targeting rules** — No individual user or context targeting is set up.
- **Percentage rollouts** — No gradual rollout percentages are configured.
- **Segments** — No user segments are created or attached.
- **Variations** — For boolean flags, the default `true`/`false` variations are used. No custom variations are defined.
- **Prerequisites or dependencies** — No flag prerequisites are configured.
- **Approvals or scheduled changes** — No workflows are triggered.

After the plugin creates a flag, the developer or team lead needs to go to the **LaunchDarkly dashboard** to:

1. Turn the flag **on** in the desired environment(s)
2. Set up **targeting rules** (e.g., target specific users, contexts, or segments)
3. Configure **percentage rollouts** if doing a gradual release
4. Add any **prerequisites** or **custom variations** as needed

## Flag Naming Conventions

| Field | Convention |
|---|---|
| **Key** | Lowercase, hyphen-separated (e.g., `new-checkout-flow`). Derived from the PR branch name or feature description. |
| **Name** | The PR or issue title. |
| **Tags** | Repo name and PR/issue number (e.g., `repo:my-org/my-repo`, `pr:123`). |
| **Description** | A short summary of what the flag gates. |

## Getting Started

1. Install the plugin in your GitHub organization or repository.
2. Ensure LaunchDarkly OIDC is configured for your organization, **or** set the `LD_API_KEY` environment variable.
3. Mention the agent in a PR or issue to trigger flag analysis and creation.

## Using This Agent on an Existing Repo

This section walks through adding the agent to an existing codebase that already uses (or wants to use) LaunchDarkly. We'll use [react_qr_app_ld_beginners](https://github.com/smohdarif/react_qr_app_ld_beginners) — a React QR code demo app with 8 existing LaunchDarkly feature flags — as a real example.

### Step 1: Copy the Plugin Files Into Your Repo

Copy the plugin's agent, skill, hook, and script files into your repository. The exact location depends on how your GitHub Copilot Extensions environment discovers plugins — it may be at the repo root or under `.github/copilot/`:

```
your-repo/
├── .github/
│   └── copilot/                          ← add this directory
│       ├── plugin.json
│       ├── hooks.json
│       ├── agents/
│       │   ├── main.agent.md
│       │   └── flag-creator.agent.md
│       ├── skills/
│       │   ├── create-flag/SKILL.md
│       │   ├── list-flags/SKILL.md
│       │   └── suggest-flags/SKILL.md
│       └── scripts/
│           └── validate-ld-context.sh
├── src/                                  ← your existing code
└── ...
```

### Step 2: Check Prerequisites

| Requirement | What to Do |
|---|---|
| **GitHub Copilot Extensions** | Must be enabled for your GitHub org/account. Check your organization's Copilot settings. |
| **LaunchDarkly account** | You need an active LaunchDarkly account with at least one project. |
| **Authentication (Option A: OIDC)** | Set up OIDC trust between GitHub and LaunchDarkly so the agent can authenticate without stored secrets. This is a one-time setup in your LaunchDarkly account under **Account Settings > Authorization**. |
| **Authentication (Option B: API Key)** | Alternatively, create a LaunchDarkly API key and set it as `LD_API_KEY` environment variable or GitHub repo secret. Simpler but less secure. |

### Step 3: Know Your LaunchDarkly Project Key

The agent needs to know which LaunchDarkly project to create flags in. You can find your project key in the LaunchDarkly dashboard under **Account Settings > Projects**.

For example, the `react_qr_app_ld_beginners` repo uses a project with flags like `show-qr-code`, `release-new-ui`, and `config-background-color`. The agent will:

- **Detect existing flags** via the `list-flags` skill and avoid creating duplicates
- **Ask you** for the project key if it can't determine it from context
- **Create new flags** only for genuinely new features in your PRs

### Step 4: Trigger the Agent

Once installed, trigger the agent by @-mentioning it in a PR or issue:

**Analyze a PR for flag opportunities:**
```
@launchdarkly-agent analyze this PR and suggest feature flags
```

**Create a flag for a specific feature:**
```
@launchdarkly-agent create a feature flag for the new payment flow in this PR
```

**Check if a flag already exists:**
```
@launchdarkly-agent does a flag for QR code display already exist?
```

### Step 5: What Happens Next

The agent will post a comment on your PR/issue with:

```
Created flag: **New Payment Flow** (`new-payment-flow`)
Project: default
Environments: production, staging, development
URL: https://app.launchdarkly.com/projects/default/flags/new-payment-flow

Suggested next steps:
- The flag is currently OFF in all environments
- Wrap the PaymentV2 component (lines 42-67) in your flag evaluation
- Turn it on in staging first to test
```

### Step 6: Integrate the Flag in Your Code

The agent creates the flag definition in LaunchDarkly but does **not** modify your application code. You still need to add the flag evaluation yourself using the LaunchDarkly SDK.

For a React app like `react_qr_app_ld_beginners` (which uses the LaunchDarkly React SDK):

```jsx
import { useFlags } from 'launchdarkly-react-client-sdk';

function MyComponent() {
  const { newPaymentFlow } = useFlags();

  if (newPaymentFlow) {
    return <PaymentV2 />;
  }
  return <PaymentV1 />;
}
```

For other frameworks/languages, use the appropriate LaunchDarkly SDK:

| Stack | SDK | Flag Evaluation |
|---|---|---|
| React | `launchdarkly-react-client-sdk` | `useFlags()` hook |
| Node.js | `@launchdarkly/node-server-sdk` | `client.variation('flag-key', context, default)` |
| Python | `launchdarkly-server-sdk` | `client.variation('flag-key', context, default)` |
| Go | `github.com/launchdarkly/go-server-sdk` | `client.BoolVariation("flag-key", context, false)` |

### What the Agent Will and Won't Do

| Will Do | Won't Do |
|---|---|
| Analyze PR diffs for flag opportunities | Modify your application code |
| Check for duplicate flags | Add SDK imports or flag evaluations |
| Create flag definitions (off by default) | Set up targeting rules or rollouts |
| Tag flags with repo/PR info | Change your SDK key configuration |
| Post summary comments with links and next steps | Touch your Terraform/IaC config |
| Ask clarifying questions when unsure | Guess the wrong project key |

### Example: What Would Happen on `react_qr_app_ld_beginners`

If you opened a PR adding a new "dark mode" feature to the QR app:

1. **You mention**: `@launchdarkly-agent analyze this PR`
2. **Agent reads the diff**: Sees a new `DarkModeToggle` component with conditional rendering
3. **Agent checks duplicates**: Calls `list-flags` — finds 8 existing flags, none for dark mode
4. **Agent creates flag**: `dark-mode-toggle` with tags `repo:smohdarif/react_qr_app_ld_beginners`, `pr:15`
5. **Agent posts comment**: Flag name, key, dashboard URL, and "wrap the DarkModeToggle render in your flag evaluation"
6. **You add the code**: `const { darkModeToggle } = useFlags()` in your component
7. **You turn it on**: Go to LaunchDarkly dashboard, enable in staging, then production

## License

MIT
