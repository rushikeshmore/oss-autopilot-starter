---
type: skill
name: Open Source Contribution Workflow
description: General SOP for contributing to any open source project -- fork to merge lifecycle
---

# Open Source Contribution Workflow

General process that applies to ALL open source ecosystems. Per-project details (coding style, PR format, testing) live in each ecosystem's own `_skills/<repo>-contribution.md`. Per-repo PR-time lessons live in dedicated `<repo>-PR-Gotchas.md` files co-located with the repo's folder -- check the matching gotchas file before every `gh pr create`.

For agents operating in autopilot mode (Claude Code, Codex, Cursor, etc.), the operational config -- autonomy boundary, safety wires, per-session ritual, lessons-learned protocol -- lives in [autopilot-starter](./autopilot-starter.md). Load that skill before working on any OSS task.

## Code Quality Mindset (MANDATORY on every change)

Think like a human developer and reviewer -- not a compiler doing mechanical translations.

1. **Trace ALL code paths.** If you fix path A, search for every other path that touches the same state and ask "does it have the same problem?" Never do point fixes in isolation.
2. **Audit behavioral contracts, not just functionality.** Does the change preserve the same guarantees? Reference stability, callback timing, side effect ordering, error propagation -- not just "correct output."
3. **When refactoring (class -> function, extracting helpers, etc.), audit every method/property for implicit guarantees.** An inline arrow is NOT equivalent to a class method (ref stability). A new helper may change call timing.
4. **If tests pass but you have doubts, tests are incomplete.** Add regression tests for behavioral contracts the existing suite doesn't cover. Passing tests != correct behavior.
5. **Before pushing, role-play as the reviewer.** Read your own diff line by line. If you can think of a concern a maintainer would flag, fix it before pushing.
6. **Never dismiss test failures as "environment issues."** Investigate fully -- they may reveal a real gap in your fix.

## Before First Contribution

### 1. Research the project
- Read CONTRIBUTING.md, README, and any developer docs
- Check .editorconfig, linter configs, CI setup
- Look at recent merged PRs to understand actual conventions (not just documented ones)
- Identify reviewers/maintainers from recent PR activity
- Check if "good first issue" labels exist

### 2. Set up vault structure
- Create ecosystem folder under `./`
- Create `_skills/<project>-contribution.md` with coding style, PR flow, testing from official docs
- Create `Overview.md` with PR Tracker table (empty initially)
- Create `00. Logs/` for session logs

### 3. Fork and clone
```bash
mkdir -p <your-code-path>/03-open-source/<ecosystem>/
cd <your-code-path>/03-open-source/<ecosystem>/
gh repo fork <org>/<repo> --clone=true
```

### 4. Set git identity (CRITICAL -- do this immediately)
```bash
cd <repo>
git config user.name "Rushikesh More"
git config user.email "<your-email>"
```

Verify before every session:
```bash
git config user.email  # must be <your-email>
```

## PR Creation Flow

### 1. Sync with upstream
```bash
git checkout <default-branch>
git pull upstream <default-branch>
```

### 2. Create branch
- Use the project's naming convention (check their recent PRs)
- Common patterns: `fix-<thing>`, `add-<thing>`, `TICKET-123-description`

### 3. Make changes
- Follow the project's .editorconfig (indent style, line endings, final newline)
- Run their linter before committing
- Run their tests if possible

### 4. Commit
- Imperative mood, capitalize first letter, no period
- No em dashes anywhere
- No Co-Authored-By lines
- Write like a developer, not AI -- keep it direct and simple
- Include ticket/issue reference if the project uses them
- Body explains why, not what

### 4.5 Before push: per-repo PR Gotchas check (MANDATORY)

Each repo has a dedicated `<repo>-PR-Gotchas.md` file co-located with its folder. The file contains a Pre-Push Checklist + lessons from real PRs to that repo. Open the matching file and run its checklist before push.

| Repo | File | Location |
|---|---|---|
| Gutenberg | [gutenberg-PR-Gotchas](../examples/wordpress/gutenberg-PR-Gotchas.md) | `examples/wordpress/` |
| Openverse | [openverse-PR-Gotchas](../examples/wordpress/openverse-PR-Gotchas.md) | `examples/wordpress/` |
| Greenpeace planet4 | [planet4-PR-Gotchas](../examples/greenpeace/planet4-PR-Gotchas.md) | `examples/greenpeace/` |
| Shopify theme-tools | [theme-tools-PR-Gotchas](../examples/shopify/theme-tools-PR-Gotchas.md) | `examples/shopify/` |
| Shopify (general / multi-repo) | [shopify-oss-PR-Gotchas](../examples/shopify/shopify-oss-PR-Gotchas.md) | `examples/shopify/` |

For Shopify repos without a dedicated file (Shopify-AI-Toolkit, shop-chat-agent, agent-skills, etc.), use [shopify-oss-PR-Gotchas](../examples/shopify/shopify-oss-PR-Gotchas.md). Create a per-repo file once that repo has 2+ specific gotchas worth capturing.

If you encounter a NEW PR-time gotcha (rework requested, change closed unmerged, surprising review feedback): add it to the matching gotchas file after the PR closes. Cumulative file > losing the lesson.

This is the PR-time counterpart to [Research-Gotchas](./Research-Gotchas.md) (repo qualification, runs BEFORE starting work). Two gates protect the lifecycle.

### 5. Push and create PR
```bash
git push -u origin <branch>
gh pr create --repo <org>/<repo> --title "<title>" --body "<body>"
```

- Follow the project's PR template if they have one
- Reference the issue being fixed
- Keep PR body plain and direct

### 6. Post-PR
- Update vault: PR Tracker in Overview.md, session log
- Note the PR URL for monitoring

## Review Response Protocol

### When you get review feedback
1. Read all comments before making changes
2. Respond to each comment (even if just "done" or "good point, fixed")
3. Make the fix

### How to apply fixes (varies by project)
- **Amend + force push** (Greenpeace, most smaller projects):
  ```bash
  git commit --amend --no-edit
  git push -f origin <branch>
  ```
- **New commit** (some projects prefer visible history):
  ```bash
  git commit -m "Address review: <what changed>"
  git push origin <branch>
  ```
- Check the project's contribution guide -- it usually says which they prefer
- When in doubt, look at how other PRs handled review feedback

### Resolving conflicts
- Always rebase, never merge commits (unless the project says otherwise)
  ```bash
  git fetch upstream
  git rebase upstream/<default-branch>
  git push -f origin <branch>
  ```

## After Merge

1. Update PR Tracker status to "Merged"
2. Check if the project has a contributors list (e.g. .all-contributorsrc) and add yourself
3. Delete the local branch:
   ```bash
   git checkout <default-branch>
   git pull upstream <default-branch>
   git branch -D <branch>
   ```
4. Look for the next contribution opportunity

## Identity Checklist (run before every push)

```bash
git config user.email                    # must be <your-email>
git log -1 --format='%ae'               # verify last commit email
git diff --cached --name-only | xargs grep -l "<your-employer-domain>\|<work-handle>\|Obsidian\|<your-vault>" 2>/dev/null  # must return nothing
```
