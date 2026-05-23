---
type: skill
name: Openverse Contribution Guide
description: Coding style, PR process, testing, and conventions for contributing to WordPress/openverse
---

# Openverse Contribution — Agent Skill Guide

## Repository
- Monorepo: https://github.com/WordPress/openverse
- Local clone: `<your-code-path>/openverse`
- Frontend source: `frontend/src/`
- Components: `frontend/src/components/`

## Identity (CRITICAL)
- Use personal GitHub: `<your-github>` (<your-email>)
- NEVER use work identity for this project
- Verify before every commit: `git config user.email`

## Development Workflow

### Setup
```bash
# Already cloned:
cd <your-code-path>/openverse

# Fork on GitHub first, then:
git remote add fork https://github.com/<your-github>/openverse.git

# Init dev environment (requires Docker/OrbStack):
./ov init

# Install Node deps:
ov just node-install

# Run frontend:
ov just frontend/run dev
# → http://localhost:8443
```

### Branch & PR Flow
1. Fork the repo on GitHub
2. Checkout branch from `main`, named after the issue: `fix/497-audio-alt-text`
3. Make changes, commit, push to your fork
4. Open PR from your fork to `WordPress/openverse:main`
5. Tag `@WordPress/openverse-maintainers` if you need assignment
6. Fill out PR template completely (testing instructions required!)

### PR Checklist (from Openverse docs)
- [ ] PR template filled correctly with testing instructions
- [ ] Passes linting: `ov just lint`
- [ ] Unit tests pass: `pnpm -r run test` (for JS changes)
- [ ] New tests added if applicable
- [ ] No Playwright snapshot changes unless intentional

## Coding Style

### TypeScript / Vue
- **Strict TypeScript** — `strict: true` in tsconfig
- **Vue 3 Composition API** with `<script setup lang="ts">`
- **No semicolons** — Prettier config: `semi: false`
- **Double quotes** — Prettier config: `singleQuote: false`
- **Trailing commas** — `"es5"` style
- **2-space indentation** (tabWidth: 2)
- **Tailwind CSS** — use utility classes, config at `frontend/tailwind.config.ts`
- **Pinia** for state management
- **`#imports`** for Nuxt auto-imports
- **`#shared/`** alias for shared utilities
- **`~/`** alias for frontend src root

### Component Conventions
- Component names: `V` prefix + PascalCase (e.g., `VAudioThumbnail`)
- Each component in its own folder: `VComponentName/VComponentName.vue`
- Stories in `meta/` subfolder: `VComponentName/meta/VComponentName.stories.ts`
- Props defined with `defineProps<{}>()` (TypeScript generic syntax)
- Use `useI18n` for all user-facing strings
- i18n keys follow dot notation: `audioThumbnail.alt`

### Accessibility (HIGH PRIORITY for Openverse)
- All images MUST have descriptive `alt` text
- Interactive elements need keyboard support (Enter, Space, arrow keys)
- Focus styles required on all interactive elements
- Test with VoiceOver on macOS
- Follow WAI-ARIA spec
- Read: https://make.wordpress.org/accessibility/handbook/

### File Structure
```
frontend/
├── src/
│   ├── components/     # Vue components (V-prefixed)
│   ├── composables/    # Vue composables (use-* pattern)
│   ├── stores/         # Pinia stores
│   ├── pages/          # Nuxt pages (file-based routing)
│   ├── layouts/        # Nuxt layouts
│   ├── middleware/      # Nuxt middleware
│   ├── plugins/        # Nuxt plugins
│   ├── utils/          # Utility functions
│   ├── data/           # Static data
│   ├── styles/         # Global styles
│   └── assets/         # Static assets
├── test/
│   ├── playwright/     # E2E + visual regression tests
│   ├── storybook/      # Storybook visual regression
│   └── unit/           # Unit tests (vitest)
├── i18n/               # Internationalization
└── shared/             # Shared types and utils
```

## Testing

### Unit Tests (Vitest)
```bash
ov just frontend/run test:unit
```
- Vue Testing Library preferred
- Legacy Vue Test Utils being phased out

### Playwright (E2E + Visual Regression)
```bash
# Run all Playwright tests (in Docker):
ov just frontend/run test:playwright

# Run locally (may have snapshot diffs on non-Linux):
ov just frontend/run test:playwright:local

# Debug mode (headed browser):
ov just frontend/run test:playwright:debug

# Update tapes (API responses):
ov just frontend/run test:playwright:update-tapes
```

### Storybook
```bash
# Run Storybook:
ov just frontend/run storybook

# Run Storybook visual regression:
ov just frontend/run test:storybook
```

### Visual Regression Snapshots
- Playwright CI runs with `-u` flag (auto-updates snapshots)
- If snapshots change, a GitHub comment with diff zip appears
- Apply: `ov unzip -p *_snapshot_diff.zip | git apply`

## Linting
- ESLint: custom `@openverse/eslint-plugin`
- Prettier: semi: false, double quotes, trailing commas es5, 2-space tabs
- Tailwind plugin for Prettier in frontend files
- Run: `ov just lint`

## Communication
- Slack: `#openverse` on Making WordPress Slack
- Ping maintainers: `@WordPress/openverse-maintainers` on GitHub
- No need to ask permission for `good first issue` or `help wanted` — just comment and start
- Weekly dev chat: Mondays @ 15:00 UTC

## SSR Considerations
- Openverse is a Nuxt SSR app
- Test both SSR (page reload) and CSR (client-side navigation)
- Be careful with `window`/`document` — may not exist during SSR
- Use `onMounted()` for browser-only code

## PR Gotchas

For PR-time gotchas to this repo (pre-push checklist + lessons from past PRs), see [openverse-PR-Gotchas](./openverse-PR-Gotchas.md) in `./` (this folder). Check it before every `gh pr create`. After each PR closes, add new lessons there.

This is gate 2 of the contribution lifecycle. Gate 1 is [Research-Gotchas](../../core/Research-Gotchas.md) (qualify the repo BEFORE starting work).
