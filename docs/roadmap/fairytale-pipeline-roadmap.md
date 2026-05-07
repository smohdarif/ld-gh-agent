# Roadmap: The Fairytale Pipeline — Fully Automated Agentic Release Lifecycle

## Vision

LaunchDarkly's "Fairytale Pipeline" is a fully automated release lifecycle where AI agents handle every step from PR creation to flag retirement — with zero human intervention for standard releases.

```
1. Devin creates a PR for an issue
2. Planner agent assesses risk and routes to specialists
3. Agent reviews PR, creates flag, wraps code in flag, sets rollout plan
4. Agent deploys the new change (flag is OFF)
5. Agent performs a Guarded Release (progressive rollout with metrics monitoring)
6. Guarded Release is successful → Agent retires the flag
```

## Current State vs Target State

```
THE FAIRYTALE PIPELINE                          CURRENT AGENT STATUS
═══════════════════                             ════════════════════

Step 1: Devin creates PR                       Not our scope (external AI coding agent)
            |
            v
Step 2: Tag PR for pipeline                    NOT YET — need planner agent
            |
            v
Step 3: Agent reviews PR,                      PARTIALLY DONE
        creates flag,                              DONE: Reviews PR diff (main agent)
        wraps code in flag,                        DONE: Creates flag (flag-creator)
        adds guarded release metrics,              MISSING: Does NOT wrap code in flag
        sets progressive rollout plan              MISSING: Does NOT set rollout plan
            |
            v
Step 4: Agent deploys (flag OFF)               NOT YET — need deploy agent
            |
            v
Step 5: Agent performs Guarded Release         NOT YET — need release guardian agent
            |
            v
Step 6: Release successful -> retire flag      NOT YET — need flag cleanup agent
```

## New Components Needed

### New Agents

| Agent | Purpose | Status |
|---|---|---|
| `main.agent.md` | Orchestrator — analyze PR, create flag | DONE |
| `flag-creator.agent.md` | Create a single flag via MCP | DONE |
| `planner.agent.md` | Assess risk, route to specialists, create rollout plan | NEW |
| `code-wrapper.agent.md` | Wrap source code in flag evaluations | NEW |
| `deploy.agent.md` | Trigger deployment, verify health | NEW |
| `release-guardian.agent.md` | Progressive rollout with metrics monitoring | NEW |
| `flag-cleanup.agent.md` | Remove flag from code, archive in LaunchDarkly | NEW |

### New Skills

| Skill | Purpose | Status |
|---|---|---|
| `create-flag` | Create a flag in LaunchDarkly | DONE |
| `list-flags` | List existing flags (duplicate check) | DONE |
| `suggest-flags` | Analyze PR/issue for flag opportunities | DONE |
| `wrap-code-in-flag` | Modify source code to add flag evaluation | NEW |
| `set-rollout-plan` | Configure progressive rollout percentages and metrics | NEW |
| `check-compliance` | Verify flag meets org policies | NEW |
| `monitor-metrics` | Watch error rates, latency, business metrics | NEW |
| `advance-rollout` | Increase rollout percentage to next stage | NEW |
| `rollback-flag` | Turn flag OFF immediately if metrics regress | NEW |
| `retire-flag` | Archive/delete flag, open cleanup PR | NEW |

### New MCP Server Integrations

| MCP Server | Purpose | Status |
|---|---|---|
| LaunchDarkly MCP | Create, list, update, delete flags | DONE |
| Metrics/Monitoring MCP | Read error rates, latency, custom metrics (Datadog, New Relic, CloudWatch) | NEW |
| Deployment MCP | Trigger and monitor deployments (Vercel, AWS, GitHub Actions) | NEW |

---

## Detailed Plan: Each New Agent

### Agent 1: Planner Agent

```
File: agents/planner.agent.md

TRIGGER: PR is opened or labeled with "auto-release"

WHAT IT DOES:
  1. Reads the PR diff, title, description, labels
  2. ASSESSES RISK:
     - How many files changed?
     - Does it touch critical paths (auth, payments, database)?
     - Is it a new feature, bug fix, or refactor?
     - Does it have tests?
  3. ASSIGNS A RISK LEVEL: low / medium / high / critical
  4. ROUTES TO SPECIALISTS based on risk:
     - Low risk  -> fast-track: create flag, wrap code, deploy, 100% rollout
     - Medium    -> standard: create flag, wrap code, deploy, 10% -> 50% -> 100%
     - High      -> careful: create flag, wrap code, deploy, 1% -> 5% -> 25% -> 50% -> 100%
     - Critical  -> manual review required, notify team
  5. CREATES A ROLLOUT PLAN:
     {
       risk_level: "medium",
       stages: [
         { percentage: 10, duration: "1h", success_criteria: "error_rate < 0.1%" },
         { percentage: 50, duration: "4h", success_criteria: "error_rate < 0.1%" },
         { percentage: 100, duration: "24h", success_criteria: "error_rate < 0.1%" }
       ]
     }
  6. Posts plan as PR comment, tags PR for the pipeline

SKILLS USED: suggest-flags (existing), check-compliance (new)

MCP SERVERS: LaunchDarkly (existing)

TOOLS: ["view"] — reads code but doesn't modify it
```

