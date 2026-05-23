---
type: skill
name: theme-tools Contribution Guide
description: Coding standards, dev setup, changesets, PR flow, and testing for contributing to Shopify/theme-tools (Liquid checks, prettier plugin, language server, VS Code extension).
---

# theme-tools Contribution - Agent SOP

## Pre-flight (must pass before any work)
Run the 3-gate OSS qualification from `../../core/Research-Gotchas.md` §1.

Current state verified 2026-04-18:
- Gate 1 PASS: workflows are `ci.yml`, `cla.yml`, `release.yml`, `vscode-release.yml` — no auto-close.
- Gate 2 PASS: externals merged in last 60d — andershagbard (@grafikr) #1171, elia (@nebulab) #1157, FKauwe #1167, NathanPJF #1169, graygilmore #1168, aswamy #1172 / #1180.
- Gate 3 N/A: no auto-close proposal exists.

Re-verify before any new target cycle.

## Repository
- Upstream: https://github.com/Shopify/theme-tools
- Fork: `<your-github>/theme-tools`
- Local clone: `<your-code-path>/03-open-source/shopify/theme-tools`
- License: MIT
- Monorepo author: albert.chu@shopify.com

## Identity (CRITICAL)
- Personal GitHub only: `<your-github>` (<your-email>)
- Verify before every commit: `git config user.email` returns `<your-email>`
- No <your-employer> references anywhere
- No Shopify Partner ID <your-id> references
- CLA: sign via `cla.yml` workflow bot comment on first PR

## Stack
- Package manager: **yarn 1.22.22** (NOT pnpm, NOT npm)
- Node: **v22.14.0** (from `dev.yml`)
- Monorepo: yarn workspaces, packages under `packages/`
- Tests: **vitest**
- Linter/formatter: **prettier ^3.0.0**
- Typecheck: per-workspace `yarn type-check`
- Changesets: **@changesets/cli** — required for every PR that changes production code
- Build: webpack + ts-loader (vscode-extension), tsc elsewhere

## Packages (primary targets for PRs)
```
packages/
  liquid-html-parser/           # AST parser
  prettier-plugin-liquid/       # formatter
  theme-check-common/           # shared check engine + checks
  theme-check-node/             # CLI runner
  theme-check-browser/          # browser runner
  theme-check-docs-updater/     # docs sync
  theme-language-server-common/ # LSP core
  theme-language-server-node/   # node LSP host
  theme-language-server-browser/# browser LSP host
  codemirror-language-client/   # editor client
  vscode-extension/             # VS Code Shopify Liquid extension
  theme-graph/                  # theme graph primitives
  release-orchestrator/         # internal release tooling (skip)
  lang-*                        # TextMate/grammar
```

Check fixes almost always land in `theme-check-common/src/checks/<CheckName>/`.
Language-server fixes land in `theme-language-server-common/`.
Formatter fixes land in `prettier-plugin-liquid/`.
VS Code extension fixes land in `vscode-extension/src/`.

## Local Development Setup
```bash
# Fork on github.com (first time only)
cd <your-code-path>/03-open-source/shopify/
git clone git@github.com:<your-github>/theme-tools.git
cd theme-tools
git remote add upstream git@github.com:Shopify/theme-tools.git

# Identity sanity check
git config user.email  # must be <your-email>
[[ "$(git config user.email)" == "<your-email>" ]] || \
  git config user.email <your-email>

# Node version
nvm use 22 || nvm install 22

# Install + build (build is REQUIRED before tests per graygilmore comment on #1086)
yarn
yarn build
```

## Running the Full Local Check Suite
Nothing commits until all four pass.
```bash
yarn build              # required for cross-package tests to resolve
yarn test               # vitest across all packages
yarn workspaces run type-check
yarn format:check       # prettier --check
```

