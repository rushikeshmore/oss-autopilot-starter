---
type: skill
name: OSS Autopilot Starter
description: Operational config for running an agent in autopilot mode on real OSS contributions. Defines the autonomy boundary, the non-negotiable safety wires, the per-session orchestration ritual, and the lessons-learned protocol. Adapt the identity placeholders for your own setup; the process is portable.
---

# OSS Autopilot Starter

> A starter skill for running an agent (Claude Code, Codex, Cursor, etc.) in autopilot mode on real open source contributions. This file defines what the agent can do without asking, what requires explicit permission, the non-negotiable safety wires that prevent autopilot from breaking things, and the per-session ritual that keeps PRs landing rather than getting closed.
>
> **Adapt the placeholders** (`<your-personal-email>`, `<your-personal-github>`, `<work-identifier>`, etc.) for your own identity. The PROCESS is portable; the IDENTITY is yours. The system is the moat — not the agent.

## What "autopilot mode" actually means

Autopilot is NOT "tell the agent 'fix this' and walk away." It's:

- **Agent reads the entire system before writing any code** — research-gotchas, contribution skill, PR-gotchas, all loaded before line 1
- **Agent operates within explicit autonomy boundaries** — can commit, push, open PR, respond to review; cannot force-push upstream, cannot touch unrelated files
- **Agent self-checks before every push** — identity, lint, tests, per-repo gotchas checklist
- **You orchestrate at the contract level** — "fix issue #X to spec Y"; agent handles the line-level work
- **Lessons get captured after every PR** — success or failure, the gotchas file grows

If any of these break down, autopilot drifts into "AI slop" — PRs that look right but fail review. The system below is what prevents that.

## Non-negotiable safety wires (the things that DO NOT bend)

### 1. Identity firewall (the highest-priority rule)

Personal contributions go out under personal identity only. Work identity (employer email, work GitHub handle, work paths) NEVER appears in personal OSS commits, PR bodies, or issue comments. One leak attached to a public PR is permanent and tied to the personal GitHub used for OSS / networking / hiring signal.

**Before EVERY `git push`, `git push -f`, and `gh pr create`** — run the two-line check in the same Bash call:

```bash
git config user.email      # must be <your-personal-email>
git log -1 --format='%ae'  # must be <your-personal-email>
```

If either fails: STOP. Fix with `git config user.email <your-personal-email>` and amend or reset the offending commit before pushing.

**Bonus check** for new repos or after long sessions:

```bash
git diff origin/main..HEAD | grep -iE "<work-handle>|<work-domain>|<private-paths>"
# Must return empty
```

A session-start check is NOT enough. `git config` can change mid-session via tooling, repo switches, or `--local` overrides in worktrees. Run the check every time.

### 2. Pre-flight: research-gotchas gate (BEFORE you target a repo)

Before targeting any new repo, run the qualification gates from your domain's `Research-Gotchas.md`. Standard OSS gates:

- **PR-exists vs PR-merged** — workflows dir audit for auto-close bots; don't waste effort on repos that auto-close externals
- **Recent external-accepting activity** — verify at least one non-bot, non-org-member PR merged in the last 50 PRs
- **No "disable auto-close" proposals stuck open** — if such a proposal exists but is closed unmerged, treat as strong negative signal

If a repo fails any gate, do NOT target it. Move on. Document the failure as a new gotcha entry within 24 hours if it's a new failure mode.

When memory asserts external state (e.g., "repo X accepts externals"), re-verify with live commands (`gh api`, `gh pr view`). Memory is frozen at write time; reality moves.

### 3. Pre-push: per-repo PR-Gotchas checklist (BEFORE every `gh pr create`)

Each target repo has a `<repo>-PR-Gotchas.md` file with a Pre-Push Checklist of lessons from past PRs. Run it before every push.

If any item fails: fix it, re-run. Do NOT push past a failed check. Common pre-push items:
- CHANGELOG entry in the right section (project-specific — Unreleased > Bug Fixes, etc.)
- Lockfile not leaked from a routine `npm install`
- No Co-Authored-By in commits
- AI Disclosure section completed (if project requires it)
- All scoped tests pass locally
- Branch name follows project convention

If your target repo doesn't have a PR-Gotchas file yet, create one as you encounter the first lesson worth capturing.

### 4. Commit style discipline (no em dashes, no AI tone, no Co-Authored-By)

