# GitHub Agent Approaches Compared

## Overview

GitHub Copilot Extensions is **not the only way** to create agents and automation in GitHub. This document compares all available approaches and explains why this plugin uses Copilot Extensions.

## All Approaches at a Glance

| Approach | What It Is | How Agents Are Defined | Best For |
|---|---|---|---|
| **GitHub Copilot Extensions** (this repo) | Plugin framework for extending Copilot with custom agents | Markdown + YAML (`.agent.md`, `SKILL.md`) | AI-powered agents that reason, analyze code, and interact in PRs/issues |
| **GitHub Actions** | CI/CD workflow automation | YAML workflow files (`.github/workflows/*.yml`) | Build, test, deploy, automated checks, scheduled jobs |
| **GitHub Apps** | Custom integrations that receive webhooks and call GitHub APIs | Any language (Node.js, Python, Go, etc.) — you host the server | Complex integrations, bots, custom PR checks, dashboards |
| **GitHub Copilot Coding Agent** | GitHub's built-in AI agent that can write code and open PRs | Triggered by assigning issues to Copilot | Automated code generation, bug fixes, simple feature implementation |
| **Probot** | Framework for building GitHub Apps in Node.js | JavaScript/TypeScript — you host it | GitHub bots (auto-label, auto-merge, comment bots) |
| **GitHub Script** | Run JavaScript directly in GitHub Actions using the GitHub API | JavaScript inside a workflow step | Quick API automations without a full app |

## Detailed Comparison

### 1. GitHub Copilot Extensions (This Repo)

**What it is:** A plugin framework that lets you add custom agents, skills, and tools to GitHub Copilot. Agents are defined in markdown with YAML front matter — no application code needed.

**How it works:**
```
Developer @-mentions agent in PR
    -> Platform invokes the agent
    -> LLM reads the markdown instructions
    -> LLM reasons about the code/context
    -> LLM uses MCP tools to interact with external services
    -> Platform posts the result back
```

**Pros:**
- No application code to write or maintain
- LLM-powered analysis — understands code semantics, not just regex patterns
- MCP protocol for connecting to external services (LaunchDarkly, etc.)
- OIDC authentication — no secrets to manage
- Natural language instructions — easy to modify behavior

**Cons:**
- Requires GitHub Copilot Extensions access (not universally available yet)
- LLM reasoning is probabilistic — not 100% deterministic
- Limited to what the platform provides (tools, MCP servers)
- Newer technology — less documentation and community support

**Example files:**
```
plugin.json
agents/main.agent.md
skills/create-flag/SKILL.md
```

### 2. GitHub Actions

**What it is:** GitHub's built-in CI/CD system. You define workflows in YAML that run on events (push, PR, schedule, etc.).

**How it works:**
```
PR is opened
    -> GitHub triggers the workflow
    -> Workflow runs on a virtual machine
    -> Steps execute shell commands, scripts, or pre-built actions
    -> Results posted via GitHub API
```

**Pros:**
- Widely available — works on any GitHub repo today
- Deterministic — same input = same output
- Huge marketplace of pre-built actions
- Full control over the execution environment
- Free for public repos, generous free tier for private

**Cons:**
- You write all the logic yourself — no LLM reasoning
- Can't "understand" code semantically (only regex/pattern matching)
- YAML can get complex for advanced workflows
- You manage secrets (API keys stored as repo secrets)

**Example — same flag automation as a GitHub Action:**
```yaml
# .github/workflows/flag-automation.yml
name: LaunchDarkly Flag Automation
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  suggest-flags:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Get PR diff
        id: diff
        run: |
          gh pr diff ${{ github.event.pull_request.number }} > diff.txt
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Check for flag patterns
        run: |
          # Rule-based: look for TODO comments and feature toggles
          if grep -q "TODO: gate this\|featureEnabled\|feature_flag" diff.txt; then
            echo "FLAG_NEEDED=true" >> $GITHUB_ENV
          fi

      - name: Create flag in LaunchDarkly
        if: env.FLAG_NEEDED == 'true'
        run: |
          curl -X POST \
            -H "Authorization: ${{ secrets.LD_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{
              "name": "${{ github.event.pull_request.title }}",
              "key": "pr-${{ github.event.pull_request.number }}-flag",
              "kind": "boolean"
            }' \
            https://app.launchdarkly.com/api/v2/flags/default

      - name: Post comment on PR
        if: env.FLAG_NEEDED == 'true'
        run: |
          gh pr comment ${{ github.event.pull_request.number }} \
            --body "Created flag: pr-${{ github.event.pull_request.number }}-flag"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3. GitHub Apps

**What it is:** A custom integration that receives webhooks from GitHub and can interact with repos, PRs, issues via the GitHub API. You write and host the server yourself.

**How it works:**
```
PR is opened
    -> GitHub sends a webhook to your server
    -> Your server processes the event
    -> Your server calls GitHub API + LaunchDarkly API
    -> Posts results back to the PR
