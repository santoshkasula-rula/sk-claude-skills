---
name: estimate-effort
description: Turns a spec (and its technical structure) into an AI-assisted estimate. Slices work into tasks, sorts each into Bucket A (agent-ready) or Bucket B (human-led), then groups them into milestones with dated plans, wait-time assumptions, and a RAD. Falls back to a lite mode (PRD plus transcript) when no spec exists.
disable-model-invocation: true
argument-hint: "[--root] [--lite] [--cards]"
allowed-tools: Read Bash(find *) Bash(ls *) Bash(git rev-parse *) Write
---

# estimate-effort

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`.

Flags in $ARGUMENTS:
- `--root`: multi-repo discovery in Step 2.
- `--lite`: no spec exists. Start from PRD and transcript, and do Step 3b (requirements synthesis and a brief solution check).
- `--cards`: also write one self-contained Claude Code task card per Bucket A task. Off by default, because cards are implementation detail.

Core idea: shrink the human-led work. Sort every task into A or B, carve A pieces out of B, then turn the sorted tasks into milestones and dates a PM can plan around.

Conventions used throughout:
- **Evidence tags** on every non-trivial claim: **said**, **inferred**, **assumed**, **from vendor docs**. Check vendor claims against current docs (fetch them); if you cannot, tag **assumed**.
- **Heads-down engineering time only** in effort and dates. Meetings, alignment, and decision-making are never estimated. Waiting goes in Step 8b.
- **No blanket AI discount.** AI help shows up as Bucket A review time, not a percentage cut.
- **Never invent** paths, functions, or commands. Write "not found".

---

## Step 1 — Collect inputs

Ask for each in turn and read it before asking for the next.

1. **Spec** (required unless `--lite`): latest version, with requirement IDs if it has them.
2. **Technical structure**: output of `spec-to-technical-structure`. If missing, say confidence is lower and offer to run that skill first.
3. **Mocks** (optional, for UI work): reference screens by name.
4. **Transcript or discussion notes**: "The part where someone said 'I'm not sure how long that'll take' counts. Path, paste, or skip." Normalize it: strip timestamps, speaker labels, `[inaudible]`.
5. **Context**: team size, test and CI conventions, feature flag system, definition of done, anything Claude Code must never touch.
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
- An **existing pattern to follow**, with its path. No precedent is a signal toward Bucket B or a spike.
- How tests are written and run (discover from package.json, Makefile, CI config; do not guess commands).
- How feature flags, config, and migrations are done.

If you cannot read the repos, say so and lower confidence.

## Step 3b — Requirements check (only if `--lite` or the spec is thin)

Extract use cases, acceptance criteria, success metrics, scope in and out, constraints. Tag each with evidence. Present one summary:

> **Here's what I think we're building. Fight me on anything wrong:** [use cases, AC, metrics, constraints, out of scope]

Ask at most one question. Then, in lite mode only, give a **brief** solution check: 2 candidate approaches with one line each on touches and risk, a recommendation, and why. In normal mode, take the approach from the technical structure and do not re-litigate it.

## Step 4 — Slice into tasks

Smallest units that each deliver a testable result and fit one PR. Slice vertically where you can, horizontally where you must (shared types, schema). One-sentence goals; if it needs more, it is two tasks. Every task traces to spec IDs (and a mock screen if UI). Separate "build" and "verify". Decisions are not tasks: list them as open decisions (prerequisites) with an owner, and do not size them.

Name each task by the functionality it delivers, in plain language a PM could read ("Users can save a payment method at checkout"), not by file or layer. Put the technical detail (files, pattern to follow, data or API change, tests) in a bulleted list under the task.

Also create **Operational** tasks (setup, infra, access, config, secrets) and **Rollout** tasks (reconciliation, UAT preparation and execution, training material). Count hands-on work only; leave out meetings, demo sessions, and alignment.

## Step 5 — Classify: the Bucket A gate

A build task is **Bucket A only if all eight are true**; otherwise it is Bucket B (or split until the A part passes).

1. **Unambiguous**: no open question, ASSUMED item, or undecided decision affects it.
2. **Pattern exists**: a close precedent in the repo, cited by path.
3. **Small and contained**: one layer, one service, about 8 files or fewer, reviewable in under 30 minutes.
4. **Testable**: automated tests or a concrete command can check it.
5. **Reversible and safe**: additive, behind a flag, no destructive migration or public contract change.
6. **Not high-judgment**: not authorization, encryption or secrets, PHI handling or logging, consent, money or billing, audit-trail semantics, or compliance rules.
7. **No missing inputs**: no credentials, vendor decision, missing design, or other team's unfinished work.
8. **Self-contained**: doable from the task description plus the repo.

Record which gates failed for every B item. Mark `A*` for tasks that pass but sit next to sensitive code, and say what the reviewer must check. Sensitive logic itself is always B. Operational and Rollout tasks are human work and are not sorted into A.

## Step 6 — Shrink Bucket B

For each B item: carve out A tasks (types, additive migration scaffolding, stubs, fixtures, tests written from the spec, UI shells, API client wiring, docs); state the **human core**; state the **unlock** (the decision that would turn it into A); suggest how human and Claude Code collaborate.

## Step 7 — RAD and execution order

Present a pre-filled RAD and up to 3 targeted questions in one message:

> **Here's the RAD. Fix anything that's wrong:** Risks / Assumptions / Dependencies / Success criteria, each with source.

Pick questions that would change the estimate (most-likely-to-blow-up area, what must quietly be true, outside teams who must say yes, what people will assume is in scope, new-ground work). Never ask more than 3, never re-ask what documents answered. Wait, then update the RAD.

Then plan execution: dependency graph grouped into waves; tasks in a wave must not edit the same files; unblocking work first; open decisions listed as prerequisites with owners, not sized; name the critical path.

## Step 8 — Estimate, milestones, dates

**Task sizing**
- Bucket A: XS (one file, about 5 min review), S (about 3 files, 10 to 15 min), M (about 8 files, 20 to 30 min). Bigger than M, split. Report review time plus a rework allowance (for example "S: 15 min, 30% chance of one revision").
- Bucket B: S 1 to 2 days, M 3 to 5, L 6 to 10, XL over 10 (split or spike first). Give a range, a confidence, and what would tighten it.
- Operational and Rollout: days, with a range where uncertain.

**Milestones.** 3 to 5 incremental milestones (no Discovery milestone, since scoping and alignment are not estimated), each putting something usable in a named stakeholder's hands, tied to an acceptance criterion, each unblocking the next. Use the documents' own language, no generic placeholders.

**Per milestone, three separate buckets:**
- **Core Engineering**: A review time (converted to days) plus B days: design, code, review, tests, debugging.
- **Operational** and **Rollout**: as above.

**Buffer.** 20% on the Core Engineering B portion only, shown as its own column. Bucket A already carries a rework allowance. Never buffer Operational or Rollout.

**Focused to calendar.** Focused days = Core Eng + buffer + Operational + Rollout. Calendar days = focused ÷ 0.7 (state this as an assumption to calibrate). Effort stays focused-only: meetings, alignment, and decisions are not estimated as work, and the 0.7 factor is only the conversion to calendar time.

**Dates.** One planning date per milestone as a running total of calendar working days from day 0, with the underlying range shown beside it. No T-shirt sizes. Every file written uses the same dates.

**Confidence and risk flag** per milestone: High, Medium, or Low, and 🔴 (anxiety marker or RAD risk), 🟡 uncertain, 🟢 well understood.

**Bottom-up task table** behind the milestones: task, bucket (A, A*, B, Ops, Rollout), focused days, basis with evidence tag, certainty. Task totals must reconcile to milestone bucket totals; show the reconciliation and fix any mismatch.

**Totals to report:** task counts by A, A*, B; share in A by count and by effort; A review hours; B day range; excluded items; and the effect of unresolved decisions ("if D-2 resolves as X, N tasks move B to A").

## Step 8b — Wait-time assumptions

Waiting is **not** in effort or dates. List, per milestone: PR review turnaround, Legal/Security/architecture review, access and environment provisioning, vendor replies, QA queue, pilot runs, flag-ramp soak, UAT scheduling, sign-off. For each give assumed working days, who it depends on, an evidence tag, and which milestone dates slip day for day if it runs late. State a labeled default where nothing is known. Call out that **review delays (Legal, Security) are the biggest schedule risk**.

> **These waits are not in the dates. Each says what slips if it's late. Fix anything off:** [table]

Wait, then update. Also report an expected-wait scenario as a separate line, never replacing the plan dates. Assign confidence per milestone and overall.

## Step 9 — Write the output

Write `Effort Breakdown Analysis - [Project Name].md` in the current directory following `OUTPUT_TEMPLATE.md`. Tasks go in the Tasks section grouped by milestone: a functional title, then a bulleted technical-work list, then size and AI-assisted split. Keep the document lean; there is no per-repo breakdown or rationale essay, because the technical detail lives in each task. If `--cards`, also write `tasks/A-<nn>-<slug>.md` per Bucket A task: goal, context, files and pattern path, steps, checkable acceptance criteria, exact test commands, guardrails (additive only, no PHI or real names in fixtures, behind a flag), human review focus. Cards must work pasted into a fresh Claude Code session.

**Mermaid.** Use Mermaid in every markdown file. Labels short, no parentheses, no line breaks inside labels. Whenever you produce a visual, also export it to a standalone HTML file next to the markdown.

**Calibration.** End the document with a short plan: after wave 1, record actual agent time, review time, rework passes, and tasks that bounced to B, then re-run the estimate. First numbers are assumptions.

**Safety.** Read repos only. Never claim tests pass without running them. Healthcare and compliance-sensitive areas default to Bucket B.

**Consistency check (final step).** Re-read every file written. Confirm milestone dates, bucket totals, buffer, calendar days, confidence, and wait assumptions match exactly. Fix stale numbers.

End with:
> **Share with PMs / leadership:** Executive Summary, Requirements, Recommendation, Milestones
> **Share with engineers:** full document

Then reply in chat with the file locations, the headline (task counts, share in A, review hours, B day range, final date), the top 3 decisions that would move the most work to A, the first wave ready to start, and what you were least sure about.
