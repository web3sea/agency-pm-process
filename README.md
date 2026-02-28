# Agency PM Process

Structured discovery process for an AI automation agency. Converts client needs into Jira-ready engineering artifacts — fast, specific, and factual.

Used for every project type: no-code (n8n, Make.com), low-code (hybrid automation + Claude agents), and full web app (custom product builds).

---

## What This Is

This repo is the operating system for how we run product discovery. When a client need comes in — a Slack message, a call summary, a rough brief — this process produces the artifacts the dev team needs to scope, estimate, and commit to a timeline. No follow-up meetings required.

All output goes directly into Jira (`https://automationarchitecture.atlassian.net`). There is no parallel documentation system.

---

## What's Included

| File | Purpose |
|------|---------|
| `CLAUDE.md` | PM agent identity, operating principles, agency context, domain expertise, and Jira integration config. Load this into Claude Code to activate the PM agent. |
| `SKILL.md` | User story format and skill definition. Produces Jira-ready stories with full acceptance criteria (Given/When/Then), failure states, and open questions. |
| `SKILL-prd.md` | PRD format and skill definition. Used for initiatives spanning 3+ stories or requiring architectural decisions before stories can be written. |
| `SKILL-backlog-import.md` | CSV backlog importer skill. Reads a backlog CSV, classifies story completeness, and formats full stories — complete rows directly, thin rows via subagent. |
| `user-stories-backlog.csv` | Backlog CSV schema and story data. |

---

## How to Use

### Setup

1. Clone this repo into your working directory
2. Open with [Claude Code](https://claude.ai/claude-code) — `CLAUDE.md` is automatically loaded as the system prompt
3. Ensure the `mcp-atlassian` MCP server is configured in `~/.claude.json` for Jira access

### Starting a New Project

Tell the PM agent:
- What the client needs (any format — rough is fine)
- Delivery type: no-code, low-code, or full app
- Jira project key

The agent will produce a first draft immediately, document assumptions as open questions, and offer to push directly to Jira.

### Skills

Invoke skills by describing what you need:

- **"Write a user story for..."** → triggers `jira-user-stories` skill
- **"Write a PRD for..."** → triggers `prd-writer` skill
- **"Process the backlog"** → triggers `backlog-csv-import` skill

---

## Artifact Standards

- **Acceptance criteria:** Given/When/Then, grouped by semantic concern, failure states in every section
- **Numbers:** Always concrete — "15 minutes," "7 days," "80% confidence" — never vague
- **Open questions:** Technically framed with tradeoffs, cross-referenced by story ID
- **Event tracking:** Always specifies exact fields (user ID, timestamp, method, etc.)
- **Operational stories:** Always includes Admin Visibility section
- **Webhook/integration stories:** Always includes idempotency criterion

---

## Dev Team Agreement

Stories and PRDs are the agreement boundary. The dev team reviews artifacts and commits to a timeline based on them. Underspecified stories block timeline agreement and return to discovery. This keeps scope clear before a single line of code is written.