```

**Pros:**
- Full control — any language, any logic, any integration
- Can react to any GitHub event in real-time
- Fine-grained permissions
- Can have a UI (dashboard, settings page)

**Cons:**
- You host and maintain a server (or use serverless functions)
- You write all the code
- You manage authentication, secrets, scaling
- More complex setup

**Example — same flag automation as a GitHub App (Node.js):**
```javascript
// server.js
const { App } = require("@octokit/app");
const LaunchDarkly = require("launchdarkly-node-server-sdk");

app.webhooks.on("pull_request.opened", async ({ payload }) => {
  const diff = await getDiff(payload.pull_request);

  // Your analysis logic (no LLM — you write the rules)
  if (diff.includes("TODO: gate this")) {
    await createFlag({
      key: `pr-${payload.pull_request.number}-flag`,
      name: payload.pull_request.title,
    });

    await postComment(payload.pull_request.number,
      "Created flag for this PR"
    );
  }
});
```

### 4. GitHub Copilot Coding Agent

**What it is:** GitHub's built-in AI agent (powered by an LLM) that can read issues, write code, run tests, and open PRs — all autonomously.

**How it works:**
```
Developer assigns an issue to Copilot
    -> Copilot reads the issue
    -> Copilot writes code to solve it
    -> Copilot runs tests
    -> Copilot opens a PR with the changes
```

**Pros:**
- Built into GitHub — no setup needed
- Can write and modify code
- Understands code semantically (LLM-powered)
- Opens PRs with proper descriptions

**Cons:**
- Designed for **code changes**, not for managing external services
- Can't easily call external APIs like LaunchDarkly
- You can't customize its behavior with markdown instructions
- Limited to what GitHub exposes

**Could it do flag automation?** Partially — it could modify your code to add flag evaluations, but it's not designed to call the LaunchDarkly API to create flags.

### 5. Probot

**What it is:** A Node.js framework for building GitHub Apps. Simplifies webhook handling and API calls.

**How it works:** Same as GitHub Apps, but with a simpler developer experience.

**Example:**
```javascript
// index.js
module.exports = (app) => {
  app.on("pull_request.opened", async (context) => {
    const diff = await context.octokit.pulls.get({
      ...context.pullRequest(),
      mediaType: { format: "diff" },
    });

    // Your logic here
    if (needsFlag(diff)) {
      await createLDFlag(...);
      await context.octokit.issues.createComment({
        ...context.issue(),
        body: "Created a feature flag for this PR",
      });
    }
  });
};
```

### 6. GitHub Script

**What it is:** A GitHub Action that lets you write JavaScript directly in your workflow YAML, with the GitHub API client pre-configured.

**Example:**
```yaml
- uses: actions/github-script@v7
  with:
    script: |
      const diff = await github.rest.pulls.get({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.issue.number,
        mediaType: { format: 'diff' }
      });

      if (diff.data.includes('TODO: gate this')) {
        await github.rest.issues.createComment({
          ...context.repo,
          issue_number: context.issue.number,
          body: 'This PR might need a feature flag!'
        });
      }
```

## The Key Difference: LLM vs Rule-Based

| GitHub Actions / Apps / Probot / Script | Copilot Extensions (this repo) |
|---|---|
| **You write the logic** — regex, rules, pattern matching | **LLM does the reasoning** — reads code semantically |
| Deterministic — same input = same output | Probabilistic — LLM interprets context |
| Can match `TODO: gate this` (exact string) | Can understand "this new component should probably be behind a flag" |
| Works today, widely available | Requires Copilot Extensions access |
| You maintain the code | Markdown instructions, no code to maintain |
| Can't understand intent | Understands natural language in issues, PRs, and code comments |

### Example: What Each Approach Would Detect

Given this PR diff:
```diff
+ function CheckoutV2({ items, user }) {
+   // New checkout experience with improved UX
+   return (
+     <div className="checkout-v2">
+       <CartSummary items={items} />
+       <PaymentForm user={user} />
+     </div>
+   );
+ }
```

| Approach | What It Detects |
|---|---|
| **GitHub Actions** (regex) | Nothing — no `TODO` or `featureEnabled` keyword to match |
| **GitHub App** (custom logic) | Maybe — if you write rules for "new component" detection |
| **Copilot Extensions** (LLM) | "This is a new user-facing checkout component. It should have a feature flag for gradual rollout." |

The LLM understands **intent and context**, not just text patterns. That's the core advantage of the Copilot Extensions approach.

## Which Should You Choose?

| If You Need... | Use |
|---|---|
| AI-powered code analysis with reasoning | **Copilot Extensions** |
| Deterministic automation on every PR | **GitHub Actions** |
| Full control with custom server logic | **GitHub Apps** |
| Quick bot with minimal code | **Probot** |
| AI to write code and open PRs | **Copilot Coding Agent** |
| Simple one-off API scripting in a workflow | **GitHub Script** |

## Why This Plugin Uses Copilot Extensions

This plugin chose Copilot Extensions because:

1. **Flag detection requires understanding code** — not just pattern matching
2. **Natural language instructions** are easier to maintain than analysis code
3. **MCP protocol** provides a clean integration with LaunchDarkly's API
4. **OIDC authentication** eliminates secret management
5. **No server to host** — the platform runs everything
6. **Conversational** — the agent can ask clarifying questions in PR comments