Every commit message and PR body must read as a developer writing, not an AI generating.

Rules:
- **No em dashes** (`—`). Use regular hyphens or rewrite.
- **No AI-sounding language.** No "I'd be happy to", no "Certainly!", no trailing summaries, no "great question!" energy.
- **No `Co-Authored-By: <AI>` lines.** AI disclosure (when required) lives in the PR body's dedicated section, NOT in commit metadata.
- **Imperative mood, capitalize first letter, no trailing period.**
- **Body explains "why", not "what".** The diff already shows what.
- **Follow each project's specific format.** Branch naming, ticket prefixes, PR templates — match the repo's conventions exactly. Read their `CONTRIBUTING.md` once per new repo and capture rules in the per-repo skill file.

## Code quality mindset (think like a human reviewer, not a compiler)

Two PRs in a row in early development of this system had reviewers catch things the agent should have caught itself. Mechanical translations break things. Apply this checklist to every change:

1. **Trace ALL code paths, not just the one in the issue.** Search for every caller, every consumer, every path that touches the same state. If you fix path A, ask "does path B have the same problem?" If yes, fix it too.

2. **Audit behavioral contracts, not just functionality.** Ask: "Does this change preserve the same guarantees the old code provided?" Reference stability, callback timing, side effect ordering, error propagation — all of these matter. Not just "does it produce the right output."

3. **When refactoring (class → function, extracting helpers, etc.), audit every instance method.** Each one may have implicit stability guarantees (refs, event handlers, callbacks passed to children). An inline arrow function is NOT equivalent to a class method.

4. **If existing tests pass but you have doubts, the test suite is incomplete.** Add regression tests for behavioral contracts that existing tests don't cover. Passing tests ≠ correct behavior.

5. **Before pushing, role-play as the reviewer.** Read your own diff line by line. Ask: "what would a maintainer flag here?" If you can think of a concern, fix it before pushing — don't wait for review feedback.

6. **Never dismiss test failures as 'environment issues'.** If a test fails locally, investigate fully. It may be revealing a real gap in your fix.

## Autonomy boundary

### What the agent CAN do without asking

- Read any file in the repo, the vault, or the agent's own context
- Run tests, linters, type-checks, build commands
- Create branches following the repo's naming convention
- Implement the fix per the filed issue / spec
- Run all relevant tests locally
- Run the per-repo PR-Gotchas pre-push checklist
- Commit (with project-conforming message format)
- Push to the agent's own fork
- Create the PR with the project's PR template filled out
- Respond to maintainer review feedback (amend / new commit per project convention)
- Resolve rebase conflicts
- Update the vault tracker + log after PR is filed
- Apply the cross-repo cleanup discipline (see below) when an identity / privacy issue is found

### What the agent CANNOT do without asking

- Force-push to anything other than the agent's own fork
- Push directly to upstream / main / protected branches
- Modify files outside the scope of the filed issue
- Add new dependencies in a bug-fix PR
- Submit a PR that involves renames, naming changes, or scope expansion (file an issue first to validate direction with maintainer)
- Submit meta-files (AGENTS.md, CONTRIBUTING additions, governance docs) without confirming the project wants them
- Skip the pre-push checklist for any reason
- Disable any pre-commit hook or signing requirement (`--no-verify`, etc.)
- Edit append-only records (session logs, decisions ledger, git history) retroactively

## Cross-repo cleanup discipline

When the agent finds an identity / privacy / security issue in one repo, it must immediately scan ALL public repos for the same pattern. Don't wait for the user to ask. Don't fix only the repo currently in focus.

```bash
gh repo list <your-github> --json name,isPrivate --jq '.[] | select(.isPrivate==false) | .name'
# Then grep each for the same problem pattern
```

This is the "don't leave the leak in three repos because you only noticed it in one" discipline. Identity leaks compound — each public commit is permanent.

## Per-session orchestration ritual

Every OSS work session, in order:

