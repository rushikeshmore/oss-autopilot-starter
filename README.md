# OSS Autopilot Starter

> Operating manual + skill files for running an agent on real open source contributions. Two-gate system, per-repo gotchas, identity firewall. **The system is the moat, not the agent.**

**8 merged PRs across WordPress and Greenpeace in a month.** Most shipped on autopilot — the agent read the system, picked an issue, wrote the fix, ran the checks, opened the PR, responded to maintainer review. I orchestrated at the contract level.

This repo is that system, stripped of identity, ready to fork and adapt.

## Watch the video

`[Video URL coming]`

## The system

```
OSS Harness System
├── autopilot-starter (operating manual — what the agent can do without asking)
│
├── Skills (how to do the work)
│   ├── open-source-workflow.md (general lifecycle: fork → PR → merge)
│   ├── pr-monitoring.md (escalation cadence: Day-7 / Day-14 / Day-21+)
│   └── per-repo contribution skills (one per target repo)
│
└── Gates / Safeguards (what to check)
    ├── Research-Gotchas (gate 1: qualify the repo BEFORE starting)
    └── per-repo PR-Gotchas (gate 2: lessons from past PRs BEFORE pushing)
```

## Why two gates

Most agent-on-OSS attempts fail at one of two points:

1. **Targeting a dead repo** — auto-close bots, dormant maintainers, scope-debate culture. You waste hours before discovering nobody merges externals.
2. **Repeating mistakes that maintainers already flagged** — wrong CHANGELOG section, wrong test file naming, scope expansion in a bug fix, unsolicited meta-files.

Research-Gotchas is the gate-1 fix. PR-Gotchas is the gate-2 fix. Together they form the safety net that lets autopilot actually work without the agent shipping AI slop.

## Repository structure

```
oss-autopilot-starter/
├── README.md              <- you are here (for humans)
├── CLAUDE.md              <- entry point for agents (Claude Code, Codex, Cursor, etc.)
├── AGENTS.md              <- thin pointer to CLAUDE.md (one source of truth)
├── LICENSE                <- MIT
│
├── core/                  <- start here, these are the system files
│   ├── autopilot-starter.md       <- operating manual; autonomy boundary + safety wires
│   ├── open-source-workflow.md    <- general contribution lifecycle
│   ├── pr-monitoring.md           <- post-PR escalation cadence
│   └── Research-Gotchas.md        <- gate 1: repo qualification checks
│
└── examples/
    └── wordpress/         <- worked example ecosystem (mirror this layout for any new one)
        ├── _skills/
        │   ├── gutenberg-contribution.md
        │   ├── openverse-contribution.md
        │   └── agent-skills-contribution.md
        ├── gutenberg/
        │   └── gutenberg-PR-Gotchas.md
        └── openverse/
            └── openverse-PR-Gotchas.md
```

The `examples/wordpress/` structure is the convention: `_skills/` holds one contribution skill per target repo; each target repo gets its own folder for its PR-Gotchas (and any future per-repo notes, logs, etc.). Mirror this pattern when you target a new ecosystem (`examples/<your-ecosystem>/_skills/` + `examples/<your-ecosystem>/<repo>/`).

PR-receipts from Greenpeace and Shopify (8 merged total) are linked in the "Proof of system" section below — same system, different ecosystems. Add per-ecosystem folders to your own fork as you expand.

## Quick start (adapt for your setup)

1. **Read [core/autopilot-starter.md](./core/autopilot-starter.md)** end-to-end. It defines the operating contract.
2. **Replace placeholders** across all files:
   - `<your-email>` → your personal email
   - `<your-github>` → your personal GitHub handle
   - `<your-employer>`, `<your-employer-domain>`, `<work-handle>` → your work-identity tokens (these become the identity firewall)
   - `<your-code-path>` → where you clone repos locally
   - `<your-vault>` → your knowledge vault name (Obsidian, Logseq, plain folder, etc.)
