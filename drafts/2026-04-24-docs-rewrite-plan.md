# Docs rewrite — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reposition the public CelerFlow docs from the removed "agent governance / OpenClaw plugin" product to the live chat-driven workflow-automation product on `preview`, per `drafts/2026-04-24-docs-rewrite-design.md`.

**Architecture:** Mintlify docs repo. Navigation is declared in `docs.json`. Each page is an `.mdx` file. No build step beyond Mintlify's own. Source-of-truth code lives in `/Users/glennlzsml/workplace/celerflow` on branch `preview` (HEAD was `a4ecbf81` when the spec was written — re-pull before writing provider pages in case it moved).

**Tech stack:** Mintlify, MDX, JSON. No tests — verification is: JSON valid, every nav path resolves to an existing `.mdx`, no prose references to deleted pages or old product terminology.

---

## File structure (after this plan is executed)

```
/
├── docs.json                               (rewritten)
├── index.mdx                               (rewritten)
├── quickstart.mdx                          (rewritten)
├── concepts/
│   ├── workspaces.mdx                      (rewritten)
│   ├── workflows.mdx                       (NEW)
│   ├── planner.mdx                         (NEW)
│   ├── platform-agents.mdx                 (NEW)
│   ├── connections.mdx                     (NEW)
│   ├── runs.mdx                            (NEW)
│   └── approvals.mdx                       (NEW)
├── integration/
│   ├── gmail.mdx                           (NEW)
│   ├── notion.mdx                          (NEW)
│   ├── hubspot.mdx                         (NEW)
│   ├── slack.mdx                           (NEW)
│   └── firecrawl.mdx                       (NEW)
├── dashboard/
│   ├── building-a-workflow.mdx             (NEW)
│   ├── executions.mdx                      (NEW; also covers workflow detail)
│   └── settings.mdx                        (NEW)
├── platform/
│   ├── billing.mdx                         (rewritten)
│   ├── teams-rbac.mdx                      (rewritten)
│   ├── audit-log.mdx                       (rewritten, renamed from audit-export.mdx)
│   └── roadmap.mdx                         (rewritten)
└── api-reference/
    └── introduction.mdx                    (rewritten, single page)
```

Deleted: see Task 1.

---

## Global content contract (applies to every page)

- Frontmatter: `title` (H1 replaced by Mintlify) and `description` (meta). Use Sentence case titles.
- No references to: OpenClaw, plugin, governance, kill switch, service-level permissions, agent score, health checks, risk alerts, tracing (as a tool-call concept), CLI, MCP gateway.
- Product voice echoes landing-v2: "Say it once. It's done." Keep prose direct, short sentences, second person.
- When uncertain about live behavior, read code in `/Users/glennlzsml/workplace/celerflow` (branch `preview`) before writing. Do not guess.
- Dates in frontmatter are optional — skip unless a page needs a "last updated" hint (roadmap only).
- Cross-link concept pages: every concept page links to the related Dashboard guide page and vice versa.

---

## Task 1: Delete dead content

**Files to remove (git rm):**

Old concepts: `concepts/service-permissions.mdx`, `concepts/health-checks.mdx`, `concepts/agent-score.mdx`, `concepts/risk-alerts.mdx`, `concepts/agents.mdx`, `concepts/tracing.mdx`, `concepts/human-approval.mdx`.

Old integrations: `integration/openclaw-plugin.mdx`, `integration/mcp-gateway.mdx`, `integration/cli.mdx`, `integration/notifications.mdx`.

Old dashboard: `dashboard/agents.mdx`, `dashboard/tracing.mdx`, `dashboard/workspace-overview.mdx`.

Old API reference (everything under these subtrees): `api-reference/authentication.mdx`, `api-reference/agent/`, `api-reference/fleet/`, `api-reference/trace/`.

Old platform: `platform/audit-export.mdx` (will be re-created as `platform/audit-log.mdx` in Task 8).