1. **Verify identity** — `git config user.email` returns personal
2. **Sync upstream** — `git pull upstream <default-branch>`
3. **Pick a target issue** — one with a clear repro, not a scope-debate
4. **Run research-gotchas check** if this is a new repo
5. **Read the per-repo contribution skill** — coding style, PR flow, testing
6. **Read the per-repo PR-Gotchas file** — past lessons applicable to this PR
7. **Implement the fix** following the skill's rules
8. **Test locally** — run their tests, linter, type-check, build
9. **Run the per-repo PR-Gotchas pre-push checklist**
10. **Commit + push to fork**
11. **Open PR** with their template filled correctly
12. **Update vault** — PR tracker, session log
13. **Monitor per the PR-monitoring SOP** (Day-7 / Day-14 / Day-21+ cadence — escalate explicitly, don't wait passively)

## Lessons-learned protocol (after every PR)

Whenever a PR:

- **Gets reworked after maintainer feedback** → capture the lesson in `<repo>-PR-Gotchas.md`
- **Gets closed unmerged** → capture WHY (direction-decline, scope-debate, auto-close bot, etc.) in `<repo>-PR-Gotchas.md`
- **Reveals a project-specific convention** → update the `<repo>-contribution.md` skill
- **Surfaces a new failure mode worth gate-checking before future repos** → add an entry to `Research-Gotchas.md`

Each lesson entry follows the same shape:
- What happened (1-2 sentences with PR# / commit ref)
- Why it happened (the root cause, not the symptom)
- What to check next time (the rule that prevents recurrence)
- Source (PR# / issue# / commit)

Cumulative file > losing the lesson. The gotchas files are the moat that compounds; lose an entry and you'll repeat the mistake.

## How to adapt this starter for your setup

1. **Replace identity placeholders** with your own:
   - `<your-personal-email>` → your email
   - `<your-personal-github>` → your GitHub handle
   - `<work-identifier>`, `<work-domain>`, `<work-handle>`, `<private-paths>` → your work identifiers to firewall against

2. **Create your own `Research-Gotchas.md`** at the OSS folder root. Start with the 3 standard gates (auto-close audit, recent-external-merge check, "disable auto-close" verification). Add new gotchas as you encounter new failure modes.

3. **Create per-repo contribution skill files** for each target repo at `<ecosystem>/_skills/<repo>-contribution.md`. Capture: identity rule, local dev setup, branch/PR flow, coding standards, testing, communication channels.

4. **Create per-repo `<repo>-PR-Gotchas.md` files** as you accumulate PR-time lessons. Co-locate with the repo's folder so it's discoverable alongside the logs.

5. **Wire this skill into your agent's autoload:**
   - **Claude Code:** place in vault `_skills/` or in `.claude/skills/` at the repo root
   - **Cursor:** use rules / `.cursorrules`
   - **Codex:** use `AGENTS.md`
   - **Aider / Continue / etc.:** check their context-loading docs

6. **Verify the loop closes:** after your first PR through this system, confirm a lesson got captured. If nothing went into the gotchas file, either you got lucky or you didn't read the diff carefully. Re-read.

## What loads at session start (the autopilot payload)

When the agent opens a session in a target OSS repo, it should have ALL of these in context BEFORE writing any code:

| File | Purpose |
|---|---|
| This file (`autopilot-starter.md`) | Operating manual — what to do and what not to |
| `open-source-workflow.md` | General lifecycle (fork → PR → review → merge) |
| `Research-Gotchas.md` | Gate 1 — qualify the repo before work |
| `<repo>-contribution.md` | How to contribute to THIS repo specifically |
| `<repo>-PR-Gotchas.md` | Gate 2 — lessons from past PRs to this repo |
| `pr-monitoring.md` | Escalation cadence (Day-7 / Day-14 / Day-21+) |

That's what "autopilot" actually means: the agent has the entire SYSTEM in head, not just the issue. The issue is the input; the system is the engine.

## Anti-patterns (autopilot is NOT this)

- "Just tell the agent 'fix this PR' and walk away" — no, you orchestrate at the contract level
- "Disable the pre-push checks because they're annoying" — no, those are the safety wires
- "Use the agent's default commit message format" — no, follow the project's
- "Skip the gotchas file because it's tedious" — no, that's the system
- "Trust memory about external state" — no, re-verify with live commands
- "Fix the identity leak in just the repo where I noticed it" — no, scan all repos
- "Force-push to upstream because it'll be faster" — no, never push outside your fork without explicit ask

If you find yourself reaching for any of these, autopilot is broken. Stop and fix the boundary or the safety wire, not the rule.

---

**Adapt the placeholders. Set up your gotchas files per repo. The system is the moat — not the agent.**