### Per-package focused testing
```bash
# Run one check's tests (faster iteration)
yarn workspace @shopify/theme-check-common test <CheckName>
# Or cd in:
cd packages/theme-check-common
yarn test --run src/checks/<CheckName>/index.spec.ts
```

### VS Code extension manual test
```bash
# From repo root
code .
# Then F5 to launch Extension Development Host
# Open a .liquid file and verify the bug reproduces / the fix lands
```

### Browser playground (for language-server work)
```bash
yarn playground
```

## Branch Naming
- `fix/<short-name>` for bug fixes (e.g. `fix/unused-assign-style-tag`)
- `feat/<short-name>` for new checks / features
- `docs/<short-name>` for docs-only
- `chore/<short-name>` for tooling

## Commit Messages
No repo-level enforced convention, but match the merged-PR style:
- Imperative subject line, capitalize first letter, no trailing period
- Under 72 chars
- Body explains why and how
- **No em dashes** — regular hyphens or rewrite
- **No Co-Authored-By AI lines**
- **No AI-sounding polish** — write like a developer
- Reference the issue in body: `Closes #<N>` or `Refs #<N>`

Example from merged PRs (aswamy #1172, andershagbard #1171):
```
fix: use createRequire for node_modules resolution in VSCode extension

<Paragraph on why the old require path failed in the packaged
extension and how createRequire resolves it>

Closes #<N>
```

## Changesets (REQUIRED per PULL_REQUEST_TEMPLATE.md)
Every PR that changes production code MUST include a changeset.

```bash
yarn changeset
# -> choose affected packages with space, enter
# -> patch for bug fix, minor for new check/feature, major never (maintainer call)
# -> type a one-line summary that will appear in CHANGELOG
# Commit the resulting .changeset/<slug>.md file with the code change.
```

Bump convention (from repo PR template):
- **Bug fix → patch** bump
- **New check / new feature → minor** bump, must be backward compatible
- **New check specifically**: also add to `allChecks` array in `src/checks/index.ts`, run `yarn build`, commit updated configs. If applicable, update `packages/theme-check-node/configs/theme-app-extension.yml`.

## PR Template (from .github/PULL_REQUEST_TEMPLATE.md)
Fill only the sections that apply. The template asks for:
```markdown
## What are you adding in this PR?
<What changed + the why. Use `fixes #N` or `solves #N` to auto-close.>

## What's next? Any followup issues?
<Follow-ups or leave blank>

## What did you learn?
<Optional>

## Before you deploy
- [ ] Checkbox that applies (new check / public API / bug fix)
- [ ] Changeset included (patch/minor per rule above)
```

No AI disclosure required (Shopify does not mandate it). Still write naturally.

## PR Flow
1. Fork on GitHub (first time only)
2. `git checkout -b fix/<topic> main`
3. Reproduce the bug with a failing test in the relevant package
4. Implement the fix
5. Run the full check suite: `yarn build && yarn test && yarn workspaces run type-check && yarn format:check`
6. `yarn changeset` → commit the generated markdown
7. `git push -u origin fix/<topic>`
8. `gh pr create --repo Shopify/theme-tools --base main --head <your-github>:fix/<topic>` with the filled template
9. Sign CLA on first PR (bot will comment with link)
10. Respond to review — prefer amend + force-push for small tweaks:
    `git commit --amend --no-edit && git push -f origin fix/<topic>`
11. Resolve conflicts via rebase, not merge:
    `git fetch upstream && git rebase upstream/main`

## Review expectations
- Maintainers present on recent merges: `graygilmore`, `charlespwd`, `aswamy`, `mgmanzella`
- Bug-fix PRs with failing-test-first pattern tend to merge fast
- Scope creep (renames, unrelated refactors) will get pushback — keep PRs narrow

## Coding Standards
- TypeScript, strict. Prefer explicit types at module boundaries.
- Match existing style in the file/package being edited. Do not introduce formatter changes.
- No new dependencies in bug-fix PRs.
- Prefer explicit imports over wildcards.
- Keep check logic in `theme-check-common` — only add to `theme-check-node` / `theme-check-browser` when runner-specific wiring is needed.

## Testing Patterns
Theme-check tests live next to the check as `index.spec.ts`. Template:
```ts
import { describe, it, expect } from 'vitest';
import { runLiquidCheck, autofix } from '../../test';
import { MyCheck } from './index';

describe('MyCheck', () => {
  it('reports the specific bad pattern', async () => {
    const offenses = await runLiquidCheck(MyCheck, `<bad liquid>`);
    expect(offenses).toHaveLength(1);
    expect(offenses[0].message).toMatch(/expected phrase/);
  });

  it('does not flag valid code', async () => {
    const offenses = await runLiquidCheck(MyCheck, `<good liquid>`);
    expect(offenses).toHaveLength(0);
  });
});
```

## Confidence Gate (before committing)
Before any commit or push:
- [ ] Bug reproduced by a NEW failing test at current `main`
- [ ] Fix makes the new test pass
- [ ] `yarn build` clean
- [ ] `yarn test` all green
- [ ] `yarn workspaces run type-check` all green
- [ ] `yarn format:check` clean
- [ ] `yarn changeset` file committed, bump level correct
- [ ] `git config user.email` = `<your-email>`
- [ ] `git diff | grep -iE "<your-employer>|<work-handle>|/Users/<your-github>|Obsidian"` returns empty
- [ ] No em dashes in commit / PR body
- [ ] No Co-Authored-By AI line

## Security / Identity Checklist
- [ ] `git config user.email` = `<your-email>`
- [ ] No `<your-work-domain>`, `<work-handle>`, `<your-employer>`, Partner ID <your-id> in diff
- [ ] No private paths (`/Users/<your-github>/...`, `Obsidian-vault`, `<your-vault>`) in diff
- [ ] No credentials / tokens / Shopify app secrets

## After First Merge
- Update `<your-ecosystem-overview>` PR tracker with merged status
- Add session log under `<your-ecosystem-logs>/`
- Update `MEMORY.md` project entry if the target list changes

## Do NOT
- Use npm or pnpm (repo is yarn classic)
- Skip the changeset (PRs are blocked without one)
- Add a new dep for a narrow bug fix
- Rename public APIs in a bug-fix PR (submit as separate feat/ PR)
- Include AI-authored tone or Co-Authored-By AI lines
- Touch `release-orchestrator/` or `.spin/` (Shopifolk-only)

## Key Reference Files
- `CONTRIBUTING.md` — dev setup, top-level PR steps
- `.github/PULL_REQUEST_TEMPLATE.md` — what the PR body must contain
- `.changeset/config.json` — fixed-version groups + access level
- `dev.yml` — node/yarn versions
- `packages/theme-check-common/src/checks/index.ts` — registry of all checks
- `packages/vscode-extension/src/node/formatter.ts` — referenced by issue #619

## Gotchas (build / setup)
- Build MUST run before tests or vitest will fail to resolve cross-package imports (graygilmore note on issue #1086)
- `theme-check-docs-updater` tests can fail if `@shopify/liquid-html-parser` package is not built (same thread)
- Issue #1086 is actively claimed by `cbueker-it` (Nov 28) — do not target

## PR Gotchas

For PR-time gotchas to this repo (pre-push checklist + lessons from past PRs), see [theme-tools-PR-Gotchas](./theme-tools-PR-Gotchas.md) in `./` (this folder). Check it before every `gh pr create`. After each PR closes, add new lessons there.

For Shopify OSS-wide gotchas (apply across multiple Shopify repos), see [shopify-oss-PR-Gotchas](./shopify-oss-PR-Gotchas.md) in `./` (this folder).

This is gate 2 of the contribution lifecycle. Gate 1 is [Research-Gotchas](../../core/Research-Gotchas.md) (qualify the repo BEFORE starting work).
