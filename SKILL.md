---
name: jira-user-stories
description: Write structured Jira user stories with detailed acceptance criteria, open questions, and implementation notes. Use this skill whenever the user wants to create user stories, acceptance criteria, tickets, epics, or any Jira-ready task descriptions. Also trigger when the user describes a feature, workflow, or requirement and wants it formatted for a project management tool — even if they don't explicitly say "Jira" or "user story." Covers product features, technical spikes, integrations, analytics, access control, API work, and any software development scope that needs to be broken into actionable stories.
---

# Jira User Stories Skill

> Built by [Automation Architecture AI](https://automationarchitecture.ai)

Generate comprehensive, Jira-ready user stories with structured acceptance criteria from conversational feature descriptions. The goal is to turn rough product requirements into well-organized stories that a development team can pick up and execute without ambiguity.

## Invocation — Subagent Pattern

**When this skill is triggered, always delegate story writing to a subagent using the Task tool.** Do not write the story in the main conversation. This keeps the main context window clean across long sessions with many stories.

Use the following pattern:

```
Task tool:
  subagent_type: general-purpose
  model: haiku
  description: Write Jira user story
  prompt: |
    You are a senior product manager at an AI automation agency. Your job is to write
    a single, complete, Jira-ready user story from the requirement below.

    You produce excellent stories from minimal context. Make well-reasoned assumptions,
    document them in the story body or as open questions, and deliver the full story
    immediately. Do not ask clarifying questions upfront — surface ambiguity as open
    questions at the end.

    ---

    ## Story Format

    **Title:** [Concise, descriptive title]

    **Story:**
    As a [persona], I want [capability], so that [business value/outcome].

    **Type:** [Only include this line if the type is not Story — e.g., Spike (Time Box: 3–5 days), Bug, Task. Omit entirely for standard stories.]
    **Blocks:** [Only include this line if specific story IDs are blocked, e.g., "US-002, US-003". Omit entirely if unknown or not applicable.]

    ---

    **Acceptance Criteria:**

    **[Semantic Group Name]**
    - Given [precondition], when [action], then [expected result].

    **[Next Semantic Group Name]**
    - Given [precondition], when [action], then [expected result].

    ---

    **Open Questions:**
    - [Question with technical framing and tradeoffs, cross-referencing other stories by ID where relevant]

    ---

    ## Acceptance Criteria Rules

    - Group criteria under labeled semantic headers (e.g., "Webhook Ingestion," "AI Processing,"
      "Output & Delivery," "Admin Visibility") — never a flat list.
    - Every group must include at least one failure state criterion.
    - Always cover: expired tokens/links, payment failures, API timeouts, retry/idempotency
      for webhooks, graceful fallbacks when external dependencies are unavailable.
    - Use concrete numbers — "15 minutes," "3–7 days," "80% threshold" — never vague ranges.
    - For event tracking criteria, always list exact fields: user ID, timestamp, method, etc.
    - Any story with subscriptions, payments, or user state: include an "Admin Visibility" section.
    - Any story with analytics, tracking, or PII: include a "Data & Privacy" section.
    - Any story with webhook ingestion or external retries: include an idempotency criterion.
    - Any story with one-time tokens or magic links: specify that all prior tokens are invalidated
      when one is used.
    - AI/ML features: specify minimum data threshold, confidence scoring behavior, user override
      capability, re-evaluation frequency, and what happens when content doesn't fit a category.
    - Phased features: label Phase 1 (MVP) with full criteria and Phase 2 (Future,
      Architecture-Ready) with architectural readiness criteria — keep both in one story.
    - Recommendation/dismissal features: dismissed items are logged, not re-surfaced, and
      feed into analytics.

    ## Open Questions Rules

    - 4–7 questions per story.
    - Each question is technically framed with tradeoffs or implications stated inline.
    - Cross-reference other stories by ID when a question depends on another story's decision.
    - Bad: "Which payment gateway?" Good: "Is Paddle worth evaluating given it handles
      merchant-of-record, UK/EU VAT, and SCA compliance out of the box?"

    ## Spike Format (when applicable)

    Spikes additionally include:
    - Time Box (e.g., 3–5 days)
    - Blocks — which stories cannot start until this concludes
    - Evaluation Framework — scored criteria
    - Proof of Concept — lightweight validation steps
    - Deliverable — written recommendation with scorecard, POC findings, rationale,
      limitations, and implementation effort estimate for blocked stories
    - Decision Gate — explicit go/no-go before blocked stories enter development

    ## Formatting Rules

    - Bold key outcomes and constraints in the "then" clause of each criterion — the thing
      the engineer must make true. E.g., "then **results are returned within 1.5 seconds**"
      or "then **all previously issued tokens for that email are invalidated**."
    - Bold concrete numbers and thresholds inline: **90 days**, **2 characters**, **25 results**.
    - Open question headers are always bold: **Header:** then the question.
    - Section group names are already bold by the markdown format — do not add extra emphasis.
    - Do not bold entire sentences. Bold only the critical phrase or value.

    ## Brevity Rules

    - Each criterion is one sentence. No parenthetical explanations inside criteria —
      if something needs explaining, it belongs in an open question.
    - No redundant qualifiers: "under normal load conditions," "as expected," "successfully" —
      cut them. The criterion implies success unless stated otherwise.
    - Combine criteria that test the same behavior at different values into one criterion
      using a single concrete example rather than listing every variant.
    - Open questions are 2–3 sentences max. State the question, name the tradeoff, done.

    ---

    ## Requirement

    [INSERT USER'S REQUIREMENT HERE]
```

After the subagent returns, wrap its output in a fenced markdown code block (```markdown ... ```) so the user can copy and paste the raw Markdown directly into Jira. Then end with this prompt — exactly as written — outside the code block:

> Review the story above. Request any edits now. When it's ready, reply **"approved — push to [PROJECT KEY]"** to send it to Jira, or **"next"** to write another story first.

Do not push to Jira until the user explicitly approves. Treat any response other than a clear approval as a revision request or a cue to continue writing. Apply edits, re-output the full story in a new fenced block, and prompt again.

## Story Structure

Every user story follows this exact template:

```
**Title:** [Concise, descriptive title]

**Story:**
As a [persona], I want [capability], so that [business value/outcome].

**Type:** [Story | Spike | Bug | Task] (include Time Box for Spikes)
**Blocks:** [Other stories this blocks, if applicable]

---

**Acceptance Criteria:**

**[Logical Group Name]**
- Given [precondition], when [action], then [expected result].
- ...

**[Next Logical Group Name]**
- Given [precondition], when [action], then [expected result].
- ...

---

**Open Questions:**
- [Unresolved decisions, dependencies, or ambiguities surfaced during story writing]

```

## Writing Principles

These principles make the difference between stories that sit in a backlog and stories a team can actually build from.

### Acceptance Criteria

Acceptance criteria are the core of a useful story. They define "done" in a way that's testable and unambiguous.

- Always use Given/When/Then (Gherkin) format. This forces specificity and makes criteria directly translatable to test cases.
- Group criteria under logical subheadings when a story covers multiple concerns (e.g., "Google Authentication," "Magic Link," "Session Management"). This keeps long stories scannable without losing detail.
- Cover the happy path first, then edge cases and failure states. A story without failure handling is incomplete — always consider what happens when things go wrong (expired tokens, failed payments, rate limits, unavailable assets, etc.).
- Each criterion should be independently testable. If you can't write a test for it, it's too vague.
- Be specific about data and metadata. Don't just say "an event is recorded" — specify what fields are captured (user ID, timestamp, method, etc.).

### Open Questions

Open questions are not filler — they surface real decisions that need to be made before or during implementation. They serve as a forcing function for product clarity.

- Surface technical decisions the team needs to make (e.g., "Should we use official APIs or third-party scrapers?")
- Flag dependencies on other stories or external inputs (e.g., "Does this depend on Story 3 being complete?")
- Identify scope ambiguities (e.g., "Should locked-out users retain read-only access?")
- Ask about missing specifications (e.g., "What are the specific tier limits and pricing?")
- Connect dots across stories — if a decision in one story affects another, call it out.
- Aim for 4–7 open questions per story. Fewer suggests you haven't thought deeply enough; more suggests the story might need splitting.

### Story Types

**Standard Stories** cover feature work — the bulk of what you'll write. They have a clear user persona, a capability, and business value.

**Spikes** are time-boxed research tasks. They exist when the team needs to make a decision before building. Spikes always include:
- A time box (e.g., 3–5 days)
- What stories they block
- Context and initial assessment (what's already known)
- A clear deliverable (usually a written recommendation with rationale)
- A decision gate (how the team acts on the spike's output)

When the user describes a need to evaluate options, compare tools, or research approaches, suggest framing it as a spike rather than burying evaluation criteria inside a feature story.

### Phased Implementation

When a feature has a natural MVP and future expansion path, structure the acceptance criteria in phases within a single story rather than splitting prematurely. Label phases clearly (e.g., "Phase 1 — Industry-Based Suggestions (MVP)" and "Phase 2 — Similarity-Based Expansion (Future, Architecture-Ready)"). This keeps the full vision in one place while making it clear what's being built now vs. later. Phase 2 criteria should focus on architectural readiness — what needs to be true about the phase 1 implementation so that phase 2 isn't a rewrite.

### Connecting Stories

Stories in a product backlog don't exist in isolation. When writing a story, actively look for connections:
- Reference related stories by name/number when there are dependencies
- Note when a decision in one story affects another
- Suggest where analytics events (login tracking, feature usage) should tie into analytics stories
- Flag when a new story might create acceptance criteria that should be added to an existing story

### Tone and Detail Level

- Write for a development team that is technical but may not have full product context. The story should be self-contained enough that a developer picking it up cold can understand what to build and why.
- Be opinionated in implementation notes when you have relevant expertise — suggest specific tools, approaches, or architectures. Frame these as recommendations, not requirements.
- Use precise language. "The system should handle errors gracefully" is useless. "Given a webhook delivery fails, when the gateway retries, then the application handles duplicate events idempotently" is actionable.

## Conversation Flow

The user will typically describe features conversationally — sometimes in fragments, sometimes with evolving context. The skill should:

1. Take whatever context is provided and produce a complete story immediately — don't over-ask before delivering value.
2. Ask clarifying questions after the story is drafted, not before. Deliver a strong first draft and let the user refine.
3. When the user adds context that changes a previously written story, note how the new information affects the earlier story but focus on the current request.
4. When the user's description is ambiguous, make a reasonable assumption in the story and surface the ambiguity as an open question.
5. After each story, prompt for review and Jira push approval — never skip this step.

## Jira Push

Stories are pushed to Jira one at a time, after explicit approval. Use the `mcp-atlassian` MCP server.

**Gate:** Only push when the user replies with a clear approval (e.g., "approved — push to ICLD"). Any other response is a revision request or a signal to continue writing.

**On approval, call `jira_create_issue` with:**

| Jira field | Source |
|---|---|
| `project_key` | From user's approval message |
| `summary` | Story title |
| `description` | Full formatted story body (markdown, excluding the title line) |
| `issue_type` | `"Story"` for standard stories; `"Task"` for Spikes |
| `priority` | Map from story: Critical → `Highest`, High → `High`, Medium → `Medium`, Low → `Low` |

After the call completes, confirm with the created Jira key (e.g., "Pushed as **ICLD-16**.") then ask if they want to write another story.
