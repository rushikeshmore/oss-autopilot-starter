---
type: skill
name: Shopify OSS Contribution Guide
description: Identity, setup, coding standards, PR flow, and testing for contributing to Shopify open source repos (Shopify-AI-Toolkit, agent-skills, shop-chat-agent, etc.)
---

# Shopify OSS Contribution -- Agent Skill Guide

## Pre-flight (read before any work)
Before adding any `Shopify/*` repo to the target list, run the 3-gate OSS qualification from `../../core/Research-Gotchas.md` §1. Shopify/Shopify-AI-Toolkit failed gate 1 (auto-close bot) on 2026-04-18 — do not re-target it without first confirming the bot has been removed and at least one external PR has merged in the last 90 days.

## Scope
This guide covers all `Shopify/*` open source repos targeted for contribution, with emphasis on the AI / agent / MCP developer-tooling line (primary dotdev networking target).

## Identity (CRITICAL)
- Use personal GitHub: `<your-github>` (<your-email>)
- **NEVER use work identity** for this project
- Verify before every commit: `git config user.email` must return `<your-email>`
- No <your-employer> references anywhere in commits, PR bodies, or issue comments
- Shopify Partner ID <your-id> is tied to <your-employer> — do NOT reference it in PRs or issue threads

## Primary Target Repos

| Repo | Local Clone Path | Fork |
|------|------------------|------|
| `Shopify/Shopify-AI-Toolkit` | `<your-code-path>/03-open-source/shopify/Shopify-AI-Toolkit` | `<your-github>/Shopify-AI-Toolkit` |
| `Shopify/agent-skills` | `<your-code-path>/03-open-source/shopify/agent-skills` | `<your-github>/agent-skills` |
| `Shopify/shop-chat-agent` | `<your-code-path>/03-open-source/shopify/shop-chat-agent` | `<your-github>/shop-chat-agent` |
| `Shopify/ai-agent-partner-sales-channel-template` | `<your-code-path>/03-open-source/shopify/ai-agent-partner-sales-channel-template` | `<your-github>/ai-agent-partner-sales-channel-template` |

Clone parent: `<your-code-path>/03-open-source/shopify/`

## Local Development Setup

### Shopify-AI-Toolkit
Self-contained marketplace for Claude Code, Cursor, Codex, Gemini CLI plugins.

```bash
git clone git@github.com:<your-github>/Shopify-AI-Toolkit.git
cd Shopify-AI-Toolkit
git remote add upstream git@github.com:Shopify/Shopify-AI-Toolkit.git

# Structure
# .claude-plugin/marketplace.json  <- Claude Code plugin definition
# .cursor-plugin/                  <- Cursor plugin
# .codex-plugin/                   <- Codex plugin
# gemini-extension.json            <- Gemini CLI extension
# .mcp.json                        <- MCP server definition
# skills/                          <- 16 SKILL.md files + validate/search scripts
# plugin.json
```

**Verifying a Claude Code plugin fix locally:**
```bash
# In Claude Code:
/plugin marketplace add /absolute/path/to/local/Shopify-AI-Toolkit
/plugin install shopify-plugin@shopify-ai-toolkit
# Expect success, no schema errors
```

