---
name: team-deployment-frequency
description: >
  Report per-team engineering activity for a GitHub org over a time window:
  merged PRs per team member and pooled deployment frequency (successful
  deploy-workflow runs on default branches whose deployed commit contains at
  least one merged PR authored by a team member). Also emits a weekly
  deployment-frequency scatter chart correlating merged-PR volume against
  deploy volume per team, with a Pearson r per team. Use when asked for team
  contribution stats, deployment frequency, DORA-style deploy counts across
  the repos a team touches, or how PR throughput correlates with deploy
  frequency.
allowed-tools:
  - Bash
  - Write
---

# Team Deployment Frequency & Contribution Report

## Purpose

Given one or more GitHub teams in an org, produce a per-team report with:

1. **Merged PRs per team member** over a time window (default: last 1 month).
2. **Deployment frequency, pooled per team** — count of successful *deploy*
   workflow runs (across ALL repos the team touched, including repos it does not
   own) whose deployed commit contains at least one merged PR authored by a
   member of that team.

Deployment frequency is defined **per team, not per repo** — a team contributes
to many repos and deploys are pooled across all of them.

## Prerequisites
- `gh` CLI authenticated for the org (`gh auth status`); `jq` and `bc` available.
- Read access to the target repos' Actions runs.

## Inputs (ask the user if missing)
- **ORG** — org login (e.g. `pathccm`).
- **TEAMS** — team slugs. Default: `partnerships partnership-deals partnership-new-verticals martech partnership-core`.
- **SINCE** — window start `YYYY-MM-DD` (default: 1 month ago).
- **DEPLOY_INCLUDE** — deploy-workflow name regex (default: `deploy`).
- **DEPLOY_EXCLUDE** — exclude regex (default: `preview|teardown|cleanup|\bpr\b`). Use a word
  boundary around `pr`, not a bare substring — a bare `pr` false-positives on
  "Deploy to **Pr**oduction" / "(**pr**od)" and silently drops real prod-deploy
  workflows.

## Deploy-run definition (all must hold)
- `status = completed` and `conclusion = success`
- run's `head_branch` == repo default branch
- workflow `name` matches DEPLOY_INCLUDE (case-insensitive)
- workflow `name` does NOT match DEPLOY_EXCLUDE (case-insensitive)
- `created_at` within the window

Attribution: for each qualifying run take `head_sha`, list PRs on that commit
(`/commits/{sha}/pulls`); if any is **merged** and authored by a member of team
T, the run counts once for team T. A run may count for multiple teams. Dedupe by
`run_id` within a team.

## Procedure

### Step 1 — Resolve teams -> members
```bash
ORG=pathccm
TEAMS="partnerships partnership-deals partnership-new-verticals martech"
for t in $TEAMS; do
  echo "== $t =="
  gh api "orgs/$ORG/teams/$t/members" --paginate --jq '.[].login'
done
```
Dedupe across teams; note multi-team members.

**The `partnerships` GitHub team is not a single-purpose roster** — it also
contains members of `partnership-deals` and `partnership-new-verticals` (e.g.
anuragchaudhrypath, avelez-rula, ilam-rula sit on `partnerships` *and* one of
the narrower teams). If the report needs a "core partnerships" view that
excludes deals/new-verticals crossover members, don't filter `partnerships`
by exclusion — use the fixed `partnership-core` roster below instead, which
is a hand-curated member list, not a GitHub team slug.

**`partnership-core` — fixed roster (not resolved via `gh api orgs/.../teams`):**
```
ashleyh-path       # Ashley Huntsman
rkrone-rula        # Rebecca Krone
dtrifonova-rula    # Diana Trifonova
ktalley            # Kevin Talley
sahal-asghar-rula  # Sahal Asghar
l-wholey-rula      # Lauren Wholey
imhlanga-rula      # Imran Mhlanga Jr
jbennett-rula      # Jason Bennett
```
These 8 logins were confirmed against `orgs/pathccm/teams/partnerships/members`
by name match (all 8 are members of the `partnerships` GitHub team). Treat
`partnership-core` as a team like any other in Steps 2-8 — it just skips the
`gh api orgs/.../teams/.../members` lookup and uses this literal list instead.
If the roster changes, update this list directly; there is no GitHub team to
re-query.

