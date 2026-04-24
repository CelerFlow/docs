# Docs rewrite — repositioning CelerFlow as a workflow-automation product

**Date:** 2026-04-24
**Status:** Approved IA, pending final spec review
**Source of truth:** `celerflow` repo, branch `preview` (HEAD `a4ecbf81` at time of writing)
**Spec location:** `drafts/` is in `.mintignore`, so this file is committed but not published.

## Why this rewrite

The current docs describe an "agent governance platform" built around OpenClaw plugins, tracing, service-level permissions, kill switches, and risk alerts. That product has been removed from the codebase. The live product on `preview` is a chat-driven workflow-automation tool: a Planner turns natural-language intent into a validated workflow DAG, runs it on Trigger.dev, and surfaces runs back in the dashboard. Connections to Gmail, Notion, HubSpot, Slack, and Firecrawl are team-scoped via Composio.

Every existing docs page either describes the dead product or needs substantial rewriting. This is a from-scratch repositioning, not a patch.

## New positioning

> **CelerFlow** — chat-driven workflow automation. Describe what you want, connect your tools, ship it.

The `index.mdx` hero, quickstart framing, and every reference to "governance / observability / kill switch" in the current docs go away. New pillars:

1. **Build in chat** — the Planner drafts the workflow from a conversation.
2. **Connect your tools** — team-scoped OAuth for Gmail, Notion, HubSpot, Slack, Firecrawl.
3. **Approve what matters** — HITL pauses for OAuth gates, Notion target selection, and confirmations.
4. **Run and rerun** — workflow detail page shows runs, steps, result markdown, rerun.

## Target information architecture

**Tab: Documentation**

```
Getting started
  index                          (Introduction)
  quickstart

Core concepts
  concepts/workspaces
  concepts/workflows
  concepts/planner
  concepts/platform-agents
  concepts/connections
  concepts/runs
  concepts/approvals

Integrations
  integration/gmail
  integration/notion
  integration/hubspot
  integration/slack
  integration/firecrawl

Dashboard guide
  dashboard/building-a-workflow
  dashboard/executions        (list + detail merged)
  dashboard/settings

Platform
  platform/billing
  platform/teams-rbac
  platform/audit-log
  platform/roadmap
```

**Tab: API reference**

```
api-reference/introduction     (single placeholder page)
```

## Page-by-page spec

### Getting started

**`index.mdx` — Introduction (rewrite)**
Hero restates the new positioning verbatim. A 4-card CardGroup (Build / Connect / Approve / Run) linking to the matching concept pages. A "How it works" Steps block in 4 steps mirroring the quickstart (Sign in → Connect a tool → Describe the workflow → Deploy). Drop the "Why not just observability" table entirely. Pull any signature phrasing from `frontend/src/app/landing-v2.tsx` so the docs and the marketing site don't contradict each other.

**`quickstart.mdx` — Quickstart (rewrite)**
Five-minute path, no CLI. Steps: 1) sign in at celerflow.vercel.app, 2) create a workspace via the onboarding flow (name + use-case), 3) connect Gmail from Settings → Connections, 4) open New workflow, describe the task in chat (example: "When a new inbound lead email arrives, summarize it and file it to my Leads Notion database"), 5) confirm the draft and deploy, 6) watch it run on the workflow detail page. Prerequisites block: a CelerFlow account, a Gmail account willing to OAuth, Node/CLI NOT required.

### Core concepts (7 pages)

**`concepts/workspaces.mdx` — Workspaces (rewrite of existing page)**
Define workspace = `agent_fleets` row with a slug and a team behind it. Every user goes through onboarding to create one. Membership is team-scoped via Supabase auth. Default platform agents are seeded into `team_pool_agents` on workspace creation. Routing uses `[slug]` from the URL. Mention that a user can belong to multiple workspaces (team members) and that switching happens from the workspace picker.