Do NOT run `npm install` at repo root without checking — some skills' `package-lock.json` files point to Shopify's private npm.shopify.io registry (Issue #11 acknowledges this). Run installs only inside specific skill dirs that use public registry.

### agent-skills
```bash
git clone git@github.com:<your-github>/agent-skills.git
cd agent-skills
git remote add upstream git@github.com:Shopify/agent-skills.git
nvm use 20  # validate.js has Node 22 compat issues (Issue #5)
```

### shop-chat-agent
Node + React Router reference app.
```bash
git clone git@github.com:<your-github>/shop-chat-agent.git
cd shop-chat-agent
git remote add upstream git@github.com:Shopify/shop-chat-agent.git
npm install
npm run dev  # requires Shopify app credentials via env vars
```

## Branch & PR Flow

### Branch naming
- `fix/short-description` for bugfixes (e.g. `fix/claude-code-marketplace-source-format`)
- `docs/short-description` for docs-only changes
- `chore/short-description` for dep bumps or tooling
- `feat/short-description` for features
- Keep branch names short, kebab-case, imperative

### Commits
- Imperative, capitalize first letter, no trailing period
- Subject line under 72 chars
- Body explains "why and how" (separated by blank line)
- **No em dashes (`--`)** — use regular hyphens or rewrite
- **No Co-Authored-By AI lines** — never include these
- **No polished AI tone** — write like a developer, natural and direct
- Reference the issue in the body: `Closes #10` or `Refs #10`

Example:
```
Fix marketplace.json source format for Claude Code install

The plugins[*].source field used a string "./" which fails Claude Code
schema validation. Convert to object form { source: "local", path: "./" }
matching the format used by other working marketplace plugins.

Verified locally with /plugin marketplace add followed by /plugin install
on Claude Code v2.x.

Closes #10
```

### PR flow
1. Fork on GitHub (first time only)
2. Branch from `main`: `git checkout -b fix/topic main`
3. Make changes, commit, push to fork: `git push -u origin fix/topic`
4. Open PR against upstream `main` via `gh pr create`
5. PR title should match commit subject for single-commit PRs
6. PR body: what, why, how verified, link to issue
7. **No AI disclosure required** (Shopify doesn't require it the way Gutenberg does). Still write naturally, not in AI tone.
8. Respond to review feedback by amending existing commits when possible: `git commit --amend --no-edit` + `git push -f origin branch-name`
9. Rebase (not merge) to resolve conflicts: `git fetch upstream && git rebase upstream/main`

### Review expectations
- Primary reviewer on Shopify-AI-Toolkit: `nelsonwittwer`
- External PRs typically reviewed/merged within 24 hours for clean bug fixes
- Expect push-back on:
  - Naming/rename changes (PR #17 precedent)
  - Scope expansion beyond the filed issue
  - Style-only PRs without bug fix content

## Coding Standards

### JavaScript / TypeScript
- Follow existing style in the repo (don't introduce formatter changes)
- Node version: per `.nvmrc` if present, else check `engines` in `package.json`
- No new dependencies in bug-fix PRs unless unavoidable
- Prefer explicit imports over wildcards

### SKILL.md files (Shopify-AI-Toolkit, agent-skills)
- Use fenced code blocks with explicit language tags (`bash`, `json`, `tsx`)
- Keep `<system-instructions>` tags on their own lines
- Preserve existing directive phrasing where possible
- Test that validation scripts still parse after edits

### marketplace.json / plugin.json schemas
- `source` field must be an object, not a string:
  ```json
  { "source": { "source": "local", "path": "./" } }
  ```
- Valid source types: `local`, `url`, `git-subdir`
- Reference known-working plugins (e.g. `adspirer-ads-agent`) when unsure

## Testing

### Shopify-AI-Toolkit
- Install the plugin locally into Claude Code / Cursor / Codex / Gemini CLI to verify
- Run skill validation scripts where present: `node scripts/validate.mjs --file <path> --target <target>`
- Run `node scripts/search_docs.mjs "<query>"` to smoke-test doc search

### agent-skills
```bash
node scripts/validate.js --file test.liquid --filetype sections
node scripts/search_docs.js "product"
```

### shop-chat-agent
```bash
npm test          # unit tests if present
npm run lint
```

## Commit Message Templates

### Bug fix
```
<Verb in imperative> <short subject>

<Why this matters + how the fix works>

<How verified>

Closes #<num>
```

### Docs
```
Docs: <what changed>

<Why>

Refs #<num>
```

### Chore / dep bump
Usually handled by Dependabot. If manual:
```
Bump <pkg> from <old> to <new>

<Reason, e.g. security advisory or compatibility>
```

## Security / Identity Checklist (before every push)

- [ ] `git config user.email` returns `<your-email>`
- [ ] No <your-employer> refs in diff: `git diff | grep -i <your-employer>` returns empty
- [ ] No Partner ID <your-id> in diff
- [ ] No `<work-handle>` (work GitHub handle) refs in diff
- [ ] No private paths (`/Users/<your-github>/...`) committed
- [ ] No API keys, tokens, or Shopify app credentials committed
- [ ] No em dashes in commit message or PR body
- [ ] No Co-Authored-By AI line in commits

## After First Merge
- Add the merged PR to the ecosystem `Overview.md` PR Tracker with status update
- Log the session under `00. Logs/`
- Update Dashboard priority line
- Consider follow-up: does the fix suggest a related issue worth filing?

## Escalation / Fallback
- If PR stalls more than 7 days with no review: polite bump comment on the PR referencing the issue
- If PR is closed unmerged with reason: record the lesson in ecosystem Overview (see PR #17 precedent)
- If a PR idea is rejected in advance by maintainers: do NOT submit. Pick a different issue.

## PR Gotchas

For PR-time gotchas that apply across Shopify OSS repos (pre-push checklist + lessons), see [shopify-oss-PR-Gotchas](./shopify-oss-PR-Gotchas.md) in `./` (this folder). Check it before every `gh pr create` to any Shopify repo. After each PR closes, add new lessons there.

For repo-specific gotchas (e.g. theme-tools-only patterns), see the per-repo gotchas file: [theme-tools-PR-Gotchas](./theme-tools-PR-Gotchas.md). Create new per-repo files as material accumulates.

This is gate 2 of the contribution lifecycle. Gate 1 is [Research-Gotchas](../../core/Research-Gotchas.md) (qualify the repo BEFORE starting work).

## Quick Reference

### Files commonly edited (Shopify-AI-Toolkit)
- `.claude-plugin/marketplace.json` — plugin marketplace schema
- `skills/*/SKILL.md` — per-skill instructions
- `skills/*/scripts/validate.mjs` — validation scripts
- `README.md`

### Files commonly edited (agent-skills)
- `shopify-liquid/SKILL.md`, `shopify-hydrogen/SKILL.md`, etc.
- `scripts/validate.js`
- `scripts/search_docs.js`

### Key GitHub URLs
- Issues across all Shopify repos: https://github.com/issues?q=org%3AShopify+is%3Aissue+is%3Aopen+assignee%3A%40me
- Your PRs: https://github.com/pulls?q=is%3Apr+author%3A<your-github>+org%3AShopify
