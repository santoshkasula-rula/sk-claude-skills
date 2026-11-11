---
name: estimate-effort
description: Turns a spec, technical design (TDD), and repo into a visual estimate document, as markdown plus a standalone HTML file. Sections are Problem Statement, Solution Approach, Requirements, Architecture, and Estimates by milestone in heads-down engineering days, assuming a coding bot writes the code, with a Delivery days view at the top that adds a 20% buffer and states that compliance review handling and engagement run in parallel. Falls back to a lite mode (PRD plus transcript) when no spec exists.
disable-model-invocation: true
argument-hint: "[--root] [--lite] [--cards]"
allowed-tools: Read Bash(find *) Bash(ls *) Bash(git rev-parse *) Write
---

# estimate-effort

Read `OUTPUT_TEMPLATE.md` and `OUTPUT_TEMPLATE.html` from `${CLAUDE_SKILL_DIR}`.

Flags in $ARGUMENTS:
- `--root`: multi-repo discovery in Step 2.
- `--lite`: no spec exists. Start from PRD and transcript, and do Step 3b.
- `--cards`: also write one self-contained Claude Code task card per bot-ready task. Off by default, because cards are implementation detail.

Core idea: estimate the **human heads-down engineering days**, assuming a coding bot writes the code and tests. Present it visually, short, for someone who has not read the spec.

Conventions used throughout:
- **Evidence tags** on every non-trivial claim: **said**, **inferred**, **assumed**, **from vendor docs**. Check vendor claims against current docs (fetch them); if you cannot, tag **assumed**.
- **Heads-down days are pure.** They exclude waits (reviews, access, vendor replies), buffer, and decision-making. Buffer is added only in the **Delivery days** view at the top (Step 7), never inside heads-down days or milestone blocks. Security and compliance review handling and stakeholder engagement are assumed to run in parallel with heads-down work, so they are called out as an assumption and not added as days.
- **No blanket AI discount.** The bot writes code, so human time is design of the hard parts, reviewing bot output, validating behavior, integration, and hands-on setup. Say which.
- **Visuals first.** Every section leads with a diagram or table where one helps. Prose is short.
- **Never invent** paths, functions, or commands. Write "not found".

---

## Step 1 — Collect inputs

Ask for each in turn and read it before asking for the next.

1. **Spec** (required unless `--lite`): latest version, with requirement IDs if it has them.
2. **Technical design (TDD / technical structure)**: for example the output of `spec-to-technical-structure`. If missing, say confidence is lower and offer to run that skill first.
3. **Mocks** (optional, for UI work): reference screens by name.
4. **Transcript or discussion notes**: "The part where someone said 'I'm not sure how long that'll take' counts. Path, paste, or skip." Normalize it: strip timestamps, speaker labels, `[inaudible]`.
5. **Context**: team size, test and CI conventions, feature flag system, definition of done, anything the bot must never touch.
6. **More**: "Anything else that would embarrass us if we ignored it? (path, paste, or done)". Repeat until done.

If the spec and a 1-pager disagree, the spec wins. Transcript anxiety markers override optimism on the same component.

## Step 2 — Repo discovery