### Agent 2: Code Wrapper Agent

```
File: agents/code-wrapper.agent.md

TRIGGER: Invoked by main agent after flag is created

WHAT IT DOES:
  1. Receives: flag_key, file paths, line ranges from main agent
  2. READS the source code using "view" tool
  3. DETECTS the framework/language:
     - React: looks for JSX, component patterns, existing useFlags() usage
     - Node.js: looks for Express routes, middleware patterns
     - Python: looks for Flask/Django patterns
  4. IDENTIFIES where to wrap the code
  5. MODIFIES the code using "edit" tool:

     React example:
       BEFORE:
         function CheckoutV2() {
           return <div>...</div>
         }
       AFTER:
         import { useFlags } from 'launchdarkly-react-client-sdk';
         function CheckoutV2() {
           const { newCheckoutFlow } = useFlags();
           if (!newCheckoutFlow) return <CheckoutV1 />;
           return <div>...</div>
         }

     Node.js example:
       BEFORE:
         app.get('/checkout', (req, res) => { ... });
       AFTER:
         app.get('/checkout', async (req, res) => {
           const showNewCheckout = await ldClient.variation('new-checkout-flow', user, false);
           if (!showNewCheckout) return oldCheckoutHandler(req, res);
           ...
         });

  6. ADDS SDK import if not already present
  7. COMMITS the change to the PR branch
  8. Returns: files modified, lines changed

SKILLS USED: wrap-code-in-flag (new)

MCP SERVERS: LaunchDarkly (existing) — to get flag details

TOOLS: ["view", "edit"] — reads and modifies code
```

### Agent 3: Deploy Agent

```
File: agents/deploy.agent.md

TRIGGER: PR is merged with flag in OFF state

WHAT IT DOES:
  1. Detects PR merge event
  2. Verifies the flag exists and is OFF in all environments via LaunchDarkly MCP
  3. Triggers deployment via one of:
     - Option A: GitHub Actions — triggers a deployment workflow
     - Option B: Deployment MCP — calls Vercel/AWS/etc. deployment API
     - Option C: GitOps — merges to deploy branch
  4. WAITS for deployment to complete
  5. VERIFIES deployment health:
     - Checks health endpoint
     - Verifies no crash loops
     - Confirms service is reachable
  6. Posts status: "Deployed to production. Flag `new-checkout-flow` is OFF."
  7. Hands off to the release guardian agent

SKILLS USED: (none from existing set — uses deployment-specific tools)

MCP SERVERS:
  - LaunchDarkly (existing) — verify flag state
  - Deployment MCP (new) — trigger and monitor deploys

TOOLS: [] — no codebase access needed
```

### Agent 4: Release Guardian Agent

```
File: agents/release-guardian.agent.md

TRIGGER: Deploy agent confirms successful deployment

WHAT IT DOES:
  1. Reads the rollout plan created by the planner agent
  2. EXECUTES the progressive rollout:

     FOR each stage in rollout plan:

       a. UPDATE flag targeting via LaunchDarkly MCP:
          - Set rollout percentage to stage.percentage
          - e.g., 10% of users see the new feature

       b. MONITOR metrics for stage.duration:
          - Error rate (via Metrics MCP — Datadog, New Relic, etc.)
          - Latency p50, p95, p99
          - Custom business metrics (conversion rate, etc.)
          - Compare against stage.success_criteria

       c. EVALUATE:
          IF metrics are healthy:
            -> Post comment: "10% rollout healthy for 1h. Advancing to 50%."
            -> Continue to next stage
          IF metrics regress:
            -> ROLLBACK: Set flag to OFF immediately
            -> Post comment: "ROLLBACK! Error rate spiked to 2.3% at 10% rollout.
               Flag turned OFF. Investigate and retry."
            -> Notify team (tag PR reviewers, post to Slack if available)
            -> STOP the pipeline

  3. After all stages complete successfully:
     Post: "Guarded Release complete. `new-checkout-flow` at 100% for 24h. All healthy."
  4. Hands off to flag cleanup agent

SKILLS USED: monitor-metrics (new), advance-rollout (new), rollback-flag (new)

MCP SERVERS:
  - LaunchDarkly (existing) — update flag targeting percentages
  - Metrics MCP (new) — read error rates, latency, business metrics
  - (Optional) Slack MCP — notify team on rollback

TOOLS: [] — no codebase access needed

CRITICAL CHALLENGE:
  This agent needs to run as a LONG-LIVED PROCESS or be triggered
  periodically (e.g., every 15 minutes) to check metrics. Current
  Copilot Extensions agents are short-lived (run once and exit).
  Solutions:
    a. Event-driven chaining — each stage completion triggers the next
    b. GitHub Actions cron job that invokes the agent periodically
    c. External orchestrator (e.g., LaunchDarkly's own workflow engine)
```

