---
name: prd-and-technical-design
description: Assemble a single combined PRD + Technical Design (+ optional estimate) document with one shared vocabulary, stable requirement IDs, and traceability from requirement to design to effort. Pulls together outputs from the requirements, design, and estimation skills (or raw one-pagers, specs, transcripts, notes). Use when asked to produce one doc that combines requirements, technical design, and estimate, or to reconcile separate docs into one.
argument-hint: "[--no-estimate]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# prd-and-technical-design

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`. It is the structure to follow.

## Objective
Produce **one document in two or three parts** (Part 1 PRD, Part 2 Technical Design, optional Part 3 Estimate) that reads as if one person wrote it. The same term, ID, and option name means the same thing everywhere. A reader should be able to follow any requirement from "why we want it" to "how we build it" to "what it costs" without leaving the page.

This skill is the glue. It does not replace `one-pager-to-spec`, `estimate-effort`, or `llm-assisted-effort-estimate`; it consumes their output when it exists and reconciles it.

## Step 1: Collect inputs
Expect three Markdown files and ask for their paths in one message (pasted text or Confluence/Jira links also work; read those via the `atlassian-cli` skill):

| Input | Typical producer | Feeds |
| --- | --- | --- |
| **Spec** (`.md`) | `one-pager-to-spec` | Part 1: summary, workflow, requirements, FR/NFR |
| **Technical spec** (`.md`) | `spec-to-technical-structure` or a design doc | Part 1 architecture overview, all of Part 2 |
| **Estimate** (`.md`) | `estimate-effort` / `llm-assisted-effort-estimate` (skip with `--no-estimate`) | Part 3, regrouped into milestones |

Optional extra context: transcripts, Slack threads, API docs, a prior version of this doc.

The three files were written separately, so expect them to disagree on names, IDs, scope, and numbers. Do not paste them together. Reconcile in Step 2, keep the spec's requirement wording as the source for Part 1, the technical spec's decisions as the source for Part 2, and the estimate's numbers as the source for Part 3. If they conflict on substance (for example the estimate covers work the spec does not mention), flag it in Open Items instead of choosing silently.

If a file is missing, do not invent it. Draft that part from what exists and mark the gap in Open Items.

## Step 2: Build the shared vocabulary first
Before writing any section, extract and fix:

1. **Glossary of terms and field names.** One name per concept. If sources disagree (for example `merchant_id` vs `entity_id`), pick the one the external system or code actually uses and note the retired name once, in the glossary.
2. **Increments / phases.** Name them once (Increment 1, Increment 2) and use only those names.
3. **Options.** Name each design alternative once (Option 1, Option 2) with a one-line label. Mark the status: *Selected starting point*, *Fallback*, or *Discarded*.
4. **Actors and systems.** One name each, and a fixed ownership tag used in every diagram: *external system*, *new build*, *existing system*.
5. **ID scheme.** `FR-<area>.<n>` for functional requirements, `NFR-<n>` for non-functional. IDs never change once issued; new requirements get new numbers.

Show the glossary, increments, and options to the user as a short list and fix anything they correct before continuing. This is the one checkpoint worth pausing on.

## Step 3: Write Part 1, the PRD
Follow the template sections in order. Rules that make this part work:

- **Lead with the outcome.** Executive summary says what is being built, for whom, and the split into increments. Value drivers are a short bullet list, each with a bold lead-in.
- **One color-coded flow diagram** (Mermaid) showing external, new build, and existing systems, grouped by increment. Use the ownership tags from Step 2 as diagram styles.
- **Business requirements as a two-column table** (category, details): timeline, approvals, analytics, external dependencies, future scope. Put unconfirmed items in the text, flagged.
- **NFRs and FRs get IDs and a short bold title**, for example `NFR-2 (Sanity gate and kill switch)`. FRs are grouped by capability and tagged with who builds them and which increment. Describe behavior, not code.
- **Mark certainty honestly.** Anything not confirmed is written as `(Proposal, to be confirmed)`. Anything that depends on an outside party is named with that party.
- **Fallback triggers.** When a requirement has a selected approach and a fallback, list the exact conditions that would make us switch.
- **Architecture overview** is a short list of components, one line each, tagged by increment or option. No class or endpoint detail.

## Step 4: Write Part 2, the Technical Design
Every section answers a requirement from Part 1. Cite the IDs.

- **Overview** in 2 to 3 sentences, restating the increments.
- **Design options, driven by the requirements.** One section per requirement area that has a real design choice, titled for the topic. Each lists its options in the same order: Option 1 as the selected starting point (mechanism, why start here, trade-off, guardrails, still to confirm, what to measure before launch), then Option 2 as the fallback (move triggers matching Part 1 word for word, mechanism, baseline). Add a sequence diagram only where it clarifies the flow.
- **Tradeoff Analysis & Discarded Alternatives** as a table with a Decision column using only *Selected starting point*, *Fallback*, or *Discarded*, and a reason for each.
- **Roadmap: After Increment N Launch**: follow-ups grouped by option, each with what decides it.
- **References & External Documentation**: each link with a one-line note on what it is for.

Do not add feature-specific sections (system context diagrams, data mapping tables, sync mechanics) unless the user asks; put that detail inside the relevant option instead.

## Step 5: Write Part 3, the Estimate (skip with `--no-estimate`)
Milestone-based and easy to read at a glance. Estimation level, not a task breakdown.

- **Milestones, not work areas.** Each milestone is a demoable outcome in plain words ("providers appear in the partner's sandbox"), not a technical layer. Aim for 3 to 6, sequenced so each builds on the last, and aligned to the increments and Option 1 as the starting point.
- **One-line flow diagram** (Mermaid) of the milestones with an effort range on each, then a table: milestone, what you can see when it is done, requirement IDs covered, heads-down effort range, main risk. End with a Total row.
- **Effort is a range** (for example 1 to 2 weeks), not a single number.
- **Where the human time goes**, in two short bullets: what is likely AI-assisted, and where human judgment is essential (clarifying requirements, choosing the approach, guardrails, review, validation, integration, security). Treat AI acceleration as an assumption to validate; apply no blanket percentage reduction.
- **Waits** listed separately from heads-down effort, each with an owner and the milestone it blocks.
- If an estimate skill output exists, reuse its numbers and wording and regroup them into milestones; do not recompute.

## Step 6: Consistency pass (do this before presenting)
Check and fix silently where the fix is obvious, otherwise list under Open Items:

1. Every FR/NFR in Part 1 appears in Part 2 (or is explicitly "no design needed") and in Part 3.
2. Every design section cites at least one requirement ID.
3. Glossary terms, field names, increment names, and option names are identical everywhere. Search for the retired names.
4. Move triggers for a fallback are the same wording in Part 1, the options section, and the trade-off table.
5. Diagram labels match the text and the ownership tags.
6. No unconfirmed item is stated as fact.
7. Nothing appears twice in different words. Keep the detailed version in one place and link from the other.

## Step 7: Output
Write the document as Markdown to a file the user names (default: `./<feature-name>-prd-and-technical-design.md`). Mermaid goes in fenced `mermaid` blocks. If the user wants it on Confluence, hand the file to the `atlassian-cli` skill (`conf page create --markdown`), which renders Mermaid automatically.

Do not add a status banner. Start with the title and the Glossary.

## Style
- Plain, natural language that an engineering leader can skim. No filler, no spec-speak.
- Tables for comparisons and mappings, bullets for requirements, diagrams for flows. Do not add a diagram that repeats another.
- Depth fits the purpose: planning and estimation level, not build-ready design. No SQL, class designs, or full endpoint contracts unless the source needs them to size the work.
- Prefer scaling down over padding. A small feature gets a short doc; drop sections that would be empty rather than filling them.
