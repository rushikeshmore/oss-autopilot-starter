---
type: skill
name: WordPress Agent Skills Contribution Guide
description: Coding style, PR process, skill authoring, and conventions for contributing to WordPress/agent-skills
---

# Agent Skills Contribution - Agent SOP

## Repository
- Upstream: https://github.com/WordPress/agent-skills
- Local clone: `<your-code-path>/agent-skills`
- Fork: `<your-github>/agent-skills`
- Default branch: `trunk` (NOT main)

## Identity (CRITICAL)
- Personal GitHub: `<your-github>` (<your-email>)
- NEVER use work identity

## What Are Agent Skills?
Portable bundles of Markdown instructions + scripts that teach AI coding assistants (Claude, Copilot, Codex, Cursor) how to do WordPress development correctly. Skills are NOT code libraries. They are structured knowledge.

## Skill Anatomy (MANDATORY STRUCTURE)

Every skill lives in `skills/<skill-name>/` and must contain:

```
skills/<skill-name>/
├── SKILL.md              # Main file (YAML frontmatter + procedural checklist)
├── references/           # Deep-dive docs on specific topics (*.md)
└── scripts/              # Deterministic helpers (*.mjs) - optional
```

### SKILL.md Required Sections
1. YAML frontmatter with `name`, `description`, `compatibility`
2. When to use - trigger conditions
3. Inputs required - what the AI needs first
4. Procedure - step-by-step checklist
5. Verification - how to confirm it worked
6. Failure modes / debugging - common problems
7. Escalation - when to ask a human

### YAML Frontmatter Rules (ENFORCED BY CI)
- `name` must match the folder name exactly
- `name`: lowercase, hyphens only, no leading/trailing hyphens, no `--`, max 64 chars
- `description`: max 1024 chars
- `compatibility`: must include "WordPress 6.9" AND "PHP 7.2.24+", max 500 chars

## Eval Scenarios (REQUIRED)

Every new skill needs at least one eval scenario in `eval/scenarios/<slug>.json`:

```json
{
  "name": "Descriptive scenario name",
  "skills": ["wordpress-router", "wp-project-triage", "your-skill"],
  "query": "A realistic user prompt",
  "expected_behavior": [
    "Step 1: ...",
    "Step 2: ..."
  ],
  "success_criteria": [
    "Criteria 1",
    "Criteria 2"
  ]
}
```

## Design Principles
- Prefer small, composable skills over mega-skills
- Keep `SKILL.md` short and procedural; push depth into `references/`
- Bundle deterministic checks as scripts when reliability matters
- Treat upstream docs as canonical; store agent-first checklists and decision trees
- Keep file references 1 hop from SKILL.md (avoid deep chains)

## Scaffolding a New Skill

```bash
node shared/scripts/scaffold-skill.mjs <skill-name> "<description>"
```

## Validation (MUST PASS BEFORE PR)

```bash
node eval/harness/run.mjs
```

This checks:
- Every skill dir has a SKILL.md
- Frontmatter has required fields (name, description, compatibility)
- Name matches folder name
- Name follows naming rules
- Description under 1024 chars
- Compatibility mentions WP 6.9 + PHP 7.2.24

## Build & Install (for testing)

```bash
node shared/scripts/skillpack-build.mjs --clean    # Build dist/
node shared/scripts/skillpack-install.mjs --global  # Install to ~/.claude/skills/
```

## Coding Style
- **No package.json** - zero dependencies, pure Node.js stdlib
- **Scripts:** `.mjs` (ES modules), use `node:` protocol imports
- **File naming:** lowercase with hyphens
- **Markdown:** Standard markdown, YAML frontmatter for SKILL.md
- **No TypeScript** - plain JavaScript for scripts
- **License:** GPL-2.0-or-later

## PR Process

### Branch Naming
```
add/<skill-name>          # New skill
fix/<description>         # Bug fix
improve/<skill-name>      # Enhancement
```

### PR Template
```markdown
## What?
[Brief description]

## Why?
[Problem being solved or gap being filled]

## How?
[Implementation details]

## Testing
- [ ] `node eval/harness/run.mjs` passes
- [ ] Skill follows required SKILL.md structure
- [ ] At least one eval scenario added

## AI Disclosure
[Per WordPress AI Guidelines]
```

### What Gets Merged
- Small fixes merge fast (1 review)
- New skills need: SKILL.md with all 7 sections, references/ if needed, eval scenario(s), README table update
- Reviews are lightweight - usually 1 approving review
- GitHub Copilot reviews as commenter, but human approval is what matters

## README Update
When adding a new skill, add a row to the Available Skills table in `README.md`:
```markdown
| **skill-name** | Brief description of what it teaches |
```

## AI Authorship Disclosure
Update `docs/ai-authorship.md` table if the skill was AI-assisted:
```markdown
| [skill-name](../skills/skill-name/SKILL.md) | Yes | Yes | Yes |
```

## Key Maintainers
- `Jameswlepage` - Primary maintainer
- `ciampo` - Automattic/WordPress contributor
- `galatanovidiu` - Reviews PRs
- `sirreal` - WordPress core team member

## Do NOT
- Create skills without eval scenarios
- Use package.json or npm dependencies
- Skip the `compatibility` frontmatter field
- Create mega-skills (keep SKILL.md short, use references/)
- Skip AI disclosure in PR
- Push to `main` (default branch is `trunk`)
