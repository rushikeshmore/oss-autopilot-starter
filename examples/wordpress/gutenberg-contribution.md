---
type: skill
name: Gutenberg Contribution Guide
description: Coding style, PR process, testing, and conventions for contributing to WordPress/gutenberg
---

# Gutenberg Contribution - Agent SOP

## Repository
- Monorepo: https://github.com/WordPress/gutenberg
- Local clone: `<your-code-path>/gutenberg`
- Fork: `<your-github>/gutenberg`

## Identity (CRITICAL)
- Personal GitHub: `<your-github>` (<your-email>)
- NEVER use work identity

## Coding Style

### JavaScript / TypeScript
- **Tabs for indentation** (not spaces)
- **Single quotes** for strings (unless string contains single quote)
- **WordPress element imports**: Use `@wordpress/element` not `react` directly
- **Import organization** with multi-line comment separators:
  ```js
  /**
   * External dependencies
   */
  import clsx from 'clsx';

  /**
   * WordPress dependencies
   */
  import { useState } from '@wordpress/element';

  /**
   * Internal dependencies
   */
  import MyComponent from './my-component';
  ```
- ES6 shorthand for object properties
- Template strings over concatenation

### CSS
- BEM-inspired naming: `package-directory__descriptor` (e.g., `components-notice__content`)
- State modifiers: `is-*` prefix (e.g., `is-active`)
- Never use a component's class outside its own folder

### React Components (MANDATORY)
- **Function components with hooks ONLY** (no class components)
- Class components acceptable ONLY for Error Boundaries
- Use `forwardRef` for components that need ref forwarding
- Props typed with TypeScript interfaces
- JSDoc with `@example` tag for public APIs

### TypeScript
- Avoid `any`, prefer `unknown`
- Use `WordPressComponentProps` for component prop types
- Strict mode enabled

## Class-to-Hooks Conversion Pattern

### Lifecycle Mapping
| Class Pattern | Hooks Equivalent |
|--------------|-----------------|
| `constructor` + `this.state` | `useState()` |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| `componentWillUnmount` | `useEffect` cleanup: `return () => {}` |
| `this.method.bind(this)` | Remove (not needed in function components) |
| `this.container` (instance var) | `useRef()` |
| `forwardRef` wrapper class | `forwardRef` on the function directly |

### Conversion Checklist
- [ ] All lifecycle methods converted to hooks
- [ ] Instance variables converted to `useRef`
- [ ] State converted to `useState`
- [ ] `this.method.bind(this)` removed (use direct function references)
- [ ] `forwardRef` pattern simplified (no wrapper class needed)
- [ ] Props destructured from function params (not `this.props`)
- [ ] All existing tests still pass
- [ ] No behavioral changes (pure refactor)
- [ ] TypeScript types preserved/improved

## Testing

### Framework
- **Jest** as test runner
- **React Testing Library** for component tests (NOT Enzyme)
- **Playwright** for E2E tests
- **user-event** for simulating interactions (NOT `fireEvent`)

### Running Tests
```bash
npm run test:unit                                    # All unit tests
npm run test:unit -- --testPathPattern "navigable"  # Specific test
npm run test:unit:watch                              # Watch mode
npm run lint                                         # ESLint + TS
npm run lint:js:fix                                  # Auto-fix
```

### Test File Location
```
component/
  src/
    index.tsx
  test/
    index.tsx       # Tests live here
```

### Test Best Practices (per WP guidelines)
- Use `screen.getByRole()` not implementation selectors
- Use `userEvent.setup()` then `user.keyboard()`, `user.click()`
- Test from user perspective, not implementation details
- Test keyboard navigation for accessibility
- Test edge cases (empty, null, overflow)

## PR Process

### Branch Naming
```
[type]/[change]
Prefixes: add/, try/, update/, remove/, fix/
Example: refactor/navigable-container-hooks
```

### Commit Messages
- Clear, descriptive subject line
- Reference issue: "Closes #22890" or "Part of #22890"
- Explain why, not just what
- Atomic commits (one logical change)

### PR Template (MUST follow)
```markdown
## What?
Closes #ISSUE

## Why?
[Problem being solved]

## How?
[Implementation details]

## Testing Instructions
[Step by step]

### Testing Instructions for Keyboard
[Required for UI changes]

## Use of AI Tools
[Disclose AI usage per WP AI Guidelines]
```

### AI Disclosure (REQUIRED by WordPress)
WordPress requires disclosure of AI tool usage. Include in PR:
```
## Use of AI Tools
Claude Code was used to assist with this refactoring. All changes were
reviewed and tested manually.
```

### PR Labels
- Maintainers add labels, not contributors
- But mention in PR if it's related to tracking issues

## Changelog
Every production code change needs a CHANGELOG.md entry:
```markdown
## Unreleased

### Internal
- Refactored `NavigableContainer` from class component to function component with hooks ([#XXXX](https://github.com/WordPress/gutenberg/pull/XXXX)).
```

## Key Directories
```
packages/
  components/
    src/
      component-name/
        index.tsx          # Main component
        types.ts           # TypeScript types
        styles.ts          # Emotion styles
        test/
          index.tsx        # Tests
        stories/
          index.tsx        # Storybook stories
        README.md          # Docs
    CHANGELOG.md           # Package changelog
```

## Communication
- Slack: `#core-editor`, `#core-js` on Making WordPress Slack
- GitHub: Tag `@WordPress/gutenberg-core` for reviews
- Tracking issue for class-to-hooks: #22890 - comment before starting

## PR Gotchas

For PR-time gotchas to this repo (pre-push checklist + lessons from past PRs), see [gutenberg-PR-Gotchas](./gutenberg-PR-Gotchas.md) in `./` (this folder). Check it before every `gh pr create`. After each PR closes, add new lessons there.

This is gate 2 of the contribution lifecycle. Gate 1 is [Research-Gotchas](../../core/Research-Gotchas.md) (qualify the repo BEFORE starting work).

## Do NOT
- Change behavior during refactor (pure refactor only)
- Touch `.native.js` files (mobile team's domain)
- Skip AI disclosure in PR
- Use `fireEvent` (use `userEvent` instead)
- Import from `react` directly (use `@wordpress/element`)
