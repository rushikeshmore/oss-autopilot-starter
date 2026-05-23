---
type: reference
created: 2026-04-18
tags: [open-source, research, gotchas, lessons, github]
---

# Open Source Research Gotchas

Catalogue of research mistakes specific to open-source / GitHub work (repo targeting, PR strategy, schema/spec fixes, issue triage) and the checks that would have caught them. Every entry is a real failure with its root cause, not a theoretical risk. When a new gotcha is found, add it here before fixing the immediate damage.

Scope: OSS only. Non-OSS research failures (finance, career, AI-work planning) belong in topic-specific gotchas files under their own folders, not here.

## How to use this file

- **Before starting a research task** → skim the relevant section's checklist.
- **Before writing a plan based on research** → run the checks listed in the most relevant gotcha.
- **After a failure** → add a new gotcha entry with the three-part template below, inside 24h while the memory is fresh.

Rule of thumb: if the check is cheap (< 30 seconds) and the consequence of skipping it is real (wasted PR, bad investment, flawed plan) — it belongs here.

## Template for a new gotcha

```
### <N>. <Short name> — <date>

**What happened:** One sentence on the mistake.
**Where it landed:** Which file / PR / plan absorbed the bad data.
**Root cause:** The single check that would have caught it.
**Fix rule (durable):** One-line rule to apply forever.
**Detection command / procedure:** Exact shell command or steps.
**Cost of the check:** seconds/minutes.
**Cost of skipping:** what the failure actually cost.
```

---

## Gotchas

### 1. "PR exists" vs "PR merged" — 2026-04-18

