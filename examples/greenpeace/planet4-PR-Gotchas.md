---
type: gotchas
project: planet4
ecosystem: greenpeace
tags: [oss, gotchas, pr-time, planet4, greenpeace]
---

# planet4 PR Gotchas

> Lessons from real PRs to greenpeace/planet4-master-theme that needed rework, were soft-declined, or required maintainer pushback. Check before every push. Add new entries as you ship more PRs.

> Related: [Research-Gotchas](../../core/Research-Gotchas.md) for repo-qualification gotchas (gate 1, runs BEFORE starting work). This file is gate 2 (PR-time, runs BEFORE pushing). For the general contribution workflow, see [planet4-contribution](./planet4-contribution.md).

## Pre-Push Checklist (run before every `gh pr create`)

- [ ] `composer run phpcs` clean (PHP changes)
- [ ] `npm run lint` clean (JS changes)
- [ ] `npm run stylelint` clean (CSS changes)
- [ ] For license / metadata changes: all 4 files synced -- `package.json`, `package-lock.json`, `composer.json`, `style.css`
- [ ] `composer validate` clean (composer.json edits)
- [ ] For dependency removal: build matches baseline (capture warnings + errors count BEFORE; verify AFTER)
- [ ] Commit amended (not new) for review feedback; force-push to update
- [ ] No em dashes in commit subject/body
- [ ] No Co-Authored-By AI line
- [ ] Single-commit PRs inherit commit name (no separate title needed)
- [ ] `git config user.email` returns `<your-email>`
- [ ] `git log -1 --format='%ae'` returns `<your-email>`
- [ ] Self-audit: re-read the diff line by line, would a maintainer flag it?

## Gotchas

### 1. License metadata must sync across 4 files -- 2026-05-02

**What happened:** PR #2962 (license fix) only updated `package.json`. @comzeradd asked for `package-lock.json` root entry sync too. Verified `style.css` and `composer.json` already had the canonical form.

**Why:** Repos that publish to multiple registries / package managers carry license metadata in multiple places. Mismatch fails `composer validate` and confuses downstream consumers.

**Check next time:** For any license change in planet4-master-theme, update `package.json` + `package-lock.json` (root entry) + `composer.json` + `style.css`. Run `composer validate` to confirm.

**Source:** PR #2962.

### 2. Use `GPL-3.0-or-later`, not `GPL-3.0` -- 2026-04-23

**What happened:** PR #2962 originally used `GPL-3.0`. `composer validate` flagged it as a deprecated SPDX identifier.

**Why:** SPDX deprecated bare `GPL-3.0` in favor of `GPL-3.0-or-later` / `GPL-3.0-only`. Planet 4's LICENSE file says "or any later version" -> use `-or-later`.

**Check next time:** Whenever specifying an SPDX license string: use `GPL-3.0-or-later` for "or any later version" wording; `GPL-3.0-only` if LICENSE is strictly v3.

**Source:** PR #2962.

### 3. PHPUnit test file naming: `test-*.php`, not `*Test.php` -- 2026-05-02

**What happened:** PR #2961 originally added a `phpunit.xml.dist` discovery rule for `*Test.php` files. @sagarsdeshmukh asked to use the existing `test-*.php` convention instead. Renamed: `tests/SqlParametersTest.php` -> `tests/test-sql-parameters.php`.

**Why:** Project conventions trump generic standards. Planet 4 standardized on `test-*.php` long ago.

**Check next time:** For PHPUnit test files in planet4: file name = `test-<class-name-kebab>.php`. Class name inside the file CAN keep its `<Class>Test` style -- only the file name needs to match convention.

**Source:** PR #2961.

### 4. Don't change `phpunit.xml.dist` if existing convention works -- 2026-05-02

**What happened:** PR #2961 round 1 added a second discovery rule to `phpunit.xml.dist`. Round 2 dropped that change -- the existing `test-*.php` rule already worked.

**Check next time:** Before changing test / CI config, check whether the existing config already handles your case. Renaming files to fit conventions is almost always preferred over changing the convention.

**Source:** PR #2961.

