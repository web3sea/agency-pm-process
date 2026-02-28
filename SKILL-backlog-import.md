---
name: backlog-csv-import
description: Format and export stories from user-stories-backlog.csv into Jira-ready markdown. Trigger when the user says anything like "process the backlog", "format the CSV", "export stories from the backlog", "pull stories from the sheet", "give me US-003 from the CSV", "show me all High priority stories", or wants batch-formatted output from the backlog file — even if they don't mention CSV or Jira explicitly.
---

# Backlog CSV Import Skill

Read rows from `user-stories-backlog.csv` and output fully-formatted Jira-ready story markdown. This is a deterministic formatting workflow: the CSV is the source of truth. AI generation is only used as a fallback for rows with missing or thin acceptance criteria.

## Deterministic Workflow

Follow this exact sequence on every invocation. Do not deviate.

---

### Step 1 — Determine Scope

Before reading any file, check the user's request for a scope filter.

| Filter type    | Example phrase                        | Behavior                                              |
|----------------|---------------------------------------|-------------------------------------------------------|
| Specific IDs   | "US-002, US-005"                      | Process only those story IDs                          |
| Priority       | "all High priority"                   | Filter rows where `priority` matches                  |
| Type           | "all spikes"                          | Filter rows where `type` matches                      |
| Tag            | "all analytics stories"               | Filter rows where `tags` contains the term            |
| No filter      | (nothing specified)                   | Process all rows                                      |

If no filter is stated, process all rows.

---

### Step 2 — Read the CSV

Use the Read tool to read `user-stories-backlog.csv` from the project root.

**Parsing rules:**
- Row 1 is the header — use it to map column positions: `story_id, title, type, story, acceptance_criteria, open_questions, blocks, time_box, priority, tags`
- Fields containing commas or newlines are wrapped in double quotes — treat their full content as a single field value
- Apply the scope filter from Step 1 to select the rows to process
- Maintain original row order (by story_id) throughout

---

### Step 3 — Classify Each Row

For each row in scope, classify it:

- **Complete** — `acceptance_criteria` has ≥ 3 Given/When/Then criteria
- **Thin** — `acceptance_criteria` is empty or has fewer than 3 criteria

Note counts for the summary in Step 5.

---

### Step 4 — Process Stories

**Complete rows:** Format directly using the Output Template below. No subagent needed.

**Thin rows:** Launch one subagent per thin row using the Task tool. Run all thin-row subagents in parallel (single message, multiple Task tool calls).

```
Task tool:
  subagent_type: general-purpose
  model: haiku
  description: Generate full AC for [story_id]
  prompt: |
    You are a senior product manager at an AI automation agency. Write a complete,
    Jira-ready user story from the data below. Follow the format and rules exactly.

    ## Story Format

    **Title:** [title]

    **Story:**
    As a [persona], I want [capability], so that [outcome].

    **Type:** [include only if not "Story" — e.g., Spike (Time Box: X days)]
    **Blocks:** [include only if specific story IDs are blocked; omit otherwise]

    ---

    **Acceptance Criteria:**

    **[Semantic Group Name]**
    - Given [precondition], when [action], then **[expected result]**.

    ---

    **Open Questions:**
    - **Header:** Question with tradeoffs stated inline.

    ## Acceptance Criteria Rules
    - Group criteria under labeled semantic headers — never a flat list
    - Every group must include at least one failure-state criterion
    - Use concrete numbers: "15 minutes," "3–7 days," "80% threshold" — never vague
    - For event tracking: always list exact fields (user ID, timestamp, method, etc.)
    - Stories with subscriptions/payments/user state: include "Admin Visibility" section
    - Stories with analytics/PII: include "Data & Privacy" section
    - Stories with webhooks: include idempotency criterion
    - AI/ML features: specify min data threshold, confidence behavior, user override, re-evaluation frequency
    - Bold key outcomes in the "then" clause and concrete numbers inline

    ## Open Questions Rules
    - 4–7 questions per story
    - Technically framed with tradeoffs stated inline
    - Cross-reference other stories by ID when relevant

    ## Input Data
    story_id: {story_id}
    title: {title}
    type: {type}
    story: {story}
    blocks: {blocks}
    time_box: {time_box}
    priority: {priority}
    tags: {tags}
    existing_ac: {acceptance_criteria}
    existing_questions: {open_questions}
```

---

### Step 5 — Output

For each story (in story_id order), output:

1. A level-2 heading: `## {story_id} — {title}`
2. A fenced markdown code block (```` ```markdown ... ``` ````) containing the formatted story
3. A blank line

After all stories, print this summary line:

> Processed **X** stories from `user-stories-backlog.csv` — Y formatted from CSV data, Z generated by AI.

Then ask: **"Push these to Jira? If yes, confirm the project key."**

---

### Step 6 — Jira Push (conditional)

Only execute if the user confirms. Use the `mcp-atlassian` MCP server to create issues.

**For each story, call `jira_create_issue` with:**

| Jira field | Source |
|---|---|
| `project_key` | Confirmed by user (e.g., `ICLD`) |
| `summary` | `{title}` |
| `description` | Full formatted story body (markdown, excluding the title line) |
| `issue_type` | `"Story"` for type Story; `"Task"` for type Spike |
| `priority` | `{priority}` from CSV — map: Critical → `Highest`, High → `High`, Medium → `Medium`, Low → `Low` |

**Run all `jira_create_issue` calls in parallel** (single message, multiple tool calls).

After all calls complete, output a confirmation table:

| story_id | Title | Jira Key | Status |
|---|---|---|---|
| US-001 | ... | ICLD-42 | ✓ Created |
| US-002 | ... | — | ✗ Failed: [reason] |

If any calls fail, report the error and the raw story data so nothing is lost.

---

## Output Template

Apply this template to every row. Instructions in `[brackets]` describe conditional logic — do not include the bracket text in output.

```
**Title:** {title}

**Story:**
{story}

**Type:** {type}  [omit this entire line if type is "Story"]
**Time Box:** {time_box}  [include only when type is "Spike" and time_box is non-empty]
**Blocks:** {blocks}  [omit this entire line if blocks is empty]

---

**Acceptance Criteria:**

{acceptance_criteria}

---

**Open Questions:**
{open_questions}
[omit the "Open Questions" heading and section entirely if open_questions is empty]
```

---

## Formatting Rules

### Acceptance Criteria

- Preserve existing group header structure: bold group names, bulleted Given/When/Then items
- If the raw CSV text is not already in Given/When/Then format, convert it: infer precondition, trigger, and expected outcome from context
- **Bold key outcomes** in the "then" clause — the thing the engineer must make true
- **Bold concrete numbers and thresholds** inline: **15 minutes**, **80%**, **3–7 days**
- Do not bold entire sentences — bold only the critical phrase or value

### Conditional Lines

- Omit the **Type:** line entirely when type is "Story"
- Include **Time Box:** only when type is "Spike" and time_box field is non-empty
- Omit **Blocks:** when the blocks field is empty
- Omit the **Open Questions** section entirely when open_questions field is empty

### Code Blocks

- Each story must be wrapped in its own fenced ` ```markdown ` code block so the user can copy-paste directly into Jira
- Do not merge multiple stories into a single code block

---

## Error Handling

- If `user-stories-backlog.csv` is not found at the project root: report the error clearly and stop — do not guess alternate paths
- If a row cannot be parsed (malformed CSV): skip it and include the story_id in the summary as "skipped (parse error)"
- If a subagent fails for a thin row: include the raw row data in the output with a note that AC generation failed, and include it in the summary as "skipped (generation failed)"
