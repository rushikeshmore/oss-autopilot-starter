---
type: gotchas
project: openverse
ecosystem: wordpress
tags: [oss, gotchas, pr-time, openverse, wordpress]
---

# Openverse PR Gotchas

> Lessons from real PRs to WordPress/openverse that needed escalation or strategic re-approach. Check before every push. Add new entries as you ship more PRs.

> Related: [Research-Gotchas](../../core/Research-Gotchas.md) for repo-qualification gotchas (gate 1, runs BEFORE starting work). This file is gate 2 (PR-time, runs BEFORE pushing). For the general contribution workflow, see [openverse-contribution](./openverse-contribution.md).

## Pre-Push Checklist (run before every `gh pr create`)

- [ ] PR template filled with testing instructions (REQUIRED)
- [ ] `ov just lint` passes
- [ ] `pnpm -r run test` passes (or `ov just frontend/run test:unit`)
- [ ] No Playwright snapshot changes unless intentional
- [ ] For accessibility changes: tested with VoiceOver on macOS
- [ ] For SSR-touching changes: tested both page reload AND client-side nav
- [ ] `git config user.email` returns `<your-email>`
- [ ] `git log -1 --format='%ae'` returns `<your-email>`

## Gotchas

### 1. Review can stall 30+ days -- escalation cadence is mandatory -- 2026-05-09

**What happened:** PR #5570 (audio thumbnail accessibility): Day-7 no comment, Day-14 still nothing, Day-23 first nudge with maintainer tags (@krysal @obulat), Day-30 still no review -> Slack #openverse outreach planned.

**Why:** Openverse has a small maintainer team and a long backlog. Quiet PRs disappear without escalation.

**Check next time:** Set calendar reminders at Day-7 (polite PR comment), Day-14 (Slack #openverse on chat.wordpress.org), Day-21+ (maintainer tag). Don't wait passively.

**Source:** PR #5570.

### 2. Label CI checks fire BEFORE maintainer applies labels -- 2026-05-09

**What happened:** PR #5570 showed CI failures for `aspect`, `priority`, `stack` label checks. The checks ran BEFORE the maintainer applied the labels. PR was fine; checks were stale.

**Check next time:** If label-related CI checks fail on a fresh PR, it's likely a false failure -- wait for a maintainer to apply labels and re-run. Don't change code based on a label-check failure.

**Source:** PR #5570.

### 3. Issue body's first suggested approach often isn't the final one -- 2026-04-09

**What happened:** Issue #497 originally suggested "add better alt text" for audio thumbnails. Maintainer discussion evolved the approach to `aria-hidden="true"` (decorative image, hide from screen readers entirely).

**Why:** Issue bodies capture the FIRST framing of a problem. Comments often refine the approach. The issue may be open BECAUSE the original framing was wrong.

**Check next time:** Read the FULL comment thread (not just the body) before implementing. Especially for accessibility, design, and architectural issues -- the right approach is often in a maintainer reply, not the issue description.

**Source:** PR #5570 / Issue #497.

### 4. Day-21+ escalation: tag specific reviewers -- 2026-05-02

**What happened:** Day-23 of PR #5570 with no reviews. Pulled requested-reviewer list from GitHub API (`@krysal`, `@obulat`) and tagged both with a low-pressure ask per Day-21+ cadence.

**Check next time:** When escalating past Day-21, identify maintainers via `gh api repos/WordPress/openverse/pulls/<N>` requested_reviewers, or check CODEOWNERS. Don't tag at random.

**Source:** PR #5570.
