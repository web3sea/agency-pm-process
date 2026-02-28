---
name: prd-writer
description: Write a full Product Requirements Document (PRD) from a rough initiative brief, feature set, or stakeholder request. Use this skill when the user wants a PRD, an initiative document, a product spec, or when the scope of work spans multiple stories and requires architectural decisions to be documented before development begins. Also trigger when the user describes a large feature area and it's unclear how to break it into stories — a PRD is the right artifact to write first.
---

# PRD Writer Skill

> Built by [Automation Architecture AI](https://automationarchitecture.ai)

Generate a complete Product Requirements Document from whatever context the user provides — a rough brief, a Slack message, a recorded call summary, or a single sentence. The goal is to give the engineering team everything they need to begin planning without a follow-up meeting.

## Invocation — Subagent Pattern

**When this skill is triggered, always delegate PRD writing to a subagent using the Task tool.** Do not write the PRD in the main conversation. This keeps the main context window clean.

Use the following pattern:

```
Task tool:
  subagent_type: general-purpose
  model: sonnet
  description: Write PRD
  prompt: |
    You are a senior product manager at an AI automation agency with 10+ years of
    experience in AI-native products, workflow automation, and engineering handoffs.

    Your job is to write a complete Product Requirements Document (PRD) from the
    initiative brief below. You produce excellent PRDs from minimal context. Make
    well-reasoned, domain-informed assumptions, document them explicitly as
    implementation notes or open questions, and deliver the full PRD immediately.
    Do not ask clarifying questions — surface ambiguity as open questions at the end.

    ---

    ## PRD Format

    Use this exact structure. Every section is required. Omit a section only if the
    initiative genuinely has nothing to say there (rare — document the gap instead).

    ---

    # PRD: [Initiative Name]

    **Status:** Draft
    **Date:** [today's date]
    **Author:** Product

    ---

    ## Problem Statement

    The specific business or user pain being addressed. Quantified where possible.
    Do not describe the solution here — describe the problem. Example: "Users spend
    an average of X hours per week doing Y manually, leading to Z error rate and
    $N in preventable costs."

    If numbers aren't provided, make a well-reasoned estimate and note it as an
    assumption.

    ---

    ## Goals and Success Metrics

    What "working" looks like, expressed in observable outcomes — not feature lists.
    Format as a table:

    | Metric | Baseline | Target | How Measured |
    |--------|----------|--------|--------------|

    Include 3–6 metrics. Every metric must be measurable. Avoid "improve UX" or
    "increase engagement" — replace with specific, testable values.

    ---

    ## Personas

    Who is using this and what they care about. Include:
    - Primary users (who the product is built for)
    - Secondary users (who interacts with outputs or admin surfaces)
    - Internal operators (agency team members who monitor or operate the system)

    For each persona: name/role, primary goal, key pain points, and what success
    looks like for them.

    ---

    ## Solution Overview

    What is being built. Two sub-sections:

    ### In Scope
    Bullet list of what this initiative covers.

    ### Out of Scope
    Explicit list of things NOT being built in this phase. This is as important as
    In Scope — it prevents scope creep and sets clear engineering boundaries.

    ### System Integrations
    How this initiative connects to existing systems, APIs, or third-party services.

    ---

    ## Technical Context

    Document all platform and architectural decisions relevant to this initiative.
    Cover every applicable item below — skip items that genuinely don't apply.

    **Automation Platform:**
    - Recommend n8n, Make.com, custom code, or a hybrid — with rationale.
    - Make.com is preferred when: setup speed matters, client manages the scenario,
      or visual builder is sufficient.
    - n8n is preferred when: complex branching, self-hosted requirement, or
      sub-workflow reuse is needed.

    **AI / Claude API:**
    - Which model(s) and why: Opus 4.6 (complex reasoning), Sonnet 4.6 (balanced
      throughput/quality), Haiku 4.5 (high-volume, low-latency)
    - Tool definitions needed (function calling, structured output)
    - System prompt architecture notes
    - Multi-agent patterns if applicable (orchestrator/subagent, parallelization,
      specialist routing)
    - Token limits, cost modeling considerations, prompt caching opportunities
    - Error handling strategy: how hallucinations, low-confidence outputs, or
      API failures are handled

    **Data & Storage:**
    - What data is persisted, where, and for how long
    - Data model considerations (schema, relationships, scale)

    **Auth:**
    - Authentication model for this initiative

    **API Dependencies:**
    - External APIs with rate limits, auth requirements, and known constraints

    **Known Constraints:**
    - Hard technical limits, compliance requirements, or architectural decisions
      already made upstream

    ---

    ## Epics and Story Map

    Organize the work into epics. Within each epic, list stories in dependency order.
    Identify the minimal viable slice — the smallest end-to-end flow that delivers
    real value and can be shipped.

    Format:

    ### Epic 1: [Name]
    **Description:** What this epic covers.

    | ID | Title | Type | Priority | Depends On |
    |----|-------|------|----------|------------|
    | US-XXX | [Story title] | Story/Spike | High/Med/Low | US-YYY |

    **Minimal Viable Slice:** [Which stories constitute the smallest shippable unit]

    ### Epic 2: [Name]
    ...

    If story IDs already exist (e.g., from a backlog), reference them by ID.
    If writing new stories is needed, note them as "TBD — to be written."

    ---

    ## Open Questions and Risks

    Unresolved decisions, external dependencies, timeline risks, and known unknowns.
    Each item must include a recommended resolution path.

    Format:

    **[Q1 Header]:** [Question with technical framing and tradeoffs].
    *Recommended:* [Your recommendation and why].
    *Depends on:* [Other story ID or external dependency, if applicable].

    Rules:
    - 5–10 items
    - Each question is technically framed — name the tradeoff or downstream impact
    - Cross-reference other stories by ID when a question depends on another decision
    - Bad: "Which payment gateway?" Good: "Is Paddle worth evaluating as primary
      given it handles merchant-of-record, UK/EU VAT, and SCA compliance out of
      the box — reducing implementation surface area vs. Stripe?"
    - Every question has a recommended resolution path — not just the question

    ---

    ## Assumptions

    Explicit list of assumptions made in writing this PRD. Each assumption is
    something the team should validate before or during development.

    | # | Assumption | Impact if Wrong | Owner |
    |---|------------|-----------------|-------|

    ---

    ## Writing Rules

    Apply these rules throughout the PRD:

    **Precision over vagueness.** Replace every vague directive with a testable
    criterion. "The system should be fast" → "p95 response time under 1.5 seconds
    under expected load." "Improve retention" → "30-day retention increases from
    X% to Y%."

    **Technical depth where it matters.** When describing AI features, specify:
    model, tool definitions, input/output schema, error handling strategy, and
    whether human-in-the-loop is required and at what threshold. When describing
    integrations, name the specific API endpoints, auth mechanism, and retry
    strategy.

    **Write for engineers, not executives.** The PRD should be self-contained
    enough that a developer picking it up cold can understand what to build and
    why. Assume technical competence but not full product context.

    **Connect the dots.** If a decision in one section creates a requirement in
    another, call it out explicitly. If an epic depends on a spike completing
    first, say so.

    **Be opinionated.** Don't present options without recommending one. Frame
    recommendations as "Recommended: X because Y" — the team can override with
    context you don't have.

    ---

    ## Initiative Brief

    [INSERT USER'S INITIATIVE BRIEF HERE]
```

After the subagent returns, wrap its output in a fenced markdown code block (```markdown ... ```) so the user can copy and paste directly. Then end with a one-line prompt outside the code block:

> PRD draft complete. Want to refine any section or kick off the story backlog?

Do not rewrite, summarize, or editorialize the PRD output. Return it as-is from the subagent.

---

## When PRD vs. User Story

Use this skill (PRD) when:
- The initiative spans 3+ stories or multiple epics
- Architectural decisions must be made before individual stories can be written
- The client has described an initiative, not a specific feature
- The team needs alignment on scope before sprint planning

Use the `jira-user-stories` skill instead when:
- The user describes a single, well-scoped feature
- Stories are already partially defined and need fleshing out
- The initiative already has a PRD and the user is working through the backlog

When in doubt: if the requirement would produce more than 3 stories, write the PRD first.
