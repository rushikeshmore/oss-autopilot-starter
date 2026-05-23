---
type: gotchas
project: gutenberg
ecosystem: wordpress
tags: [oss, gotchas, pr-time, gutenberg, wordpress]
---

# Gutenberg PR Gotchas

> Lessons from real PRs to WordPress/gutenberg that needed rework or were closed. Check before every push. Add new entries as you ship more PRs.

> Related: [Research-Gotchas](../../../core/Research-Gotchas.md) for repo-qualification gotchas (gate 1, runs BEFORE starting work). This file is gate 2 (PR-time, runs BEFORE pushing). For the general contribution workflow, see [gutenberg-contribution](../_skills/gutenberg-contribution.md).

## Pre-Push Checklist (run before every `gh pr create`)

- [ ] CHANGELOG entry under `## Unreleased > ### Bug Fixes` (or `### Internal` for refactors); never under a version-numbered heading
- [ ] `package-lock.json` NOT in `git diff --name-only` (`git checkout HEAD -- package-lock.json` if leaked)
- [ ] No Co-Authored-By line in commits: `git log origin/trunk..HEAD --format='%B' | grep -i 'co-authored'` empty
- [ ] AI Disclosure section completed in PR body (REQUIRED by WordPress)
- [ ] PR template filled: What / Why / How / Testing / Testing for Keyboard / AI Disclosure
- [ ] Branch named `[type]/[change]` per convention (`fix/`, `refactor/`, `add/`, `update/`, `remove/`, `try/`)
- [ ] All scoped tests pass: `npm run test:unit -- --testPathPattern "<scope>"`
- [ ] `git config user.email` returns `<your-email>`
- [ ] `git log -1 --format='%ae'` returns `<your-email>`

## Gotchas

### 1. CHANGELOG must be `## Unreleased > ### Bug Fixes` -- 2026-05-02

**What happened:** PR #77181 round-2: entry placed under `## 32.6.0`. @ciampo moved it to `## Unreleased > ### Bug Fixes`.

**Why:** Version-numbered headings get filled by the release process; contributors only add to Unreleased. Sub-section choice (Bug Fixes / Internal / Enhancements) drives release-notes categorization.

**Check next time:** Always `## Unreleased`. Pick: `### Bug Fixes` for fixes, `### Internal` for pure refactors, `### Enhancements` for features.

**Source:** PR #77181.

### 2. CHANGELOG PR-link added in follow-up commit -- 2026-04-09

**What happened:** Three PRs filed same day (#77181, #77183, #77184): CHANGELOG entries committed without PR links (numbers unknown at commit time). Required follow-up commits.

**Check next time:** Either accept the follow-up pattern, OR use `gh pr create --draft` first to get the number, then amend the CHANGELOG.

**Source:** PRs #77181, #77183, #77184.

### 3. `package-lock.json` leaks from `npm install` -- 2026-04-09

**What happened:** PR #77171 audit caught `package-lock.json` in the staged commit, leaked from a routine `npm install` during development.

**Check next time:** `git diff --name-only HEAD~1` before push. If lockfile shows, `git checkout HEAD~1 -- package-lock.json` and amend.

**Source:** PR #77171.

### 4. Co-Authored-By in commits -- 2026-04-10

**What happened:** 3 Gutenberg branches had `Co-Authored-By: Claude Opus 4.6` lines. Required `git filter-branch` + force push on all 3 to clean.

**Why:** AI disclosure for WordPress lives in the PR body (`## Use of AI Tools` section), NOT in commits. Per-user preference: no AI attribution in commit metadata.

**Check next time:** Pre-push: `git log origin/trunk..HEAD --format='%B' | grep -i 'co-authored'` must be empty.

**Source:** PRs #77181, #77183, #77184.

### 5. Direction-decline risk on enhancement / stabilization PRs -- 2026-04-14

**What happened:** #77183 (Composite context validation) and #77184 (BorderControl popoverProps) both closed by @ciampo without code review. Halted pending different strategies in tracking issues #66530 and #65581.

**Why:** Enhancement and "stabilize unstable API" PRs touch architectural direction. Maintainers may have a different plan that isn't yet documented.

**Check next time:** For "enhancement", "stabilization", or "new API" PRs, post on the tracking issue FIRST with the proposed approach. Wait for maintainer ack before opening the PR. Bug fixes with clear repros are the safe first-contribution pattern.

**Source:** PRs #77183, #77184.

### 6. Trace ALL code paths, not just the one in the issue -- 2026-05-12

**What happened:** PR #77181 round-2: applied unconditional comma rejoin to fix the paste-validation case. Broke the typing-space UX in `tokenizeOnSpace` mode (typing `'hello '` lost trailing space). @ciampo pushed back in round 3.

**Why:** Optimized for one path (paste) without tracing the other (typing). Both flow through `onInputChangeHandler` but have different separator-trail expectations.

**Check next time:** For input / event-handling fixes, trace EVERY entry point (paste, typing, programmatic). Add tests for each. Before pushing, ask: would a reviewer flag the path I didn't test?

**Source:** PR #77181 rounds 2-3.

### 7. Static Analysis CI failure -> rebase on trunk first -- 2026-05-09

**What happened:** PR #77181 had persistent Static Analysis CI failures. Local branch was 5 ahead, 122 behind upstream/trunk. Rebase resolved them.

**Why:** Trunk's lint plugins / static analysis configs drift forward; long-open branches drift behind. CI catches what local doesn't.

**Check next time:** If Static Analysis or lint CI fails and you haven't rebased recently: `git fetch upstream && git rebase upstream/trunk && npm install` before investigating the failure itself.

**Source:** PR #77181.

### 8. CHANGELOG rebase conflicts -- 2026-05-09

**What happened:** Rebase of #77181 hit CHANGELOG conflict in the `## 33.0.0` block. Trunk added `### Internal` there during the release shuffle.

**Why:** Multiple PRs land in `## Unreleased`; the release process moves them to a version section; your branch still has them under Unreleased.

**Check next time:** Keep trunk's structure on rebase. Place your entry under `## Unreleased > ### Bug Fixes` of the rebased file. Don't re-add to the version-numbered section.

**Source:** PR #77181.

### 9. Combined commits OK when refactor + tests interleave -- 2026-05-12

**What happened:** Round-2 changes touched `index.tsx` refactor + 4 test changes that conceptually interleave (tests assert behavior the refactor changes). Decided: ONE combined commit + per-thread replies instead of splitting via `git add -p`.

**Check next time:** For round-N review responses where multiple threads touch overlapping lines, one combined commit is fine. Document the mapping in the commit body and via per-thread replies. Only split when changes are truly independent.

**Source:** PR #77181.
