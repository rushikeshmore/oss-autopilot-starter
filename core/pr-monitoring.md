---
type: skill
name: PR Monitoring and Follow-up
description: SOP for tracking open PRs across all open source ecosystems -- check cadence, when to ping, stale PR handling
---

# PR Monitoring and Follow-up

## Quick Status Check

Run this to see all open PRs across ecosystems:
```bash
# WordPress
gh pr list --repo WordPress/openverse --author <your-github>
gh pr list --repo WordPress/gutenberg --author <your-github>

# Greenpeace
gh pr list --repo greenpeace/planet4-master-theme --author <your-github>
gh pr list --repo greenpeace/planet4-child-theme-korea --author <your-github>
```

Or check a specific PR:
```bash
gh pr view <number> --repo <org>/<repo>
gh pr checks <number> --repo <org>/<repo>
```

## Check Cadence

| Time since filing | Action |
|-------------------|--------|
| Day 1-3 | Nothing. Give reviewers time. |
| Day 4-5 | Check CI status. If CI failed, fix immediately. |
| Day 7 | First follow-up: polite comment on the PR asking if anything needs changing. |
| Day 14 | Slack outreach: message the project's channel asking for review. |
| Day 21+ | Tag a specific maintainer in the PR. If still no response, move on to other contributions. |

## Follow-up Messages

### PR comment (day 7)
Keep it short and helpful, not pushy:
```
Hi -- just checking if this is ready for review or if anything needs changing on my end. Happy to adjust.
```

### Slack message (day 14)
```
Hey, I have an open PR at [link] -- would appreciate a review when someone has a moment. Let me know if anything needs changing.
```

### Tagging maintainer (day 21+)
```
@maintainer would you be able to take a look at this when you get a chance? Happy to make any changes needed.
```

## When CI Fails

1. Check the failure -- is it your change or a pre-existing flaky test?
2. If your change:
   - Fix it locally
   - Amend or new commit (per project convention)
   - Push
3. If pre-existing:
   - Note it in the PR comment: "CI failure on X appears pre-existing -- same test fails on main"
   - Link to a failing main branch run if you can find one

## Stale PR Handling

If a PR sits for 30+ days with no response:
- Check if the project is still active (recent commits to main?)
- If active but ignoring you: close the PR with a polite note, move on
- If the project is dormant: close the PR, note it in the vault, don't invest more time
- Update PR Tracker status to "Closed (stale)" or "Closed (project dormant)"

## Vault Updates

After each monitoring check:
- Update PR Tracker status in the ecosystem's Overview.md
- If a PR gets merged, update status and check after-merge steps (contributor list, branch cleanup)
- Log significant events (review received, changes requested, merged) in session logs

## Current Open PRs

Kept in each ecosystem's Overview.md under `## PR Tracker`. Check the main `the OSS folder index` for which ecosystems have active PRs.
