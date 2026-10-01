---
name: react-conventions
description: Shared React conventions for all React repos - component structure, hooks, state management, accessibility, ESLint/Prettier setup and lint rules. Use when creating, changing or reviewing React code, setting up a new React project, or configuring ESLint/Prettier in a React repo.
---

# React conventions

Follow these conventions for all React code. Repo-specific instructions in the repo's `CLAUDE.md` take precedence.

## Stack

- Vite + React, function components and hooks only (no class components)
- JavaScript unless the repo already uses TypeScript
- Plain CSS or CSS Modules; no UI library unless the repo already uses one
- npm as package manager

## Project structure

```
src/
  components/     # one component per folder or file, PascalCase: TodoItem.jsx
  hooks/          # custom hooks, camelCase with use-prefix: useTodos.js
  utils/          # pure helper functions, no React imports
  App.jsx
  main.jsx
```

- One component per file; file name matches the component name
- Co-locate component styles: `TodoItem.jsx` + `TodoItem.module.css`
- Named exports for components and hooks; default export only where a tool requires it

## Components

- Small and focused: if a component does two things, split it
- Props destructured in the signature: `function TodoItem({ todo, onToggle })`
- Event handler props named `onX`, internal handlers named `handleX`
- No business logic in JSX: compute values above the `return`
- Lists use a stable unique `key` (an id, never the array index); generate ids with `crypto.randomUUID()`
- Form inputs are controlled components

## State and hooks

- Keep state as close as possible to where it is used; lift only when needed
- Derive values instead of storing them (filtered lists, counts, flags): no duplicated state
- Use `useReducer` when state has several related updates; put the reducer in a custom hook (e.g. `useTodos`)
- Side effects (localStorage, fetch, subscriptions) go in custom hooks, not in components
- `useEffect` only for syncing with external systems, never for deriving state from props/state
- Follow the Rules of Hooks; never silence `react-hooks/exhaustive-deps` without a comment explaining why
- Avoid premature `useMemo`/`useCallback`; add them only for a measured problem or a stable reference that is actually needed

## Accessibility

- Semantic HTML: `<button>` for actions, `<a>` for navigation, `<ul>/<li>` for lists, `<form>` for forms
- Every input has a `<label>` (visible or `aria-label` for icon-only controls)
- Everything usable with keyboard only; visible focus styles (never `outline: none` without a replacement)
- Sufficient color contrast; don't use color as the only signal

## ESLint and Prettier

Lint rules live in config, not in prose. Use the templates in this skill's `templates/` folder:

- `templates/eslint.config.js` - flat config with `@eslint/js`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`, `eslint-plugin-jsx-a11y` and `eslint-config-prettier`
- `templates/.prettierrc.json` - Prettier settings

When setting up a repo:

1. Copy both templates to the repo root
2. Install dev dependencies:
   ```
   npm install -D eslint @eslint/js globals eslint-plugin-react-hooks eslint-plugin-react-refresh eslint-plugin-jsx-a11y eslint-config-prettier prettier
   ```
3. Add npm scripts:
   ```json
   "lint": "eslint . --max-warnings 0",
   "lint:fix": "eslint . --fix",
   "format": "prettier --write .",
   "format:check": "prettier --check ."
   ```

Rules for all changes:

- `npm run lint` and `npm run build` must pass before committing
- Don't disable lint rules inline unless unavoidable, and always with a comment explaining why
- Don't change the shared ESLint/Prettier config in a single repo; change it in the conventions repo instead

## Reviewing React code

When reviewing, check in this order:

1. Bugs: incorrect state updates, missing/unstable keys, stale closures, effects with wrong dependencies
2. Conventions above: structure, naming, derived state, hooks usage
3. Accessibility
4. Readability; skip pure style nits that Prettier handles