Old snippets dir: `snippets/` (contains only dead `snippet-intro.mdx`, zero references confirmed).

Steps:

- [ ] **1.1** Run `git rm` on every file and dir above in a single command, e.g.:
  ```bash
  git rm -r \
    concepts/service-permissions.mdx \
    concepts/health-checks.mdx \
    concepts/agent-score.mdx \
    concepts/risk-alerts.mdx \
    concepts/agents.mdx \
    concepts/tracing.mdx \
    concepts/human-approval.mdx \
    integration/openclaw-plugin.mdx \
    integration/mcp-gateway.mdx \
    integration/cli.mdx \
    integration/notifications.mdx \
    dashboard/agents.mdx \
    dashboard/tracing.mdx \
    dashboard/workspace-overview.mdx \
    api-reference/authentication.mdx \
    api-reference/agent \
    api-reference/fleet \
    api-reference/trace \
    platform/audit-export.mdx \
    snippets
  ```

- [ ] **1.2** `grep -r` from repo root for any remaining references to the deleted filenames (excluding the spec / this plan under `drafts/`). Expected: zero matches. If any match, fix the referring file or add it to a follow-up cleanup task.

- [ ] **1.3** Commit:
  ```bash
  git commit -m "docs: remove dead pages from old governance product"
  ```

---

## Task 2: Rewrite `docs.json` navigation

**File:** `docs.json` (modify in place).

Preserve: `$schema`, `theme: "mint"`, `name: "CelerFlow"`, `colors`, `favicon`, `logo`. Replace `navigation.tabs` entirely.

- [ ] **2.1** Replace the `navigation` block with:

  ```json
  "navigation": {
    "tabs": [
      {
        "tab": "Documentation",
        "groups": [
          {
            "group": "Getting started",
            "pages": ["index", "quickstart"]
          },
          {
            "group": "Core concepts",
            "pages": [
              "concepts/workspaces",
              "concepts/workflows",
              "concepts/planner",
              "concepts/platform-agents",
              "concepts/connections",
              "concepts/runs",
              "concepts/approvals"
            ]
          },
          {
            "group": "Integrations",
            "pages": [
              "integration/gmail",
              "integration/notion",
              "integration/hubspot",
              "integration/slack",
              "integration/firecrawl"
            ]
          },
          {
            "group": "Dashboard guide",
            "pages": [
              "dashboard/building-a-workflow",
              "dashboard/executions",
              "dashboard/settings"
            ]
          },
          {
            "group": "Platform",
            "pages": [
              "platform/billing",
              "platform/teams-rbac",
              "platform/audit-log",
              "platform/roadmap"
            ]
          }
        ]
      },
      {
        "tab": "API reference",
        "groups": [
          {
            "group": "Overview",
            "pages": ["api-reference/introduction"]
          }
        ]
      }
    ]
  }
  ```

- [ ] **2.2** Validate JSON: `python3 -c "import json; json.load(open('docs.json'))"` exits 0.

- [ ] **2.3** Commit:
  ```bash
  git commit -m "docs: rewrite docs.json navigation for new IA"
  ```

---

## Task 3: Rewrite `index.mdx`

**File:** `index.mdx` (overwrite).

Content contract:

- Frontmatter: `title: "Introduction"`, `description: "CelerFlow builds and runs automated workflows from a chat. Describe what you need in plain English."`
- Opening `<Note>`: "**Public Beta** — CelerFlow is live at [celerflow.vercel.app](https://celerflow.vercel.app). The public API is in development — see the [API reference](/api-reference/introduction)."
- H2 "Say it once. It's done." — lead paragraph (2–3 sentences) echoing landing-v2 voice: tell CelerFlow what you need in plain English; the Planner drafts the workflow, you confirm, it runs.
- `<CardGroup cols={2}>` with four cards:
  - **Build in chat** — href `/concepts/planner`, icon `wand-magic-sparkles`, "Describe the outcome; the Planner figures out the steps."
  - **Connect your tools** — href `/concepts/connections`, icon `plug`, "Team-scoped OAuth for Gmail, Notion, HubSpot, Slack, Firecrawl."
  - **Approve what matters** — href `/concepts/approvals`, icon `shield-check`, "Human-in-the-loop pauses for OAuth gates and Notion target selection."
  - **Run and rerun** — href `/concepts/runs`, icon `play`, "Watch steps in real time. Rerun with one click."