### Step 2 — Merged PRs per member (query each author separately)
`gh search prs` does **not** OR-combine multiple `--author` flags — passing
`--author=USER1 --author=USER2` silently searches only the last one given.
Query each member individually and merge results yourself:
```bash
SINCE=2026-06-28
for login in $MEMBERS; do
  gh search prs --owner="$ORG" --author="$login" --merged-at=">=$SINCE" \
    --limit 1000 --json number,repository,author,state,closedAt,url \
    | jq -c --arg login "$login" '.[] | select(.state=="merged") | . + {member:$login}'
  sleep 0.3   # avoid GitHub's secondary rate limit when firing many queries back-to-back
done
```
Aggregate merged-PR counts per member/team; collect the set of repos touched.
If a query 403s with "secondary rate limit", wait ~45s and retry just the
failed members rather than the whole loop.

### Step 3 — Enumerate deploy runs across touched repos
```bash
INCLUDE="deploy"; EXCLUDE="preview|teardown|cleanup|\bpr\b"
> deploy_runs.jsonl
for r in $REPOS; do
  br=$(gh api "repos/$ORG/$r" --jq .default_branch 2>/dev/null)
  gh api "repos/$ORG/$r/actions/runs?branch=$br&status=success&created=>=$SINCE&per_page=100" \
    --paginate --jq ".workflow_runs[]
      | select(.name|test(\"$INCLUDE\";\"i\"))
      | select(.name|test(\"$EXCLUDE\";\"i\")|not)
      | {repo:\"$r\", run_id:.id, name:.name, sha:.head_sha, created:.created_at}" \
    >> deploy_runs.jsonl
done
```
`--paginate` is required here, not optional — high-frequency workflows (ECS/EKS
deploys, Harness config deploys) routinely exceed 100 runs in a 30-60 day
window. Without it you silently get the first page only. After fetching, sanity
check for truncation:
```bash
jq -s 'group_by(.repo,.name) | map({repo:.[0].repo, name:.[0].name, count:length}) | map(select(.count % 100 == 0))' deploy_runs.jsonl
```
Any group landing on an exact multiple of 100 is a signal the fetch was capped —
re-verify pagination worked for that workflow.

### Step 4 — Attribute runs to teams via merged PRs on the SHA
```bash
> deploy_attrib.jsonl
while read -r line; do
  repo=$(jq -r .repo <<<"$line"); sha=$(jq -r .sha <<<"$line")
  authors=$(gh api "repos/$ORG/$repo/commits/$sha/pulls" \
    --jq '[.[] | select(.merged_at!=null) | .user.login] | unique | join(",")' 2>/dev/null)
  jq -c --arg a "$authors" '. + {merged_pr_authors:$a}' <<<"$line"
done < deploy_runs.jsonl >> deploy_attrib.jsonl
```
Per team, count runs where any `merged_pr_authors` entry is a team member
(dedupe by `run_id`).

### Step 5 — Emit report, organized by team
```
| # | Team / Member | Merged PRs | Deployment frequency |
|---|---------------|:---:|:---:|
|   | **<Team>** (total) | <sum> | <pooled deploy count> |
| 1 | <member>      | <n> |  |
```

Before trusting the numbers, run `gh api repos/$ORG/<repo>/actions/workflows` on
a representative repo to confirm the deploy workflow names actually match
DEPLOY_INCLUDE and aren't called something like "Release" or "CD" instead.