**What happened:** Treated Shopify/Shopify-AI-Toolkit as a viable external-contribution target. Planned 4 PRs. The first PR (#18 fixing issue #10) was auto-closed in seconds by the repo's `close-all-prs.yml` bot. Vault scaffolding, skill guide, memory entry, and a full sequencing plan were all built on a false premise.

**Where it landed:** `<your-ecosystem-overview>` Planned-First-PRs table, the relevant contribution skill, the relevant memory file, ecosystem-index `the OSS folder index`. All had to be rewritten after the bot closed PR #18.

**Root cause:** Three compounding mistakes, not one.

1. **Read PR titles, not PR states.** PR #9 on the repo was titled *"Temporarily disable auto-close PRs workflow"*. I assumed it had landed. It was closed unmerged on 2026-04-07 — `mergedAt: null`. I never checked the field.
2. **Never opened `.github/workflows/`.** The file `close-all-prs.yml` is literally named what it does. I checked license, stars, star growth, maintainer activity, issues list, PR responsiveness — everything except the one directory that decides whether PRs are even read.
3. **Mistook timing coincidence for policy.** Two externals (austin-schaefer #3, shankarchandran #8) merged Apr 6–7. The bot was introduced Apr 1 and tightened Apr 2 to "allow Shopify employees only". Those merges happened in the bot's enforcement gap. I took them as evidence of an open-to-externals pattern. The silence from Apr 7 onward was the real signal and I didn't ask why.

**Meta root cause:** I optimized for *"is this a welcoming repo"* (qualitative) when I should have first optimized for *"is this a repo that merges non-employees at all"* (binary). A binary precondition must be verified before any qualitative grading.

**Fix rule (durable):**
Before targeting any GitHub repo for an external contribution, run the three-gate check. All three must pass.

**Detection: 3-gate OSS repo qualification**

```bash
ORG=Shopify; REPO=Shopify-AI-Toolkit   # set these

# Gate 1 — no auto-close / gatekeeping workflow
gh api repos/$ORG/$REPO/contents/.github/workflows --jq '.[].name' 2>/dev/null \
  | grep -iE 'close.*pr|external.*pr|auto.?reject' \
  && echo "BLOCKED: auto-close workflow present" || echo "gate 1 OK"

# Gate 2 — at least one non-bot, non-org-member PR merged in the last 50
gh pr list --repo $ORG/$REPO --state merged --limit 50 \
  --json number,author,mergedAt \
  --jq '.[] | select(.author.is_bot | not)
              | select(.author.login | test("shopify|dependabot|github-actions|renovate"; "i") | not)
              | "\(.mergedAt[0:10])  \(.author.login)  #\(.number)"' \
  | head -5
# Expect at least one row dated within last 90 days. Zero rows = gate 2 fail.

# Gate 3 — any PR labeled "proposal to disable auto-close" / "accept external PRs"
#          must be in MERGED state, not just OPEN or CLOSED.
gh pr list --repo $ORG/$REPO --state all --search 'auto-close OR external PR' \
  --json number,title,state,mergedAt
# If the proposal exists but is CLOSED unmerged → treat as strong negative signal.
```

**Cost of the check:** ~30 seconds total.
**Cost of skipping:** ~3 hours spent on vault scaffolding, skill guide, first PR, + reputational noise of a closed PR attached permanently to the personal GitHub identity that is being used as the dotdev networking asset.

---

### 2. "Fix suggested in issue body" ≠ correct fix — 2026-04-18

**What happened:** Issue #10 on Shopify-AI-Toolkit suggested fixing `marketplace.json` with `{"source": "local", "path": "./"}`. The suggestion was wrong — `"local"` is not a valid source type in the current Claude Code plugin schema. Only `github`, `url`, `git-subdir`, `npm`, or a relative-path string are valid. A mechanical copy-paste of the issue's suggestion would have produced a PR that fixed nothing and signalled that the author didn't read the schema.

**Where it landed:** Almost landed in PR #18. Caught before commit by running a dedicated schema-verification subagent that pulled the canonical `code.claude.com/docs` source of truth. The PR shipped with the correct `{"source": "github", ...}` form instead, and the PR body explicitly called out that the issue's suggested fix was schema-invalid.

**Root cause:** Trusted a non-authoritative source (the issue reporter's comparative analysis with another plugin) over the authoritative source (the plugin-marketplaces schema doc). The reporter was confident and detailed, which is not the same as being right.

**Fix rule (durable):**
For any spec, schema, API, or protocol change: verify the fix against the upstream canonical doc, not against precedents cited in the issue. Precedents may predate the change or misquote the schema. Quote the canonical source in the PR body.

**Detection:** Every schema/spec PR must include a link to the official doc that defines the valid shape, not just a link to another project that happens to use a similar shape.

**Cost of the check:** 1 tool call to fetch the canonical doc.
**Cost of skipping:** A second closed PR making the same mistake the issue already made, plus loss of credibility on the exact area (Claude Code / MCP) we are trying to signal expertise in.

---

### 3. "Recent activity" vs "external-accepting activity" — 2026-04-18

**What happened:** Ranked Shopify target repos by stars, recent commits, issue traffic, maintainer responsiveness. Never filtered the "recent merges" list for `author != org-member AND author.is_bot == false`. Would have caught the AI-Toolkit block in the first pass.

**Fix rule (durable):**
When assessing whether a repo accepts external contributions, the metric is **"number of non-bot, non-org-member PRs merged in the last 90 days"** — nothing else. Stars, commit frequency, maintainer activity are downstream signals that answer different questions.

**Detection:**
```bash
# Reusable snippet — drop in any repo qualification workflow
gh pr list --repo $ORG/$REPO --state merged --limit 50 \
  --json number,author,mergedAt,title \
  --jq --arg org "$ORG" '
    .[]
    | select(.author.is_bot | not)
    | select(.author.login | ascii_downcase
              | test($org | ascii_downcase) | not)
    | "\(.mergedAt[0:10])  \(.author.login)  #\(.number)  \(.title)"
  '
```

Count the rows in the last 90 days. Zero or one → repo is effectively closed. Five+ → repo is open. In between → needs manual look at whether the PRs are trivial (typo fixes only) vs. substantive.

---

### 4. Mistaking a memory claim for evidence — 2026-04-18

**What happened:** The previous session's memory entry (`project_shopify_oss_contribution.md` v1) stated as fact: *"Auto-close PRs workflow was temporarily disabled in PR #9, so external PRs now stay open."* That was a claim, not an observation — the PR was never verified as merged. This summary was then loaded as context into the autopilot plan and treated as a premise.

**Fix rule (durable):**
A memory entry that asserts the state of an external system is frozen at the moment it was written. Before acting on a memory claim that says "X repo accepts Y" or "feature Z is live", re-verify against the current state of the external system. The verification is almost always one `gh api` or `gh pr view` call.

Memory is a map, not the territory. The territory is the live repo.

**Detection:** When a memory claim says a specific PR / issue / branch landed, run `gh pr view <N> --repo <org>/<repo> --json state,mergedAt` before planning anything that depends on it.

---

## When to add to this file

- A plan was written based on a fact that turned out to be wrong → add.
- Time was lost chasing a false premise → add.
- A cheap check would have caught it → definitely add.

Skip:
- One-off typos or transient infra failures (those are logs, not gotchas).
- Subjective taste calls that turned out differently (those are judgment, not research).