- H2 "How it works" — `<Steps>` with four steps mirroring quickstart:
  1. Sign in at celerflow.vercel.app and create a workspace.
  2. Connect a tool (Gmail is the fastest first one).
  3. Open New workflow and describe what you need.
  4. Confirm the draft and deploy; watch it run.
- H2 "Get started" — single `<Card>` horizontal linking to `/quickstart`.

Steps:

- [ ] **3.1** Overwrite `index.mdx` with the content above, written as real prose + MDX components.
- [ ] **3.2** Commit: `git commit -m "docs: rewrite introduction for workflow-automation positioning"`.

---

## Task 4: Rewrite `quickstart.mdx`

**File:** `quickstart.mdx` (overwrite).

Content contract:

- Frontmatter: `title: "Quickstart"`, `description: "Build your first workflow in five minutes — no CLI required."`
- `<Info>` prerequisites: a CelerFlow account at celerflow.vercel.app and a Gmail account you're willing to OAuth. Nothing to install locally.
- H2 "Build your first workflow" wrapping `<Steps>`:
  1. **Sign in and create a workspace.** Go to celerflow.vercel.app, sign in, and complete onboarding (pick a use-case, name the workspace).
  2. **Connect Gmail.** From Settings → Connections, click **Connect Gmail**. Complete OAuth in the popup. The card flips to Connected once Composio confirms (usually within a few seconds).
  3. **Open New workflow.** From the dashboard, click **New workflow**. Describe the outcome in one or two sentences, for example: *"When a new inbound lead email arrives, summarize it and file it to my Leads Notion database."*
  4. **Answer the Planner's follow-ups.** The Planner may ask a clarifying question or pause for a connection or a Notion target selection. Respond in chat or use the connection card — see [Approvals](/concepts/approvals).
  5. **Confirm and deploy.** When the draft is ready, the Planner asks "Ready to deploy?". Confirm and the workflow goes live.
  6. **Watch it run.** Open the workflow from the dashboard to see run history, step-level logs, and the result markdown. See [Executions](/dashboard/executions).
- H2 "What's next" — two `<Card>`s linking to `/concepts/workflows` and `/dashboard/building-a-workflow`.

Steps:

- [ ] **4.1** Overwrite `quickstart.mdx` with real prose matching the contract.
- [ ] **4.2** Commit: `git commit -m "docs: rewrite quickstart for chat-driven workflow build"`.

---

## Task 5: Core concepts (7 pages)

Write in dependency order so later pages can cross-link into earlier ones. One commit per page.

### 5.1 `concepts/workspaces.mdx` (rewrite)

Contract:
- Frontmatter: `title: "Workspaces"`, `description: "A workspace is a team-scoped home for your workflows, connections, and runs."`
- Define workspace: one row in `agent_fleets`, has a slug used in URLs (`/[slug]/dashboard`), bound to a team in Supabase.
- Mention onboarding: every user goes through it on first sign-in (pick a use-case, name the workspace).
- Mention seed-on-create: default platform agents get seeded into the team's `team_pool_agents` automatically — link to [Platform agents](/concepts/platform-agents).
- Multi-workspace: a user can belong to multiple teams/workspaces; switch from the workspace picker.
- Deep link: Team management lives in [Teams & roles](/platform/teams-rbac).