### 5. AGENTS.md / meta-files: soft-decline risk -- 2026-05-02

**What happened:** PR #2964 added an AGENTS.md with no automation, purely AI-coding-assistant guidance. @comzeradd soft-declined: *"we don't want to invite/encourage fully automated AI agents PRs."* Closed politely.

**Why:** Meta-files (AGENTS.md, CONTRIBUTING additions, CODE_OF_CONDUCT, .github templates) are political. Each project has its own stance on AI tooling. Adding one unsolicited signals "I think you should adopt my opinion."

**Check next time:** For meta-files in any OSS project, ask FIRST in an issue: *"Would you be open to a PR adding X?"* Wait for explicit yes. Bug fixes don't need this gate; opinions / new docs / governance files do.

**Source:** PR #2964.

### 6. Removing CRA `react-scripts` may break transitive deps -- 2026-04-23

**What happened:** PR #2960 first attempt broke the build because `frontendIndex.js` relied on `regenerator-runtime/runtime` as a transitive of `react-scripts`. Fix: add `regenerator-runtime@^0.14.1` as explicit dep, remove the stale `eslint-disable-next-line import/no-extraneous-dependencies` comment.

**Why:** CRA-era code often imports utility packages without declaring them -- they came in via `react-scripts`. Removing the parent breaks the import chain.

**Check next time:** Before removing any large parent dependency (`react-scripts`, `@vue/cli-service`, similar), grep the codebase for imports of its known transitives (`regenerator-runtime`, `core-js`, polyfills). Add as explicit deps BEFORE removing the parent.

**Source:** PR #2960.

### 7. Verify build matches baseline before / after dependency changes -- 2026-04-23

**What happened:** After fixing PR #2960's transitive issue, verified the build produced 84 warnings, 0 errors -- matching the pre-PR baseline. Without this check, a regression could have been missed.

**Check next time:** For any `package.json` change: run `npm run build` BEFORE the change, capture warning + error count. Run AFTER. Counts must match (or be lower).

**Source:** PR #2960.

### 8. Rebase `package-lock.json` conflicts: regenerate or `--theirs` -- 2026-05-15

**What happened:** PR #2960 was 21 commits behind on `main` rebase. `package-lock.json` conflict. Resolved with `--theirs` + intent preserved + force-push.

**Check next time:** For long-open PRs that touch `package-lock.json`: on rebase conflict, either (a) `git checkout --theirs package-lock.json && npm install` to regenerate, or (b) accept upstream's lockfile and re-apply your dep change on top.

**Source:** PR #2960.

### 9. Child theme repos may be dormant >13 months -- 2026-05-09

**What happened:** PR #22 to `planet4-child-theme-korea` sat 27 days with no review. Repo last pushed 2025-03-17 (>13 months dormant). Closed per stale-PR SOP.

**Check next time:** Before contributing to any planet4 child theme, check the last push date: `gh repo view greenpeace/<repo> --json pushedAt`. If >6 months dormant, treat as "may be abandoned" and prioritize active repos.

**Source:** PR #22 (Korea child theme).

### 10. Ship independent PRs per fix, not bundled -- 2026-04-23

**What happened:** 6 PRs filed in one day to planet4-master-theme. Each kept independent rather than bundling.

**Why:** Reviewer can merge/reject each cleanly. License fix and dead-dep removal have different review audiences. Bundling means one objection blocks everything.

**Check next time:** Default to one PR per logical fix. Only bundle when changes are tightly coupled and can't ship independently.

**Source:** Apr 23 batch (#2959, #2960, #2961, #2962, #2963, #2964).

### 11. Self-audit overclaims before push -- 2026-04-23

**What happened:** PR #2964 (AGENTS.md draft) claimed "commitlint runs on pre-commit hook." Actually only CI enforces it; `.husky/pre-commit` runs lint-staged only. Caught during a final read-through before commit.

**Check next time:** For any PR that documents project behavior (READMEs, AGENTS.md, contributor guides), verify each claim against the actual config files (`.husky/`, `.github/workflows/`, `package.json` scripts). Don't rely on what you assumed when you started writing.

**Source:** PR #2964.