If `--root`:
```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

Always check for a git root (`git rev-parse --show-toplevel`). If inside a repo, read the manifests that exist (`package.json`, `go.mod`, `docker-compose.yml`, `requirements.txt`, `pyproject.toml`) and list internal services only. Confirm in one message:

> **I found these internal services. Which are coming along for the ride?**
> - `[service]` — [inferred purpose]

Wait for the reply. If no git root, skip silently.

## Step 3 — Ground it in the code

For every task you will propose, find and record:
- Files and modules it touches (paths).
- An **existing pattern to follow**, with its path. No precedent means more human time or a spike.
- How tests are written and run (discover from package.json, Makefile, CI config; do not guess commands).
- How feature flags, config, and migrations are done.

If you cannot read the repos, say so and lower confidence.

## Step 3b — Requirements check (only if `--lite` or the spec is thin)

Extract the problem, use cases, user stories, acceptance criteria, success metrics, scope in and out, constraints. Tag each with evidence. Present one summary:

> **Here's what I think we're building. Fight me on anything wrong:** [problem, stories, requirements, constraints, out of scope]

Ask at most one question. Then, in lite mode only, give a **brief** solution check: 2 candidate approaches with one line each on what they touch and their risk, a recommendation, and why. In normal mode, take the approach from the technical design and do not re-litigate it.

## Step 4 — Slice into tasks

Smallest units that each deliver a testable result and fit one PR. Slice vertically where you can, horizontally where you must (shared types, schema). Name each task by the functionality it delivers, in plain language a PM could read ("Users can save a payment method at checkout"), not by file or layer. Put the technical detail in a short bulleted list under it. Every task traces to spec IDs (and a mock screen if UI).

Decisions are not tasks: list them as open decisions with an owner, unsized.

Also include **Setup** tasks (infra, access, config, secrets) per milestone. Do not create per-milestone Rollout tasks: rollout (cutover, UAT preparation, training material) is a single 2-day general allowance applied once in the Delivery days view (Step 7).

## Step 5 — Bot or human: what drives the effort

For each build task, decide how much human time it needs. A task is **bot-ready** only if all eight are true:

1. Unambiguous: no open question or undecided decision affects it.
2. Pattern exists: a close precedent in the repo, cited by path.
3. Small and contained: one layer, one service, about 8 files or fewer.
4. Testable: automated tests or a concrete command can check it.
5. Reversible and safe: additive, behind a flag, no destructive migration or public contract change.
6. Not high-judgment: not authorization, encryption or secrets, PHI handling or logging, consent, money or billing, audit-trail semantics, or compliance rules.
7. No missing inputs: no credentials, vendor decision, missing design, or other team's unfinished work.
8. Self-contained: doable from the task description plus the repo.

Bot-ready tasks cost **review time** only. Tasks that fail a gate are **human-led** and cost human days for the part a person must own (the hard logic, integration, debugging); carve out what the bot can still draft (types, scaffolding, fixtures, tests from the spec, UI shells, docs). Sensitive logic is always human-led. Healthcare and compliance-sensitive areas default to human-led.

## Step 6 — Assumptions, risks, and execution order

Present a short list and up to 3 targeted questions in one message:

> **Here are the assumptions, risks, and dependencies. Fix anything that's wrong:** [each with source]

Pick questions that would change the estimate (area most likely to blow up, what must quietly be true, outside teams who must say yes, new-ground work). Never ask more than 3, never re-ask what documents answered. Wait, then update.

Then order the work: group tasks into waves that can run in parallel without editing the same files, and name the critical path.

## Step 7 — Estimate

**Heads-down days per task.**
- Bot-ready: review time (XS about 5 min, S 10 to 15 min, M 20 to 30 min). Bigger than M, split.
- Human-led: S 1 to 2 days, M 3 to 5, L 6 to 10. XL over 10: split or spike first.
- Setup: days of hands-on work.

Roll up to **half-day** granularity per milestone. Give a range and confidence where uncertain, and say what would tighten it.

**Milestones.** 3 to 5 incremental milestones, each putting something usable in a named stakeholder's hands, tied to an acceptance criterion, each unblocking the next. Use the documents' own language. For each milestone:
- **Technical changes** in human-readable bullets (what changes in the system, not file lists).
- **Who does what**: what the bot drafts, what a human owns.
- **Heads-down days**, with the split between review of bot output, human-led work, and setup.
- **Outcome**: what the named stakeholder can now do, tied to an acceptance criterion.
- **Complexity**: Low, Medium, or High, with one line on what drives it.
- **What would change it**: the 1 to 3 things that would move this milestone's days, with the size of the move.
- **Why this many days**: the 2 to 4 specific factors that add or remove days, each tagged with its evidence and its rough day contribution (for example "no precedent for this integration, +1.5d, inferred" or "close pattern at `src/api/addresses.ts`, bot-ready, review only"). Draw from: new ground versus existing pattern, number of systems touched, sensitive logic that must stay human-led, test coverage gaps, data migration, and unclear requirements. The contributions must add up to the milestone's heads-down days.

**Heads-down days are the plain sum.** Do not pad them, and do not count waits.

**Delivery days.** Per milestone: delivery days = heads-down + buffer, where buffer = 20% of the Core days only (not Setup). Then add one **general Rollout allowance of 2.0 days** to the project total, as its own row after the last milestone and never inside a milestone. It is a fixed assumption (tag it **assumed**); change it only if the documents give a different figure. The running total adds it at the end.

**Parallel work (assumption, not days).** Security and compliance review handling (preparing write-ups, answering Legal and Security questions, fixing findings) and stakeholder engagement (working sessions, demos, UAT, reconciliation reviews, sign-off walkthroughs) are assumed to happen alongside heads-down work and not to extend it. State this once in the Delivery days card, tagged **assumed**, and name which milestones carry the most of it. Do not size it. If the documents say it blocks the engineer, add that time back and say so.

**Core and Setup.** Split each milestone's days into Core (review of bot output plus human-led work) and Setup (infra, access, config). Show the split in the table and the bar.

**Previous version.** If an earlier estimate exists, say what moved and why, with old and new totals.

**Calendar days** = delivery days ÷ 0.7, shown as a separate column (state the 0.7 as an assumption to calibrate). The milestone "Day N" in the Delivery days table is a running total of calendar days from day 0.

**Not included** (list, no durations unless the documents give them): the reviewers' own elapsed time (Legal, Security), PR review turnaround, access provisioning, other teams' queue time, vendor replies, pilot run time, and open decisions. Name the waits you know of and who they depend on, and say which dates slip day for day if one runs late.

**Confidence and risk flag** per milestone: High, Medium, or Low, and 🔴 (anxiety marker or risk), 🟡 uncertain, 🟢 well understood.

**What moves the numbers.** List the assumptions and risks that would change the total as "if X, about +N days" rows, each with milestone and evidence tag. List the decisions that would turn the most human-led work into bot-ready work, with who decides.

**Open questions.** Where the repos can answer a question, answer it with file or table names and mark what the team still has to confirm. List changes needed from other teams, by milestone, and roughly what share of the days sit in systems they own or review. Label that split an assumption.

**Totals to report:** total delivery days, heads-down days, buffer, calendar days, final day; task counts bot-ready versus human-led; the share of effort that is human-led; and the effect of unresolved decisions ("if D-2 resolves as X, N tasks become bot-ready").

## Step 8 — Write the outputs

Write both, with the same content and numbers:
- `Effort Breakdown Analysis - [Project Name].md` following `OUTPUT_TEMPLATE.md`.
- `Effort Breakdown Analysis - [Project Name].html` following `OUTPUT_TEMPLATE.html`: one self-contained file, rendered diagrams, effort bars, light and dark friendly.

Both files open with **Delivery days** (tiles for delivery, heads-down, buffer, and calendar days; a per-milestone table with those columns and a running total; a card explaining the buffer, the parallel-work assumption for compliance review handling and engagement, and what is not included), then the sections in order:

Core, always present:
1. **Problem statement**: what is blocked or painful, what success looks like.
2. **Solution approach**: milestone flow (with a gate if one exists), main risk accepted, alternatives, out of scope.
3. **Requirements**: user stories, functional requirements with spec IDs and the milestone that delivers each, undefined items.
4. **Architecture, tl;dr**: data flow marked new / reuse / later, "what we found in the repos" precedent table from Step 3, a "watch out" card for the one behavior most likely to cause silent errors, and open design choices (as possible approaches).
5. **Estimates**: milestone table, then one block per milestone (outcome, complexity, why these days, what would change it, technical change table with who and days), and a comparison with the previous version only if one exists.

Supporting, omit any that do not apply:
6. **What moves the numbers**: if-X-then-N-days table, decisions that unlock the most work, and the not-included note.
7. **Open questions**: answered from the repos where possible, with what still needs the team, plus changes needed from other teams and the share of the estimate they touch.
8. **Sources and what I could not verify**, with the calibration line.
9. **Later phase preview**: shape only, not estimated.

Do not invent sections beyond these.

**Visuals.** Include at least: a milestone flow, an architecture flow marking new versus reused work, and effort bars. In the markdown use Mermaid (labels short, no parentheses, no line breaks inside labels). In the HTML follow `OUTPUT_TEMPLATE.html`: its CSS flow nodes, chips, tiles, and milestone cards, so the file works offline except fonts.

If `--cards`, also write `tasks/A-<nn>-<slug>.md` per bot-ready task: goal, context, files and pattern path, steps, checkable acceptance criteria, exact test commands, guardrails (additive only, no PHI or real names in fixtures, behind a flag), human review focus. Cards must work pasted into a fresh Claude Code session.

**Calibration.** End the Estimates section with one line: after the first milestone, record actual review time, rework, and tasks that turned out human-led, then re-run the estimate. First numbers are assumptions.

**Safety.** Read repos only. Never claim tests pass without running them.

**Consistency check (final step).** Re-read both files. Confirm that heads-down + buffer = delivery days per milestone, and that milestone days, totals, calendar days, dates, and confidence match exactly between the markdown and the HTML. Fix stale numbers.

End with:
> **Share with PMs / leadership:** Problem Statement, Solution Approach, Estimates summary
> **Share with engineers:** full document

Then reply in chat with the file locations, the headline (delivery days, heads-down days, calendar days, final day, human-led share), the top 3 decisions that would move the most work to the bot, the first wave ready to start, and what you were least sure about.