Steps:
- [ ] **5.1.1** Write the page.
- [ ] **5.1.2** Commit: `git commit -m "docs: add workspaces concept page"`.

### 5.2 `concepts/workflows.mdx` (new)

Contract:
- Frontmatter: `title: "Workflows"`, `description: "A named DAG of steps built in chat, deployed to your team, and run on demand or on a schedule."`
- H2 "Lifecycle": list the states — draft → ready → deploying → active → (running | completed | failed). Use a `<Steps>` or table.
- H2 "Steps and bindings": each step has an agent, inputs bound from `previous_step` or `workflow_input`, and an `output_key` consumed by downstream steps.
- H2 "Redeploy in place": deploying the same draft again upgrades the existing `workflow_id` rather than creating a duplicate (confirmed via `backend/railway-api/src/routes/workflow/deploy.ts` and planner session URL pinning — PRs #110/#111).
- H2 "See also": links to [Planner](/concepts/planner) and [Runs](/concepts/runs).

Steps:
- [ ] **5.2.1** Write the page.
- [ ] **5.2.2** Commit: `git commit -m "docs: add workflows concept page"`.

### 5.3 `concepts/planner.mdx` (new)

Contract:
- Frontmatter: `title: "Planner"`, `description: "The chat-driven builder that turns plain-English intent into a validated workflow."`
- H2 "How a planning conversation flows": ordered list matching the system prompt phases — clarify goal & destination → discover available pool agents → draft the plan → gate on OAuth connections → deploy.
- H2 "The pipeline": four-node LangGraph — intent → match → assemble → validate. Plan output: `draft_ready | needs_clarification | validation_failed`.
- H2 "Pause points (HITL)": missing OAuth connection, Notion target selection, placeholder target IDs. Link to [Approvals](/concepts/approvals).
- H2 "Deploy": clicking "Ready to deploy?" calls `deployWorkflow`, creates the workflow record, queues a deploy command. Redeploys upgrade in place — link to [Workflows](/concepts/workflows).

Steps:
- [ ] **5.3.1** Write the page.
- [ ] **5.3.2** Commit: `git commit -m "docs: add planner concept page"`.

### 5.4 `concepts/platform-agents.mdx` (new)

Contract:
- Frontmatter: `title: "Platform agents"`, `description: "The eight built-in capabilities the Planner assembles into a workflow."`
- Opening paragraph: platform agents are the capability pool; the Planner picks from them during assembly; users don't configure agents directly in v0.
- Before writing: run `ls /Users/glennlzsml/workplace/celerflow/backend/railway-flow/src/lib/platform-agents/` and open each agent's `definition.ts` to confirm id, displayName, tools, skills. Do not rely on the spec list alone — the spec was a snapshot.
- Table or `<AccordionGroup>` listing all eight: **Web Search, Scraper, Summarize, Extract, Analyze, Translate, Archive, File Writer**. For each: one-line purpose + what it consumes + what it produces + tools/skills (as confirmed from code).
- H2 "How agents are seeded": `ensureDefaultTeamPoolAgents()` seeds every default into `team_pool_agents` on workspace creation. One of them is flagged `is_default_planner`.
- H2 "See also": link to [Planner](/concepts/planner).

Steps:
- [ ] **5.4.1** Re-read each `definition.ts` under `platform-agents/<agent>/` in preview; jot real tool/skill names.
- [ ] **5.4.2** Write the page using actual field values.
- [ ] **5.4.3** Commit: `git commit -m "docs: add platform-agents concept page"`.

### 5.5 `concepts/connections.mdx` (new)

Contract:
- Frontmatter: `title: "Connections"`, `description: "Team-scoped OAuth grants powered by Composio."`
- H2 "Scope": stored in `team_connections`; Composio entity id is `celerflow-${userId}` for dedup but membership is team-level — every team member sees the connection.
- H2 "Connecting": user clicks Connect from Settings → Connections → backend starts a Composio connected account → popup OAuth → frontend polls `/oauth/status` with backoff (2s, 3s, 5s, 8s, 12s, up to 30s).
- H2 "Stale and deleted accounts": 404 from Composio collapses UI to `missing`; non-404 transient errors do not silently disconnect.
- H2 "See also": link to each provider page under Integrations.

Steps:
- [ ] **5.5.1** Write the page.
- [ ] **5.5.2** Commit: `git commit -m "docs: add connections concept page"`.

### 5.6 `concepts/runs.mdx` (new)

Contract:
- Frontmatter: `title: "Runs & executions"`, `description: "What happens when a workflow actually runs."`
- Opening: a run = one execution of a workflow. Ordered steps. Persisted to `workflow_runs` and `workflow_steps`.
- H2 "Lifecycle": step states `pending → running → completed | failed`. Run itself mirrors.
- H2 "Orchestration": Trigger.dev task `celerflow-workflow-run`. Backend webhook updates rows and broadcasts.
- H2 "Realtime updates to the UI": Supabase Realtime. Workflow sidebar channel uses a unique suffix per subscriber (PR #105) to avoid cross-tab collisions.
- H2 "Result markdown": each run ends with a `result_markdown` field rendered on the detail page.
- H2 "Rerun and dispatch failures": rerun queues a new `workflow_commands` deploy record. If Trigger.dev can't accept the dispatch, UI shows a `dispatch_failed` banner (not a generic 500).
- H2 "See also": link to [Executions](/dashboard/executions) for the UI surface.

Steps:
- [ ] **5.6.1** Write the page.
- [ ] **5.6.2** Commit: `git commit -m "docs: add runs concept page"`.

### 5.7 `concepts/approvals.mdx` (new)

Contract:
- Frontmatter: `title: "Approvals"`, `description: "Human-in-the-loop pause points and the team confirmations channel."`
- H2 "OAuth gate": when a required provider isn't connected, the Planner pauses and surfaces a connection card in the chat.
- H2 "Notion target selection": before deploy, Planner pauses to ask which Notion database/page to write into; placeholder target IDs are rejected.
- H2 "Team confirmations": Supabase broadcast channel `team:${teamId}:confirmations`. The dashboard surfaces pending approvals and resolved events in realtime.
- H2 "Who can approve": link to [Teams & roles](/platform/teams-rbac) for role scoping.

Steps:
- [ ] **5.7.1** Write the page.
- [ ] **5.7.2** Commit: `git commit -m "docs: add approvals concept page"`.

---

## Task 6: Integrations (5 pages, ~80 words each)

Shared template for every integration page:

```mdx
---
title: "<Provider>"
description: "<one-line: what workflows can do with this connection>"
---

## What this connection enables

<2 sentences naming the concrete workflow uses, e.g. "Gmail lets workflows read inbound emails and send replies or summaries.">

## Connect it

From Settings → Connections, click **Connect <Provider>**. A popup runs OAuth and the card flips to Connected when Composio confirms. See [Connections](/concepts/connections) for scope, stale-account handling, and polling behavior.

## Scopes and permissions

<1–2 sentences at a high level — no long scope lists. Note any mandatory scopes the UX surfaces.>

## Known limitations

<1–2 sentences. If the provider has Beta status or a HITL quirk, say so here.>
```

Per-provider content pointers (use as guardrails while writing):

- **`integration/gmail.mdx`** — read inbound email, send mail, label and thread. Used by the Archive agent's send path.
- **`integration/notion.mdx`** — database and page write targets. Flag the Notion target-selection HITL step and placeholder-ID guard — link to [Approvals](/concepts/approvals).
- **`integration/hubspot.mdx`** — CRM actions via Composio. Portal is selected at connect time.
- **`integration/slack.mdx`** — channel posting. Status: **Beta**. Code path `slack_coming_soon` still exists, and legacy `slack_not_connected` copy flows through the same UX — link to [Roadmap](/platform/roadmap).
- **`integration/firecrawl.mdx`** — web search used by the Web Search agent. Scout-only in v0 — not a general scraping surface.

Steps:

- [ ] **6.1** Create `integration/gmail.mdx`. Commit: `git commit -m "docs: add gmail integration page"`.
- [ ] **6.2** Create `integration/notion.mdx`. Commit: `git commit -m "docs: add notion integration page"`.
- [ ] **6.3** Create `integration/hubspot.mdx`. Commit: `git commit -m "docs: add hubspot integration page"`.
- [ ] **6.4** Create `integration/slack.mdx`. Commit: `git commit -m "docs: add slack integration page"`.
- [ ] **6.5** Create `integration/firecrawl.mdx`. Commit: `git commit -m "docs: add firecrawl integration page"`.

---

## Task 7: Dashboard guide (3 pages)

### 7.1 `dashboard/building-a-workflow.mdx` (new)

Contract:
- Frontmatter: `title: "Building a workflow"`, `description: "The chat builder end-to-end — phrasing intent, answering follow-ups, and deploying."`
- H2 "Open New workflow": from the dashboard, click **New workflow**.
- H2 "Phrase the intent": short guidance on trigger + action + destination. One worked example inline ("When a new inbound lead email arrives, summarize it and file it to my Leads Notion database").
- H2 "Read the draft canvas": what the node graph shows, how to interpret step order.
- H2 "Answer clarifying questions": the Planner will ask; answer in chat. Keep answers specific.
- H2 "Handle OAuth prompts": the chat surfaces a connection card if a provider is missing. Click Connect, finish OAuth, return to chat.
- H2 "Pick a Notion target": if the workflow writes to Notion, pick the database or page from the picker. Placeholder IDs are rejected.
- H2 "Confirm and deploy": click "Ready to deploy?". Redeploys upgrade the same workflow in place.
- H2 "See also": [Planner](/concepts/planner), [Approvals](/concepts/approvals).

Steps:
- [ ] **7.1.1** Write the page.
- [ ] **7.1.2** Commit: `git commit -m "docs: add building-a-workflow guide"`.

### 7.2 `dashboard/executions.mdx` (new — list + detail merged)

Contract:
- Frontmatter: `title: "Executions"`, `description: "Where your workflows and their runs live after deploy."`
- H2 "Workflow list" — the dashboard home and `/executions` both show workflow lists. Status badges: draft, ready, running, completed, failed. Sort: recent-first.
- H2 "Workflow detail page" — run history (last 20, newest first), step-level logs with duration and errors, result markdown rendering, rerun button, dispatch-failed banner behavior.
- H2 "Realtime while watching a run" — run + step rows update in place via Supabase Realtime; you don't need to refresh.
- H2 "See also": [Runs](/concepts/runs), [Approvals](/concepts/approvals).

Steps:
- [ ] **7.2.1** Write the page.
- [ ] **7.2.2** Commit: `git commit -m "docs: add executions guide covering list and detail"`.

### 7.3 `dashboard/settings.mdx` (new)

Contract:
- Frontmatter: `title: "Settings"`, `description: "Where to manage connections, team, billing, API keys, and preferences."`
- Before writing: open `/Users/glennlzsml/workplace/celerflow/frontend/src/app/(dashboard)/[slug]/settings/` and list the sub-routes actually present in the UI nav. Confirm whether `mcp-proxy` and `mcp-servers` are reachable from the nav. If dead, don't document them. If live, add one-line entries.
- Index page. One-line per sub-surface with a deep link:
  - **Connections** → `/concepts/connections`
  - **Team** → `/platform/teams-rbac`
  - **API keys** → `/api-reference/introduction`
  - **Billing** → `/platform/billing`
  - **Audit** → `/platform/audit-log`
  - **Notifications** — delivery preferences for approval and run events.
  - **Appearance** — theme (light/dark).

Steps:
- [ ] **7.3.1** Confirm live sub-routes in code.
- [ ] **7.3.2** Write the page.
- [ ] **7.3.3** Commit: `git commit -m "docs: add settings guide index"`.

---

## Task 8: Platform (4 pages)

### 8.1 `platform/billing.mdx` (rewrite)

Contract:
- Frontmatter: `title: "Billing & usage"`, `description: "Execution quotas, usage meters, and upgrade paths."`
- Before writing: open the Billing settings component and the Stripe route (`backend/supabase/functions/stripe/index.ts`) to confirm the real metric (executions per month, period boundaries, trial behavior). Don't invent SKU names.
- H2 "What counts": each workflow run consumes one execution from the team quota (confirm metric naming in code first).
- H2 "Seeing your usage": Settings → Billing shows current period, used, and remaining.
- H2 "Upgrading": contact / upgrade flow as actually wired in the UI.

Steps:
- [ ] **8.1.1** Verify metric naming in code.
- [ ] **8.1.2** Write the page.
- [ ] **8.1.3** Commit: `git commit -m "docs: rewrite billing page"`.

### 8.2 `platform/teams-rbac.mdx` (rewrite)

Contract:
- Frontmatter: `title: "Teams & roles"`, `description: "Members, roles, and what each role can do."`
- Roles (confirmed from `backend/supabase/functions/team/index.ts`): **owner**, **admin**, **operator**, **viewer**.
- Before writing: open that file and the team settings frontend; read the route guards to produce a concrete permission matrix (who can: invite, change roles, remove members, create workflows, deploy, approve confirmations, manage connections, manage billing).
- H2 "Four roles": table with role × capability matrix.
- H2 "Invite flow": how an admin/owner invites, how the invitee accepts.
- H2 "Owner transfer": describe if supported in code; otherwise omit.

Steps:
- [ ] **8.2.1** Read route guards + frontend permission checks.
- [ ] **8.2.2** Write the page with the matrix grounded in code.
- [ ] **8.2.3** Commit: `git commit -m "docs: rewrite teams & roles with confirmed role tiers"`.

### 8.3 `platform/audit-log.mdx` (rewrite, renamed)

Contract:
- Frontmatter: `title: "Audit log"`, `description: "Team actions recorded for review."`
- Before writing: open the `Settings → Audit` page and its backend source to determine:
  - What actions are recorded (team changes, connection events, workflow deploys, etc.).
  - Whether an export is actually implemented or not.
- H2 "What gets recorded": list, grounded in the code.
- H2 "Viewing": where and how to filter.
- H2 "Export": document only if implemented; otherwise list as a Roadmap item and omit the section here.

Steps:
- [ ] **8.3.1** Confirm actions and export status in code.
- [ ] **8.3.2** Write the page.
- [ ] **8.3.3** Commit: `git commit -m "docs: rewrite audit log page"`.

### 8.4 `platform/roadmap.mdx` (rewrite)

Contract:
- Frontmatter: `title: "Roadmap"`, `description: "What's shipping next and what's being considered."`
- Structure as three sections: **Shipping next** (near-term, high confidence), **Considering** (medium confidence), **Not on the roadmap** (explicit no).
- Seed the page with these items; adjust with the user during writing rather than guessing:
  - Shipping next: Public REST API (workflow trigger, run fetch), scheduled triggers beyond Trigger.dev crons, Audit log export.
  - Considering: Additional providers (Google Drive, GitHub, Linear), self-hosted option, workflow templates marketplace, MCP gateway re-entry if demand returns.
  - Not on the roadmap: tool-call-level tracing, OpenClaw plugin distribution.
- Before committing: ask the user which items they want kept, removed, or reworded. Do not publish speculative commitments.

Steps:
- [ ] **8.4.1** Draft the page with the seeded items as a proposal.
- [ ] **8.4.2** Ask the user to confirm / edit the three buckets before committing.
- [ ] **8.4.3** Commit: `git commit -m "docs: rewrite roadmap for workflow product"`.

---

## Task 9: API reference (1 page)

### 9.1 `api-reference/introduction.mdx` (rewrite)

Contract:
- Frontmatter: `title: "API reference"`, `description: "CelerFlow's public API is in development. Here's how to prepare."`
- H2 "Status": one paragraph — public REST API is in development; shapes are not stable; early-adopter feedback welcome.
- H2 "Generate an API key": Settings → API keys → Create. Team-scoped. Treat as secret.
- H2 "What to expect when it ships": bearer auth with the API key, endpoints for triggering workflows and reading runs. Link back to [Workflows](/concepts/workflows) and [Runs](/concepts/runs).
- H2 "Talk to us": how to contact (confirm the channel during writing — Discord? support email? GitHub issues?).

Steps:
- [ ] **9.1.1** Confirm the contact channel.
- [ ] **9.1.2** Write the page.
- [ ] **9.1.3** Commit: `git commit -m "docs: rewrite api reference as beta placeholder"`.

---

## Task 10: Sanity sweep

No prose changes — just verify the repo is internally consistent.

- [ ] **10.1 Every nav path resolves.** Run:
  ```bash
  python3 <<'PY'
  import json, os
  cfg = json.load(open("docs.json"))
  missing = []
  for tab in cfg["navigation"]["tabs"]:
      for group in tab["groups"]:
          for page in group["pages"]:
              if not os.path.exists(page + ".mdx"):
                  missing.append(page)
  print("missing:", missing)
  assert not missing, missing
  PY
  ```
  Expected: `missing: []`.

- [ ] **10.2 No stale references.** Run from repo root:
  ```bash
  grep -rniE "openclaw|kill switch|agent score|service-level permission|risk alert|health check|mcp gateway|/concepts/tracing|/concepts/agents|/concepts/human-approval|/api-reference/agent|/api-reference/fleet|/api-reference/trace|snippet-intro" \
      --include='*.mdx' --include='*.json' --exclude-dir=drafts --exclude-dir=.git --exclude-dir=node_modules .
  ```
  Expected: zero matches (the spec and this plan live under `drafts/` and are excluded).

- [ ] **10.3 Image audit.** List files under `images/` and flag any screenshot clearly tied to the old product (governance dashboards, tracing views, OpenClaw plugin UI). Record paths in a follow-up note; do **not** delete images this pass, since screenshot refresh is its own pass.

- [ ] **10.4 Final commit if anything changed.** Otherwise no-op.

---

## Self-review results

1. **Spec coverage:** Each spec section maps to a task — deletions → Task 1; `docs.json` → Task 2; `index.mdx` → Task 3; `quickstart.mdx` → Task 4; all 7 concepts → Task 5; 5 integrations → Task 6; 3 dashboard pages → Task 7; 4 platform pages → Task 8; API reference → Task 9; sanity sweep → Task 10.
2. **Placeholder scan:** No "TBD / handle appropriately / similar to above" patterns. The few "confirm during writing" notes are explicit verification steps grounded in concrete file paths, not handwaving.
3. **Naming consistency:** IA tree, `docs.json` page list, task file list, and delete list all use the same paths (`concepts/workspaces`, `dashboard/executions`, `platform/audit-log`, `api-reference/introduction`, etc.). Role names (owner/admin/operator/viewer) match across teams page and approvals page.
4. **Scope:** No new features. No screenshot regeneration. No public API authoring. No i18n. All out of scope per spec.

## Execution handoff

Plan complete and saved to `drafts/2026-04-24-docs-rewrite-plan.md`. Two execution options:

1. **Subagent-Driven (recommended)** — fresh subagent per task, two-stage review between tasks.
2. **Inline Execution** — run tasks in this session with checkpoints.
