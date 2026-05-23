---
type: gotchas
project: shopify-oss-general
ecosystem: shopify
tags: [oss, gotchas, pr-time, shopify, general]
---

# Shopify OSS PR Gotchas (general / ecosystem-level)

> Lessons that apply across multiple Shopify OSS repos (Shopify-AI-Toolkit, shop-chat-agent, agent-skills, ai-agent-partner-sales-channel-template, etc.). Check before every push to any Shopify OSS repo without a more specific gotchas file. Add new entries as you ship more PRs.

> Per-repo gotchas (when material exists): see [theme-tools-PR-Gotchas](./theme-tools-PR-Gotchas.md). New per-repo files get created as repo-specific lessons accumulate.

> Related: [Research-Gotchas](../../core/Research-Gotchas.md) for repo-qualification gotchas (gate 1, runs BEFORE starting work). This file is gate 2 (PR-time, runs BEFORE pushing). For the general contribution workflow, see [shopify-oss-contribution](./shopify-oss-contribution.md).

## Pre-Push Checklist (run before every `gh pr create`)

- [ ] Identity: `git config user.email` returns `<your-email>`
- [ ] Identity: `git log -1 --format='%ae'` returns `<your-email>`
- [ ] `git diff | grep -iE "<your-employer>|partner|<work-partner-id>|<work-handle>"` returns empty
- [ ] No private paths in diff: `git diff | grep -E "/Users/<your-github>/|Obsidian-vault|<your-vault>"` returns empty
- [ ] No API keys / tokens / Shopify app credentials in diff
- [ ] No new dependencies in bug-fix PRs (unless unavoidable)
- [ ] PR scoped narrowly to the filed issue -- no scope expansion
- [ ] No renames / API changes bundled with a bug fix (do as separate `feat/`)
- [ ] No em dashes in commit / PR body
- [ ] No Co-Authored-By AI line

## Gotchas

### 1. Scope-debate / rename PRs get closed unmerged -- 2026-04-18

**What happened:** PR #17 (`tizmagik`, external) to Shopify-AI-Toolkit was closed unmerged. It was a rename proposal -- not a bug fix with a clear repro.

**Why:** Shopify maintainers value scoped, low-friction changes. Renames touch all callers and invite debate about naming preference. Without prior maintainer buy-in, they read as opinionated rather than helpful.

**Check next time:** Stick to bug fixes with clear repros and acknowledged issues until 2-3 PRs have merged from your account. After establishing trust, opinion-heavy PRs (renames, refactors, scope expansion) have a better chance -- but still file an issue first to validate direction.

**Source:** PR #17 (tizmagik, external precedent).

### 2. Don't `npm install` at Shopify-AI-Toolkit root -- 2026-04-18

**What happened:** Issue #11 documents that some skills' `package-lock.json` files point to Shopify's private `npm.shopify.io` registry. A root-level `npm install` will fail or pollute lockfiles.

**Check next time:** Install only inside specific skill directories that use the public registry. Verify lockfile registry before running install: `grep -i shopify.io <pkg>/package-lock.json` -- if it returns matches, that directory's lockfile is private-only.

**Source:** Issue #11 in Shopify-AI-Toolkit.

### 3. Identity firewall: Partner ID <your-id> ≠ personal contribution -- 2026-04-18

**What happened:** Pre-flight identity audit before first Shopify OSS PR. Partner ID <your-id> ties to <your-employer>; personal contributions must use `<your-github>` / `<your-email>` ONLY.

**Why:** Mixing identities in Shopify-adjacent OSS contaminates the personal GitHub track record with work-account attribution. Same firewall that protects personal projects in `<your-code-path>/`.

**Check next time:** For ANY Shopify OSS commit: verify `git config user.email` and `git log -1 --format='%ae'` are personal. Diff scan: `git diff | grep -iE "<your-employer>|partner|<work-partner-id>|<work-handle>"` must be empty. Never reference Partner ID in issue comments or PR bodies.

**Source:** PR #18 (Shopify-AI-Toolkit, pre-flight).

### 4. Bug-fix PRs with failing-test-first pattern merge fastest -- 2026-04-18

**What happened:** Pattern observed across externals merged into theme-tools (aswamy, andershagbard, graygilmore, FKauwe, NathanPJF) and shop-chat-agent: bug fixes that add a failing test FIRST, then add the fix, get merged within days.

**Why:** Maintainers can run the test before the fix, see it fails, run after, see it passes -- they don't have to reproduce the bug themselves. Cuts review burden dramatically.

**Check next time:** For bug fixes: write the failing test first, commit it (or include in the same commit). Make this the default pattern, not a "nice to have."

**Source:** Merged-PR audit across theme-tools, shop-chat-agent (2026-04-18).
