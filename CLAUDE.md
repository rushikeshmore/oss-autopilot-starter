# Working with this repo

> If you're an agent (Claude Code, Codex, Cursor, Aider, etc.) loading this repo for the first time, start here.
>
> **For non-Claude agents:** treat this `CLAUDE.md` as your `AGENTS.md`. The thin `AGENTS.md` next to this file just redirects here. One file, one source of truth.

## What this repo is

This is the **OSS Autopilot Starter** — a system for running an agent on real open source contributions.

It's a **STARTER, not an active codebase.** Someone cloned or forked this because they want to set up the system for their own OSS work. This repo does NOT contain a project to ship to production. It contains the markdown files you (the agent) need to load and apply when working on OTHER repos (Gutenberg, planet4, Shopify theme-tools, or whatever the user targets).

## Read order when loading this repo

Load these in order, top to bottom, before doing any OSS work the user asks you to help with:

1. **[README.md](./README.md)** — high-level pitch, structure, and the proof-of-system PR list. Read for orientation.
2. **[core/autopilot-starter.md](./core/autopilot-starter.md)** — THE operating manual. Defines: what you can do without asking, what requires explicit permission, the safety wires (identity firewall, pre-push checks), the per-session ritual, the lessons-learned protocol. **This is the file you check before any OSS work.**
3. **[core/open-source-workflow.md](./core/open-source-workflow.md)** — general contribution lifecycle (fork → branch → PR → review → merge).
4. **[core/Research-Gotchas.md](./core/Research-Gotchas.md)** — gate 1: repo qualification checks. Run these BEFORE targeting any new repo.
5. **[core/pr-monitoring.md](./core/pr-monitoring.md)** — post-PR escalation cadence (Day-7 / Day-14 / Day-21+).
6. **`examples/wordpress/`** — see how an ecosystem is laid out: contribution skills in `_skills/`, gotchas in per-project folders. Mirror this when the user sets up a new ecosystem.

## Hard rules when applying this system

1. **Identity firewall is non-negotiable.** Before EVERY `git push`, verify `git config user.email` and `git log -1 --format='%ae'` return the user's personal email (whatever they've replaced `<your-email>` with). Stop if either is wrong. See `core/autopilot-starter.md` § "Non-negotiable safety wires" for the full identity protocol.

2. **Replace placeholders before using as-is.** This repo ships with placeholders:
   - `<your-email>` — personal email
   - `<your-github>` — personal GitHub handle
   - `<your-employer>`, `<your-employer-domain>`, `<work-handle>` — work-identity tokens (the firewall scans for these to prevent leaks)
   - `<your-code-path>` — where the user clones repos
   - `<your-vault>` — the user's knowledge vault name

   If you're working in a clone where these are still unreplaced, surface that to the user BEFORE proceeding with any contribution. Don't make commits with placeholder values.

3. **Run both gates on every PR.** Gate 1 (Research-Gotchas) BEFORE starting work on a repo. Gate 2 (per-repo PR-Gotchas) BEFORE every `gh pr create`. Skipping either breaks autopilot.

4. **Capture lessons.** When a PR gets reworked, closed unmerged, or surprises you with maintainer feedback — add the lesson to the relevant `<repo>-PR-Gotchas.md` (at `examples/<ecosystem>/<repo>/<repo>-PR-Gotchas.md`). The system grows by accumulation; lose an entry and you'll repeat the mistake.

5. **Commit style:** no em dashes, no AI-sounding language, no `Co-Authored-By: <AI>` lines, follow each target repo's specific format. See `core/autopilot-starter.md` § "Commit style discipline".

## What this repo is NOT for

- **Not an application.** No production code to ship.
- **Not a test bed.** Don't experiment here; fork it and use the system on real OSS repos elsewhere.
- **Not the user's private vault.** This is the public starter; the user's actual skills + gotchas live in their own fork or vault.

## If you're contributing back to THIS repo

If the user asks you to improve the starter itself (add a new gotcha pattern, refine the operating manual, add a worked example for a new ecosystem):

- Follow the system on itself: identity-check before push, no Co-Authored-By, no AI tone in PR bodies.
- Update the README if the structure or any file's role changes.
- Don't add identity-specific examples — use placeholders only.
- License is MIT — keep additions MIT-compatible.

## Quick reference

| File | Role |
|---|---|
| `core/autopilot-starter.md` | Operating manual (the most important file) |
| `core/open-source-workflow.md` | General OSS contribution lifecycle |
| `core/Research-Gotchas.md` | Gate 1: qualify the repo |
| `core/pr-monitoring.md` | Escalation cadence after PR is open |
| `examples/wordpress/_skills/` | Per-repo contribution skills (worked examples) |
| `examples/wordpress/<repo>/` | Per-repo PR-Gotchas (worked examples) |

When in doubt: read `core/autopilot-starter.md`. It answers "what am I allowed to do" and "what do I have to check before doing it."