3. **Pick your first target repo.** Run the Research-Gotchas gates BEFORE writing code. If the repo fails (auto-close bot, dormant, scope-debate culture), pick a different one.
4. **Create a `<repo>-contribution.md` skill file** for the target. Use the [WordPress examples](./examples/wordpress/_skills/) as templates. Convention: `examples/<your-ecosystem>/_skills/<repo>-contribution.md`. Cover: identity rule, local dev setup, PR flow, coding standards, testing, communication channels.
5. **Create an empty `<repo>-PR-Gotchas.md` file** for the target, co-located in its own folder. Convention: `examples/<your-ecosystem>/<repo>/<repo>-PR-Gotchas.md`. It starts empty. You'll add entries as PRs teach you lessons.
6. **Wire the skills into your agent's autoload:**
   - **Claude Code**: place in `~/.claude/skills/` or per-project `.claude/skills/`
   - **Cursor**: use `.cursorrules` or rules
   - **Codex**: use `AGENTS.md`
   - **Aider / Continue / others**: check their context-loading docs
7. **Ship your first PR through the system.** Capture any lesson in the relevant `<repo>-PR-Gotchas.md` after it closes (merged or not — both teach you something).

## Proof of system (PRs shipped with this)

Across April-May 2026, 8 PRs merged + 2 active across WordPress and Greenpeace:

### Merged (8)
- [WordPress/gutenberg #77171](https://github.com/WordPress/gutenberg/pull/77171) — NavigableContainer class-to-hooks refactor (released in 33.0.0)
- [WordPress/gutenberg #77181](https://github.com/WordPress/gutenberg/pull/77181) — FormTokenField validation bug fix
- [greenpeace/planet4-master-theme #2950](https://github.com/greenpeace/planet4-master-theme/pull/2950) — pa11y duplicate URL fix
- [greenpeace/planet4-master-theme #2959](https://github.com/greenpeace/planet4-master-theme/pull/2959) — Remove unused wp-scripts placeholder devDependency
- [greenpeace/planet4-master-theme #2960](https://github.com/greenpeace/planet4-master-theme/pull/2960) — Drop unused react-scripts CRA dependency
- [greenpeace/planet4-master-theme #2961](https://github.com/greenpeace/planet4-master-theme/pull/2961) — PHPUnit test file naming convention
- [greenpeace/planet4-master-theme #2962](https://github.com/greenpeace/planet4-master-theme/pull/2962) — Align license metadata with GPL-3.0-or-later
- [greenpeace/planet4-master-theme #2963](https://github.com/greenpeace/planet4-master-theme/pull/2963) — Correct misleading docblocks in Like SQL class

### Active / In Review (2)
- [WordPress/openverse #5570](https://github.com/WordPress/openverse/pull/5570) — Audio thumbnail accessibility (aria-hidden)
- [Shopify/theme-tools #1182](https://github.com/Shopify/theme-tools/pull/1182) — Parse style body as Liquid inside `{% liquid %}` tags

### Closed unmerged (5) — these became gotchas
- WordPress/gutenberg [#77183](https://github.com/WordPress/gutenberg/pull/77183) + [#77184](https://github.com/WordPress/gutenberg/pull/77184) — direction-declined by maintainer (basis of Gutenberg PR-Gotchas #5)
- greenpeace/planet4-master-theme [#2964](https://github.com/greenpeace/planet4-master-theme/pull/2964) — AGENTS.md soft-declined (basis of planet4 PR-Gotchas #5)
- [planet4-child-theme-korea #22](https://github.com/greenpeace/planet4-child-theme-korea/pull/22) — dormant repo per stale-PR SOP
- [Shopify/Shopify-AI-Toolkit #18](https://github.com/Shopify/Shopify-AI-Toolkit/pull/18) — auto-closed by bot (basis of Research-Gotchas #1)

Every closed-unmerged PR became a lesson in a gotchas file. The system is the moat — each failure makes the next one less likely.

## Tools used

- [Claude Code](https://claude.com/claude-code) — the agent
- [Obsidian](https://obsidian.md) — the vault (any markdown editor works; the system is just markdown files)
- [Excalidraw](https://excalidraw.com) — for the system diagram in the video

## License

MIT — fork it, adapt it, ship.

---

If this works for you, consider opening a PR with your own per-repo skill or gotcha entry. The system grows by accumulation.
