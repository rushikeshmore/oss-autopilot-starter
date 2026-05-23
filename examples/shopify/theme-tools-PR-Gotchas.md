---
type: gotchas
project: theme-tools
ecosystem: shopify
tags: [oss, gotchas, pr-time, theme-tools, shopify]
---

# theme-tools PR Gotchas

> Lessons from real PRs to Shopify/theme-tools that needed escalation or strategic re-approach. Check before every push. Add new entries as you ship more PRs.

> Related: [Research-Gotchas](../../core/Research-Gotchas.md) for repo-qualification gotchas (gate 1, runs BEFORE starting work). This file is gate 2 (PR-time, runs BEFORE pushing). For the general contribution workflow, see [theme-tools-contribution](./theme-tools-contribution.md). For Shopify OSS-wide gotchas, see [shopify-oss-PR-Gotchas](./shopify-oss-PR-Gotchas.md).

## Pre-Push Checklist (run before every `gh pr create`)

- [ ] `yarn build` + `yarn test` + `yarn workspaces run type-check` + `yarn format:check` all clean
- [ ] `yarn changeset` file committed with correct bump level (patch for bug fix, minor for new check)
- [ ] Bug reproduced by a NEW failing test BEFORE the fix; fix makes it pass
- [ ] No new dependencies in bug-fix PRs
- [ ] No public-API renames in bug-fix PRs (do as separate `feat/` PR)
- [ ] `git config user.email` returns `<your-email>`
- [ ] `git log -1 --format='%ae'` returns `<your-email>`
- [ ] No <your-employer> / Partner ID <your-id> / `<work-handle>` refs in diff

## Gotchas

### 1. Long review wait -- escalation cadence required -- 2026-05-09

**What happened:** PR #1182 (Liquid style-tag parsing fix): CLA + mergeability green from Day 1, 0 reviews through Day 14, Day-14 nudge sent May 2, Day-21 maintainer tag (@charlespwd + @graygilmore) sent May 9. Still awaiting first human review at Day 21+.

**Why:** Shopify maintainers triage many repos; even clean bug-fix PRs need explicit visibility. The active-reviewer list in [theme-tools-contribution](./theme-tools-contribution.md) (`graygilmore`, `charlespwd`, `aswamy`, `mgmanzella`) exists for this reason.

**Check next time:** Build a Day-7 / Day-14 / Day-21 cadence into your tracking. Tag specific maintainers from the active-reviewer list at Day 21+, not random ones.

**Source:** PR #1182.

### 2. Tag active reviewers from the maintainer list, not random -- 2026-05-09

**What happened:** Day-21 escalation on PR #1182 specifically tagged @charlespwd and @graygilmore -- both had shipped externals to this repo within the prior 60 days. Avoided tagging maintainers whose recent activity was in other Shopify repos.

**Check next time:** Active reviewers in theme-tools (verified via merged-PR audit): `graygilmore`, `charlespwd`, `aswamy`, `mgmanzella`. Tag from this list. Re-verify every 90 days -- maintainer rotations happen.

**Source:** PR #1182.