### Agent 5: Flag Cleanup Agent

```
File: agents/flag-cleanup.agent.md

TRIGGER: Release guardian confirms stable 100% rollout + bake period

WHAT IT DOES:
  1. WAITS for a bake period (e.g., 7 days at 100%)
  2. VERIFIES metrics are still healthy via Metrics MCP
  3. IDENTIFIES code that uses the flag:
     - Searches codebase for flag key references
     - Finds flag evaluation calls (useFlags(), variation(), etc.)
  4. REMOVES the flag from the code:
     BEFORE:
       import { useFlags } from 'launchdarkly-react-client-sdk';
       function CheckoutV2() {
         const { newCheckoutFlow } = useFlags();
         if (!newCheckoutFlow) return <CheckoutV1 />;
         return <div>New checkout</div>;
       }
     AFTER:
       function CheckoutV2() {
         return <div>New checkout</div>;
       }
     - Removes the old code path (CheckoutV1 reference)
     - Removes unused imports
     - Removes the flag variable
  5. Opens a cleanup PR with the changes
  6. ARCHIVES the flag in LaunchDarkly via MCP
  7. Posts summary:
     "Flag `new-checkout-flow` retired after 7-day bake.
      Cleanup PR: #57
      Old code path (CheckoutV1) removed.
      Flag archived in LaunchDarkly."

SKILLS USED: list-flags (existing), retire-flag (new)

MCP SERVERS: LaunchDarkly (existing)

TOOLS: ["view", "edit"] — reads and modifies code
```

---

## Complete Architecture

```
+----------------------------------------------------------------------+
|                    THE FAIRYTALE PIPELINE                              |
|                                                                      |
|  +----------+   +-----------+   +-------------+   +---------------+  |
|  |  Devin / |   | Planner   |   | Main Agent  |   | Code Wrapper  |  |
|  |  Copilot |-->| Agent     |-->| (existing)  |-->| Agent (new)   |  |
|  |  (PR)    |   | (new)     |   |             |   |               |  |
|  +----------+   +-----------+   +-------------+   +---------------+  |
|                      |               |                    |          |
|                      |          +----+------+             |          |
|                      |          | Flag      |             |          |
|                      |          | Creator   |             |          |
|                      |          | (existing)|             |          |
|                      |          +-----------+             |          |
|                      |                                    |          |
|                      v                                    v          |
|                +-----------+                    +-----------------+  |
|                | Deploy    |<---PR merged--------| Code committed |  |
|                | Agent     |                    | to PR branch   |  |
|                | (new)     |                    +-----------------+  |
|                +-----+-----+                                        |
|                      |                                               |
|                      v                                               |
|                +----------------+                                    |
|                | Release        |--> 10% --> 50% --> 100%           |
|                | Guardian       |                                    |
|                | (new)          |--> Rollback if metrics regress     |
|                +--------+-------+                                    |
|                         |                                            |
|                         v                                            |
|                +----------------+                                    |
|                | Flag Cleanup   |--> Remove flag from code           |
|                | Agent (new)    |--> Archive flag in LaunchDarkly    |
|                +----------------+--> Open cleanup PR                 |
|                                                                      |
+----------------------------------------------------------------------+

MCP SERVERS:
  [LaunchDarkly] ---- flags, targeting, rollout (EXISTING)
  [Metrics]      ---- error rates, latency     (NEW - Datadog/New Relic)
  [Deployment]   ---- trigger deploys           (NEW - Vercel/AWS/Actions)
  [Slack]        ---- notifications             (OPTIONAL)
```

---

## Implementation Phases

### Phase 4: Planner Agent (Risk Assessment + Routing)

| Item | Details |
|---|---|
| **New files** | `agents/planner.agent.md`, `skills/check-compliance/SKILL.md` |
| **MCP servers** | LaunchDarkly (existing) |
| **Effort** | Medium |
| **Dependencies** | None — can be built on top of existing agent |
| **Deliverable** | PR analysis with risk level + rollout plan posted as comment |

### Phase 5: Code Wrapper Agent

| Item | Details |
|---|---|
| **New files** | `agents/code-wrapper.agent.md`, `skills/wrap-code-in-flag/SKILL.md` |
| **MCP servers** | LaunchDarkly (existing) |
| **Effort** | High — must understand React, Node.js, Python, Go patterns |
| **Dependencies** | Phase 4 (planner provides context), existing flag creation |
| **Deliverable** | Agent modifies source code to add flag evaluations + SDK imports |

