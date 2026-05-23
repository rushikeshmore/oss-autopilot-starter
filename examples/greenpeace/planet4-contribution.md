---
type: skill
name: Planet 4 Contribution Guide
description: Coding style, PR process, testing, and conventions for contributing to Greenpeace Planet 4
---

# Planet 4 Contribution — Agent Skill Guide

## Repository
- Main theme: https://github.com/greenpeace/planet4-master-theme
- Blocks plugin: https://github.com/greenpeace/planet4-plugin-gutenberg-blocks
- Dev tooling: https://github.com/greenpeace/planet4-develop
- Local clone: `<your-code-path>/03-open-source/greenpeace/planet4-master-theme`

## Identity (CRITICAL)
- Use personal GitHub: `<your-github>` (<your-email>)
- NEVER use work identity for this project
- Verify before every commit: `git config user.email`

## Local Development Setup

### Requirements
- Docker and docker-compose
- Node/npm and nvm
- curl

### Installation (macOS)
```bash
# Apple Silicon — install Rosetta 2:
softwareupdate --install-rosetta

# Install Docker Desktop from https://docs.docker.com/desktop/install/mac-install/

# Clone dev tooling:
git clone https://github.com/greenpeace/planet4-develop/
cd planet4-develop
nvm use
npm install
npm run env:requirements

# Install default environment:
npm run env:install

# Start:
npm run env:start

# Stop:
npm run env:stop

# Themes live at: planet4/themes
# Plugins live at: planet4/plugins
```

### Running Tests
```bash
npm run env:e2e-install
npm run env:e2e
```

### Cleanup
```bash
npm run env:clean    # remove containers
npm run env:destroy  # full teardown
```

## Branch & PR Flow

### Branch Naming
Short, meaningful, with optional prefix:
- `fix-breadcrumbs-spacing`
- `add-author-avatars`
- `feature/change-covers-width`
- `bug/safari-button-editing`

### Commit Messages
- **Subject**: Imperative mood, capitalize first letter, no period
- **Ticket prefix**: `PLANET-1234 Force images aspect ratio`
- **Body**: Explains "why and how" (separated by blank line from subject)
- **No Co-Authored-By AI lines** -- never include these
- **No em dashes** (--) in commit messages or PR bodies -- use regular dashes or rewrite
- **Write like a developer** -- keep language natural and direct, avoid polished/formal AI tone

### PR Process
1. Fork the repo on GitHub
2. Create branch from `main`
3. Make changes, commit, push to fork
4. Open PR targeting `main` branch
5. Reference relevant issue/ticket in PR description
6. Single-commit PRs inherit commit name; multi-commit PRs need ticket in title

### Review Process
- When reviewers request changes: **amend existing commits**, don't add new ones
- Use `git commit --amend --no-edit` for small fixes
- Force-push amended work: `git push -f origin branch-name`
- Rebase to resolve conflicts (never merge commits)

## Coding Standards

### PHP
- **Standard**: PSR-12 with WordPress exceptions
- **Autoloading**: PSR-4
- **Linter**: PHP_CodeSniffer (`phpcs.xml.dist`)
- **Dependencies**: Composer

### JavaScript
- **Framework**: React (for Gutenberg blocks)
- **Linter**: ESLint (extends WordPress ESLint config)
- **Config**: `.eslintrc.json`
- **Build**: Webpack 5
- **Package manager**: npm

### CSS/Sass
- Vanilla CSS preferred, Sass for complexity
- **Linter**: Stylelint (`.stylelintrc`)

### General
- `.editorconfig` for IDE consistency
- SonarCloud automated code auditing on PRs
- GitHub security scanning for dependencies

## Testing

### E2E Tests (Playwright)
- Located in `tests/e2e/blocks/`
- Uses `@wordpress/e2e-test-utils-playwright`
- Run: `npm run env:e2e`

### Unit Tests (PHPUnit)
- Located in `tests/`
- Only 7 test files for 61+ source classes (big gap)
- Follow patterns in `test-exporter.php`, `test-open-graph.php`

### Accessibility (pa11y-ci)
- Config: `pa11y.json`
- Currently tests only 2 URLs

### Linting
```bash
# PHP:
composer run phpcs

# JS:
npm run lint

# CSS:
npm run stylelint
```

## Key Repos & Their Purpose

| Repo | What to contribute |
|------|-------------------|
| `planet4-master-theme` | Theme code, tests, accessibility, PHP |
| `planet4-plugin-gutenberg-blocks` | Block components, block.json migration |
| `planet4-child-theme-*` | NRO-specific fixes (CSS, config, translations) |
| `planet4-develop` | Dev tooling, CLI improvements |
| `planet4-docs` | Documentation, handbook |

## Communication
- Slack: `#planet4` on greenpeace.slack.com
- Jira: PLANET-XXXX (reference in commits when applicable)
- Active maintainer: `mleray` (Maud Leray)
- Friend contact: `sagarsdeshmukh` (Sagar Deshmukh, Nashik)

## PR Gotchas

For PR-time gotchas to this repo (pre-push checklist + lessons from past PRs), see [planet4-PR-Gotchas](./planet4-PR-Gotchas.md) in `./` (this folder). Check it before every `gh pr create`. After each PR closes, add new lessons there.

This is gate 2 of the contribution lifecycle. Gate 1 is [Research-Gotchas](../../core/Research-Gotchas.md) (qualify the repo BEFORE starting work).

## Quick Reference

### Files commonly edited
- `pa11y.json` — accessibility test URLs
- `phpcs.xml.dist` — PHP coding standards config
- `tests/e2e/blocks/` — Playwright E2E tests
- `tests/` — PHPUnit tests
- `assets/src/` — JS/CSS source files

### After first merge
- Add yourself to `.all-contributorsrc` in `greenpeace/planet4` (Issue #54)