**`concepts/workflows.mdx` — Workflows (new)**
Define a workflow = a named DAG of steps, owned by a team, with a lifecycle: `draft → ready → deploying → active → (running | completed | failed)`. Each step binds inputs from `previous_step` or `workflow_input` and produces an `output_key`. A workflow has a `workflow_id` that is pinned in the planner session URL so redeploys upgrade in place rather than creating duplicates (referencing PR #110/#111 behavior). Link out to Planner and Runs for the two lifecycle halves.

**`concepts/planner.mdx` — Planner (new)**
Explain the chat-driven builder in terms of the five-phase order from the planner system prompt: clarify goal and destination → discover pool agents → draft the plan → gate on OAuth connections → deploy. The planner is a LangGraph pipeline with four nodes (intent → match → assemble → validate). When the user asks to deploy, the frontend calls `generateWorkflowPlan` then `deployWorkflow`. Call out that the planner will pause the conversation for human input at known HITL points (see Approvals). Also note the re-deploy behavior: reusing an existing `workflow_id` replaces the draft in place.

**`concepts/platform-agents.mdx` — Platform agents (new; replaces old agents.mdx)**
Capability layer. Lists the seven default agents from `backend/railway-flow/src/routes/agent-pool/defaults.ts`, each with one-line purpose + what it consumes + what it produces + any attached tools or skills:

- **Web Search** — scouts the web; uses Composio Firecrawl search; skill `source-evaluation`.
- **Summarize** — condenses content; no tools.
- **Extract** — pulls structured fields from content.
- **Archive** — emits or files the final result; uses Composio Gmail send and file_writer paths.
- **Analyze** — produces structured markdown synthesis; skill `analyze-style`.
- **Translate** — translates content; skill `translate-style`.
- **File Writer** — writes to platform storage, Obsidian, or S3; tool `obsidian-formatter`.

Explain that agents are seeded per team on workspace creation and picked by the planner during assembly. Users don't configure agents directly in v0; this is a capability pool, not a tool palette.

**`concepts/connections.mdx` — Connections (new)**
Connections are team-scoped OAuth grants to external providers, powered by Composio. Scoping: stored in `team_connections`; the Composio entity is `celerflow-${userId}` so dedup works, but membership is team-level. Flow: user clicks Connect in Settings → backend starts a Composio connected account → popup OAuth → frontend polls for status (backoff 2s/3s/5s/8s/12s up to 30s). Handles stale/deleted accounts: a 404 from Composio collapses the UI to `missing`, while transient non-404 failures do not silently disconnect. Link to each provider's integration page.

**`concepts/runs.mdx` — Runs & executions (new; replaces tracing.mdx)**
A run = one execution of a workflow. Runs are orchestrated by Trigger.dev (task `celerflow-workflow-run`). Each run has ordered steps with `pending → running → completed | failed`. Status updates flow back via the webhook route, persist to `workflow_runs` and `workflow_steps`, and push to the UI via Supabase Realtime (workflow-sidebar channel has a unique suffix per subscriber — PR #105). Runs carry a `result_markdown` final output. Rerun queues a new `workflow_commands` deploy record. If the Trigger.dev dispatch fails, the UI shows a `dispatch_failed` banner instead of a generic 500.

**`concepts/approvals.mdx` — Approvals (new; replaces human-approval.mdx)**
HITL pause points in the current product:

- **OAuth gate** — planner pauses when a required provider isn't connected; a connection card appears in the chat.
- **Notion target selection** — before deploy, planner pauses to ask which Notion database/page to write into; placeholder target IDs are rejected.
- **Team confirmations** — realtime-broadcast via Supabase channel `team:${teamId}:confirmations`; the dashboard banner surfaces pending approvals and resolved events.

Explain that approvals are scoped to the team and show up for any member with the right role (see Teams & roles).

### Integrations (5 pages, each ~80 words)

Each integration page uses the same short template:
- One-line what this connection lets workflows do
- OAuth step pointer ("Settings → Connections → Connect X")
- Scopes / permissions at a high level
- Known limitations or coming-soon notes

**`integration/gmail.mdx`** — send and read email, attachments, threading; covers the archiver / send path used by Archive agent.
**`integration/notion.mdx`** — database and page targets; flag the HITL target-selection step and the placeholder-ID guard (see Approvals).
**`integration/hubspot.mdx`** — CRM actions available through Composio; portal selection at connect time.
**`integration/slack.mdx`** — channel posting. Mark as **Beta**: the codebase still has a `slack_coming_soon` UX path, and the old `slack_not_connected` copy flows through it.
**`integration/firecrawl.mdx`** — web search / scouting. Make clear it is used by the Web Search agent as a search source, not as a general scraper surface in v0.

### Dashboard guide (3 pages)

**`dashboard/building-a-workflow.mdx` — Building a workflow (new)**
A walkthrough of the New workflow chat screen. Covers: how to phrase intent (trigger + action + destination), what the draft canvas shows, how to answer clarifying questions from the planner, how to respond to OAuth prompts, how to resolve Notion target selection, and how the "Ready to deploy?" step works. Include one worked example end-to-end. Mention that deploying the same draft again upgrades the existing workflow in place.

**`dashboard/executions.mdx` — Executions (new; merges list + detail)**
First half: the Executions list and the dashboard home workflow list. Status badges and their meanings (draft, ready, running, completed, failed). Sorting and recent-runs context. Second half: the workflow detail page. Run history (last 20, newest first), step-level logs with duration and errors, result markdown rendering, rerun button, dispatch-failed banner, and how realtime updates behave while you're watching a run.

**`dashboard/settings.mdx` — Settings (new)**
Index page linking to each settings sub-surface actually present in the UI: Connections, Team, API keys, Billing, Audit log, Notifications, Appearance. One-line per sub-surface. Deep-link each to either its concept page (Connections → concepts/connections) or platform page (Billing → platform/billing).

### Platform (4 pages)

**`platform/billing.mdx` — Billing & usage (rewrite)**
Execution-count quota model, usage metrics shown in Settings → Billing, upgrade path. Confirm wording against the live Billing view before finalizing.

**`platform/teams-rbac.mdx` — Teams & roles (rewrite)**
Member list, invite flow, role tiers. During writing, read the roles actually shipped in `team_members` before locking in tier names and permission matrix. Do not invent roles that don't exist in code.

**`platform/audit-log.mdx` — Audit log (rewrite, renamed from audit-export.mdx)**
Renamed because the current code appears to surface an audit log view but not an export button. Confirm during writing. If export does exist, re-add the export section; if not, leave as read-only log and note export as a roadmap item.

**`platform/roadmap.mdx` — Roadmap (rewrite)**
Replace old roadmap entirely. Candidate items to verify with you before publishing: public REST API, more providers (Google Drive, GitHub, Linear?), scheduled triggers beyond Trigger.dev crons, self-hosted option, workflow templates marketplace. I will surface this list to you when I draft the page rather than guessing.

### API reference (1 page)

**`api-reference/introduction.mdx` — API (rewrite)**
Short page: "Public API is in development." Show how to generate a team API key in Settings → API keys so early adopters can pre-create keys. Note that `POST /workflow/trigger` and related endpoints are not stable yet. Kill the separate `authentication.mdx` and fold any content into this page. Remove every `api-reference/agent/*`, `api-reference/fleet/*`, `api-reference/trace/*` file.

## Content to delete

Pages:
- `concepts/service-permissions.mdx`
- `concepts/health-checks.mdx`
- `concepts/agent-score.mdx`
- `concepts/risk-alerts.mdx`
- `concepts/agents.mdx` (replaced by `concepts/platform-agents.mdx`)
- `concepts/tracing.mdx` (replaced by `concepts/runs.mdx`)
- `concepts/human-approval.mdx` (replaced by `concepts/approvals.mdx`)
- `integration/openclaw-plugin.mdx`
- `integration/mcp-gateway.mdx`
- `integration/cli.mdx`
- `integration/notifications.mdx` (folded into `dashboard/settings.mdx`)
- `dashboard/agents.mdx`
- `dashboard/tracing.mdx`
- `dashboard/workspace-overview.mdx` (merged into `dashboard/executions.mdx`)
- `api-reference/authentication.mdx`
- `api-reference/agent/*`, `api-reference/fleet/*`, `api-reference/trace/*` (every file)

Directories:
- `snippets/` — only contains `snippet-intro.mdx` with dead positioning copy; grep confirms zero references.

Images:
- Any screenshot under `images/` that shows the old governance / tracing / OpenClaw UI. I will flag specific files as TODOs in the PR and not ship new screenshots this round; screenshot refresh is a separate pass.

## Global config changes

- `docs.json` — rewrite navigation tree to match the IA above. Keep theme/colors/favicon/logo. Update `name` and any marketing blurb fields to match the new positioning.
- `index.mdx` — full rewrite (see Getting started).
- `quickstart.mdx` — full rewrite.

## Build sequence

1. Delete old pages, `snippets/`, old API reference (git rm).
2. Rewrite `docs.json` to the new tree.
3. Rewrite `index.mdx` and `quickstart.mdx`.
4. Write the 7 Core concepts pages in dependency order: workspaces → workflows → planner → platform-agents → connections → runs → approvals.
5. Write the 5 Integrations pages from the shared 80-word template.
6. Write the 3 Dashboard guide pages.
7. Write the 4 Platform pages, confirming roles / billing / audit copy against live code during authoring.
8. Write `api-reference/introduction.mdx`.
9. Sanity check: every path in `docs.json` resolves to an existing `.mdx`; `grep` for any dangling link to a deleted page.

## Source-of-truth guarantees

For any page, when live behavior is unclear I will go back to `/Users/glennlzsml/workplace/celerflow` (`preview` branch) and read the actual code before writing. I will not guess at role names, API surfaces, status strings, or provider capabilities. If something in code contradicts what's written here, code wins and I flag it to you.

## Explicitly out of scope for this rewrite

- Producing new screenshots or marketing imagery.
- Writing a real public API reference (waiting on API stabilization).
- Recipes / tutorials / example workflows beyond the one quickstart example and the one worked example in `building-a-workflow`.
- Any non-English version of the docs.
- Changes to `CONTRIBUTING.md`, `README.md`, `LICENSE`, or `favicon.ico`.

## Risks and open questions

- **Role names** in `platform/teams-rbac.mdx` depend on what's actually in `team_members` — I'll verify before writing.
- **Audit export** — if it does exist in code, I'll include it and keep the `audit-log.mdx` filename; only rename scope changes, not filename.
- **Slack status** — source code has both `slack_coming_soon` and `slack_not_connected` copy; I will label Slack as Beta and link to the roadmap rather than overpromising.
- **Marketing phrasing alignment** — the landing page (`landing-v2.tsx`) is the canonical product voice; if it diverges from what this spec proposes, landing page wins and I'll adjust the docs wording.