Link the Step 8 scatter chart from this report (e.g. "Weekly PR/deploy
correlation: `Team Deployment Frequency - PR-Deploy Correlation -
YYYY-MM-DD.html`") so the reader can jump from the table to the chart.

### Step 6 — Bucket merged PRs and deploys into matching weekly bins

Both series need to land in the *same* week boundaries, starting at `SINCE`,
so the two counts line up point-for-point in Step 7/8.

```bash
python3 - <<'PYEOF'
import json, datetime

SINCE = "2026-05-30"     # same value used in Steps 2-4
TODAY = "2026-07-29"

since_d = datetime.date.fromisoformat(SINCE)
today_d = datetime.date.fromisoformat(TODAY)

# merged PRs: list of {member, team, mergedAt} — from Step 2's aggregated output
prs = json.load(open("merged_prs.json"))
# deploy runs already attributed to a team: list of {team, run_id, created} — from Step 4
deploys = json.load(open("deploy_attrib_by_team.json"))

def week_bins(start, end):
    bins = []
    cur = start
    while cur <= end:
        wend = min(cur + datetime.timedelta(days=6), end)
        bins.append((cur, wend, wend < cur + datetime.timedelta(days=6)))
        cur += datetime.timedelta(days=7)
    return bins

bins = week_bins(since_d, today_d)

def bucket(dt_str):
    d = datetime.date.fromisoformat(dt_str[:10])
    idx = (d - since_d).days // 7
    return idx if 0 <= idx < len(bins) else None

out = {}
for team in TEAMS:  # TEAMS from Step 1
    weekly = [{"week_start": str(s), "week_end": str(e), "partial": p,
               "merged_prs": 0, "deploys": 0} for s, e, p in bins]
    for pr in prs:
        if pr["team"] != team:
            continue
        idx = bucket(pr["mergedAt"])
        if idx is not None:
            weekly[idx]["merged_prs"] += 1
    seen_runs = set()
    for d in deploys:
        if d["team"] != team or d["run_id"] in seen_runs:
            continue
        idx = bucket(d["created"])
        if idx is not None:
            weekly[idx]["deploys"] += 1
            seen_runs.add(d["run_id"])
    out[team] = weekly

json.dump(out, open("weekly_bins.json", "w"), indent=2)
PYEOF
```

Dedupe deploy runs by `run_id` within each team when bucketing — the same
run may already appear once per team from Step 4's multi-team attribution;
don't let it inflate a single team's weekly deploy count.

### Step 7 — Pearson correlation per team

For each team, correlate its `weekly_bins.json` `merged_prs` series against
its `deploys` series:

```bash
python3 - <<'PYEOF'
import json, statistics

bins = json.load(open("weekly_bins.json"))
results = {}
for team, weekly in bins.items():
    xs = [w["merged_prs"] for w in weekly]
    ys = [w["deploys"] for w in weekly]
    n = len(xs)
    if n < 2 or statistics.pstdev(xs) == 0 or statistics.pstdev(ys) == 0:
        results[team] = {"r": None, "n": n}
        continue
    mx, my = statistics.mean(xs), statistics.mean(ys)
    cov = sum((x - mx) * (y - my) for x, y in zip(xs, ys))
    r = cov / (n * statistics.pstdev(xs) * statistics.pstdev(ys))
    results[team] = {"r": round(r, 2), "n": n}

json.dump(results, open("correlation.json", "w"), indent=2)
print(results)
PYEOF
```

**Small-n caveat:** with the default 30-60 day window this is only ~4-9
weekly points per team. Report `r` **together with n** every time it's shown,
and for any team with `n < 6` say "too few weeks for a stable correlation"
rather than characterizing the strength (don't call an r=0.7 on 4 points
"strong").

### Step 8 — Emit the weekly PR/deploy scatter chart

Build one HTML file: a scatter plot, one point per team per week
(x = that week's `merged_prs`, y = that week's `deploys`), following the
`dataviz` skill's procedure in order:

- **Form** — scatter (`choosing-a-form.md`: "tell distinct series apart" with
  4 series → categorical color). Both axes are counts of the same unit, so
  there is no dual-axis risk here (that anti-pattern only applies to two
  *different* units sharing a plot).
- **Color** — reuse the 4 categorical slots already validated for these
  4 teams in the prior weekly-trend chart (slot 1 blue = Partnerships, slot 2
  green = Deals, slot 3 magenta = New Verticals, slot 4 yellow = MarTech).
  Don't re-derive a palette each run.
- **Secondary encoding (required)** — a scatter needs the *all-pairs* CVD
  check, not just adjacent, and the dark-mode run of these 4 slots sits in
  the 6-8 floor band. Give each team a distinct marker **shape** (circle /
  triangle / square / diamond) in addition to color, so identity survives
  even where hue alone doesn't clear the floor.
- **Marks** — ≥24px transparent hit-area per point (n is small, no need for
  a Voronoi/nearest-point layer), 2px surface ring on markers so overlapping
  points stay legible, no number labels on the points themselves.
- **Hover** — per-point tooltip on `pointermove`/`focus` (not a shared
  crosshair — points aren't aligned on one axis) showing team, week range,
  merged-PR count, deploy count.
- **Legend** — always present (4 series): swatch + shape glyph per team.
- **Correlation readout** — show each team's `r` (with `n`) as a small stat
  row above the plot, using text tokens (never colored text) with a colored
  swatch beside each team's row for identity.
- **Table view** — same Chart/Table toggle pattern as the existing weekly
  line chart; the table lists week / team / merged PRs / deploys so the data
  is reachable without hovering.
- **Validate** — run `node scripts/validate_palette.js` (from the `dataviz`
  skill's base directory) against the 4-slot palette with `--pairs all`
  (scatter needs all-pairs, not adjacent) for both light and dark mode before
  shipping; confirm the shape encoding is in place for any WARN-band pair.
- **Look at it** — open the rendered HTML and check for point overlap,
  legend collisions, and label overflow before calling this step done.

Output path: `/Users/skasula/source-code/markdown-files/Team Deployment
Frequency - PR-Deploy Correlation - YYYY-MM-DD.html` (same directory and
naming convention as the existing weekly-trend chart).

## Notes & gotchas
- Multi-team members count toward each team — add a footnote.
- The `partnerships` GitHub team overlaps with `partnership-deals` and
  `partnership-new-verticals` (see Step 1) — it is not a clean "core only"
  roster. Use the fixed `partnership-core` list (Step 1) when the user wants
  partnerships work isolated from deals/new-verticals crossover.
- Pearson `r` (Step 7) is only as good as its `n` — always show it alongside
  the week count, and don't characterize correlation strength below n=6.
- Reuse the already-validated 4-slot categorical palette (Step 8) rather than
  re-deriving colors each run; if `partnership-core` is added as a 5th
  plotted series, re-run the validator with `--pairs all` before assuming
  slot 5 clears the CVD floor alongside the other four.
- `gh search prs` does not OR-combine multiple `--author` flags — see Step 2.
- The `branch` filter may miss tag- or `workflow_dispatch`-triggered deploys;
  if so, drop it and confirm `head_branch == default_branch` per run object.
- `/commits/{sha}/pulls` maps squash-merges cleanly; spot-check known deploys.
- Always fetch workflow runs with `--paginate` — see Step 3; a bare fetch caps
  at 100 runs per workflow and silently under-counts deploy frequency for any
  workflow that deploys more often than that in the window.
- A large share of deploy runs (often the majority) won't map to any of your
  tracked teams — that's expected on a shared org where far more people touch
  a repo than the handful on your tracked team rosters, not a data-quality bug.
  Deployment frequency here is pooled per-team by design, not per-repo.
- Rate limits: querying many members'/repos' data back-to-back can trip
  GitHub's *secondary* rate limit (HTTP 403, distinct from the primary 5000/hr
  limit) even with budget remaining. Throttle with a small `sleep` between
  calls (Steps 2-4); on a 403, wait ~45s and retry just the failed items.
- Verify DEPLOY_INCLUDE against `gh api repos/$ORG/$r/actions/workflows` first.