### Phase 6: Deploy Agent

| Item | Details |
|---|---|
| **New files** | `agents/deploy.agent.md` |
| **MCP servers** | LaunchDarkly (existing) + Deployment MCP (new) |
| **Effort** | Medium — depends on deployment target (Vercel, AWS, Actions) |
| **Dependencies** | Phase 5 (code must be wrapped before deploy) |
| **Deliverable** | Automated deployment trigger + health verification |

### Phase 7: Release Guardian Agent

| Item | Details |
|---|---|
| **New files** | `agents/release-guardian.agent.md`, `skills/monitor-metrics/SKILL.md`, `skills/advance-rollout/SKILL.md`, `skills/rollback-flag/SKILL.md` |
| **MCP servers** | LaunchDarkly (existing) + Metrics MCP (new) |
| **Effort** | High — long-running process, metrics integration, rollback logic |
| **Dependencies** | Phase 6 (deploy must complete), Metrics MCP server available |
| **Deliverable** | Progressive rollout with automated metrics monitoring and rollback |

### Phase 8: Flag Cleanup Agent

| Item | Details |
|---|---|
| **New files** | `agents/flag-cleanup.agent.md`, `skills/retire-flag/SKILL.md` |
| **MCP servers** | LaunchDarkly (existing) |
| **Effort** | Medium — reverse of code-wrapper, flag archival |
| **Dependencies** | Phase 7 (release must be fully rolled out) |
| **Deliverable** | Automated flag removal from code + cleanup PR + flag archival |

---

## Biggest Technical Challenges

### 1. Code Modification (Phases 5 + 8)

The code-wrapper and flag-cleanup agents need to understand multiple frameworks and languages to correctly add and remove flag evaluations. This is the most complex part — the LLM must:
- Detect the framework (React, Node, Python, Go)
- Know the correct SDK import and API for each
- Handle edge cases (already has SDK, multiple flags, nested components)
- Remove code cleanly during cleanup (without breaking anything)

### 2. Long-Running Orchestration (Phase 7)

Current Copilot Extensions agents are short-lived — they run once and exit. The Fairytale Pipeline is a multi-day workflow:
- Deploy → wait → 10% for 1 hour → check → 50% for 4 hours → check → 100% for 24 hours → bake for 7 days → cleanup

Solutions:
- **Event-driven chaining**: Each stage completion triggers the next agent run
- **GitHub Actions cron**: Periodic job that invokes the release guardian to check metrics
- **External orchestrator**: LaunchDarkly's own workflow engine or a separate scheduler

### 3. Metrics Integration (Phase 7)

The release guardian needs real-time metrics. This requires a Metrics MCP server that can:
- Connect to the team's monitoring tool (Datadog, New Relic, CloudWatch, Prometheus)
- Query error rates, latency percentiles, and custom metrics
- Compare against thresholds defined in the rollout plan

This is an external dependency — someone needs to build or provide the Metrics MCP server.

### 4. Deployment Triggering (Phase 6)

Different teams deploy differently. The deploy agent needs to support:
- GitHub Actions workflows (most common)
- GitOps (merge to deploy branch)
- Direct API calls (Vercel, AWS, GCP)

This likely needs a configurable deployment strategy per repository.

### 5. Automated Rollback Speed (Phase 7)

If metrics regress during a rollout, the flag must be turned OFF immediately. This requires:
- Near-real-time metrics monitoring (not 15-minute polling)
- Fast MCP tool execution (turn flag OFF in seconds)
- Reliable notification (team must know about the rollback)

---

## Summary: What We Have vs What We Need

| Component | Status | Phase |
|---|---|---|
| Main agent (analyze + create flag) | DONE | Phases 1-3 |
| Flag creator (create via MCP) | DONE | Phases 1-3 |
| Skills (create, list, suggest) | DONE | Phases 1-3 |
| OIDC authentication | DONE | Phases 1-3 |
| Planner agent (risk + routing) | NEW | Phase 4 |
| Code wrapper agent (modify code) | NEW | Phase 5 |
| Deploy agent (trigger deploy) | NEW | Phase 6 |
| Release guardian (progressive rollout) | NEW | Phase 7 |
| Flag cleanup agent (retire flag) | NEW | Phase 8 |
| Metrics MCP server | NEW (external) | Phase 7 |
| Deployment MCP server | NEW (external) | Phase 6 |

**Current coverage: Steps 2-3 (partially) out of 6 steps in the Fairytale Pipeline.**

**To complete the full pipeline: 4 new agents, 7 new skills, 2 new MCP integrations.**
