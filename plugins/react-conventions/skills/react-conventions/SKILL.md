---
name: react-conventions
description: Shared React conventions for all React repos - TypeScript, Tailwind, Lucide icons, Zod, TanStack Query/Form/Table, component structure, hooks, accessibility and ESLint/Prettier setup. Use when creating, changing or reviewing React code, setting up a new React project, or configuring ESLint/Prettier/Tailwind in a React repo.
---

# React conventions

Follow these conventions for all React code. Repo-specific instructions in the repo's `CLAUDE.md` take precedence.

## Stack

| Concern | Use |
|---|---|
| Build | Vite + React, function components and hooks only |
| Language | TypeScript (strict) - no `.js`/`.jsx` source files |
| Styling | Tailwind CSS v4 via `@tailwindcss/vite` |
| Icons | `lucide-react` |
| Validation / schemas | `zod` (v4) |
| Server / async state | `@tanstack/react-query` |
| Forms | `@tanstack/react-form` + Zod schemas |
| Tables / data grids | `@tanstack/react-table` (v9) |
| Lint / format | ESLint 9 + Prettier (templates in `templates/`) |
| Package manager | npm |

Don't add other libraries for these concerns (no other UI kits, icon sets, form, validation or data-fetching libraries) without asking.

### Up-to-date API docs

TanStack packages ship agent skills with version-correct API docs. Before using an API you're unsure about, read them from `node_modules`:

- `node_modules/@tanstack/react-table/skills/*/SKILL.md` and `node_modules/@tanstack/table-core/skills/*/SKILL.md`
- Check `node_modules/@tanstack/react-form/` and `node_modules/@tanstack/react-query/` for a `skills/` folder too

Prefer these over remembered examples: the APIs change between major versions.

## Project setup

1. `npm create vite@latest <name> -- --template react-ts`
2. The Vite template ships `oxlint`: remove it (`npm remove oxlint`, delete its config) and set up ESLint as described below
3. Tailwind: `npm install tailwindcss @tailwindcss/vite`, add `tailwindcss()` to `plugins` in `vite.config.ts`, and replace `src/index.css` with `@import "tailwindcss";`
4. Install: `npm install lucide-react clsx zod @tanstack/react-query @tanstack/react-form` (+ `@tanstack/react-table` when the app has tables)
5. Add to `compilerOptions` in `tsconfig.app.json`: `"strict": true`, `"noUncheckedIndexedAccess": true`
6. Delete the template's demo content (`App.css`, logos, counter)

## Project structure

Group by feature, not by file type:

```
src/
  features/
    todos/
      components/     # TodoList.tsx, TodoItem.tsx, TodoForm.tsx
      hooks/          # useTodos.ts (query/mutation hooks)
      api.ts          # data access functions (fetch, localStorage, ...)
      queries.ts      # query key factory + queryOptions
      schemas.ts      # Zod schemas + inferred types
  components/ui/      # shared, feature-agnostic components (Button, Input, ...)
  lib/                # queryClient.ts, generic helpers (no feature logic)
  App.tsx
  main.tsx
  index.css
```

- One component per file; file name matches the component (PascalCase `.tsx`); hooks are camelCase with `use` prefix (`.ts`)
- Named exports for components and hooks; default export only where a tool requires it
- Features import from `components/ui` and `lib`, never from another feature's internals

## TypeScript

- No `any`; use `unknown` and narrow (ideally with a Zod schema)
- Types come from Zod schemas where data has a schema: `type Todo = z.infer<typeof todoSchema>`; don't write a duplicate interface
- Props typed with a `type XProps = {...}` next to the component; no `React.FC`
- `import type` for type-only imports (enforced by ESLint)
- No non-null assertions (`!`) or `as` casts to silence errors; fix the type instead

## Components

- Small and focused: if a component does two things, split it
- Props destructured in the signature: `function TodoItem({ todo, onToggle }: TodoItemProps)`
- Event handler props named `onX`, internal handlers named `handleX`
- No business logic in JSX: compute values above the `return`
- Lists use a stable unique `key` (an id, never the array index); generate ids with `crypto.randomUUID()`

## Styling (Tailwind)

- Tailwind utility classes in JSX; no separate CSS files except `index.css` (Tailwind import, `@theme` tokens, base styles)
- Design tokens (colors, fonts, spacing additions) in `@theme` in `index.css`, not hard-coded hex values in classes
- Conditional classes with `clsx`; don't build class names by string concatenation (`text-${color}-500` breaks Tailwind's scanner)
- Repeated class combinations become a component in `components/ui`, not `@apply`
- Class order is handled by `prettier-plugin-tailwindcss`; don't sort by hand
- Support dark mode with `dark:` variants when the repo uses it

## Icons (Lucide)

- Named imports only: `import { Trash2 } from 'lucide-react'` (never import the whole icon set or use dynamic icon names)
- Size and color with Tailwind: `<Trash2 className="size-4 text-red-600" />`
- Decorative icons next to text: `aria-hidden="true"`
- Icon-only buttons need an accessible name: `<button aria-label="Delete todo"><Trash2 aria-hidden="true" /></button>`

## Zod

- Schemas live in the feature's `schemas.ts`; export the schema and its inferred type
- Validate all data crossing a trust boundary with `schema.parse`/`safeParse`: API responses, `localStorage`, URL/search params, form input
- Never trust `JSON.parse` output without a schema
- Reuse schemas between forms and API layer (`.pick`, `.omit`, `.extend`) instead of duplicating them

## Data and state

- **Server / async state** (anything from an API or storage layer): TanStack Query. Never copy query data into `useState`
- **Form state**: TanStack Form
- **Local UI state** (open/closed, selected tab, filter): `useState`/`useReducer` in the closest component that needs it
- **URL state** (filters, pagination worth sharing/bookmarking): search params
- Derive values instead of storing them (filtered lists, counts, flags)
- `useEffect` only for syncing with external systems, never for deriving state or fetching data
- Avoid premature `useMemo`/`useCallback`; add them only for a measured problem or a reference that must be stable (e.g. TanStack Table `columns`/`data`)

### TanStack Query

- One `QueryClient` in `lib/queryClient.ts`, provided in `main.tsx` with `QueryClientProvider`
- Query keys from a factory per feature in `queries.ts`; use `queryOptions()` so keys and fetchers stay together:
  ```ts
  export const todoQueries = {
    all: () => queryOptions({ queryKey: ['todos'], queryFn: fetchTodos }),
  };
  ```
- Data access functions in `api.ts` are plain async functions that validate their result with Zod
- Components use feature hooks (`useTodos`, `useAddTodo`) rather than calling `useQuery` with inline options
- Mutations invalidate or update the affected queries in `onSuccess`/`onSettled`; use optimistic updates where the UI should feel instant
- Handle `isPending` and `isError` states in the UI; no silent failures

### TanStack Form

- `useForm` with `defaultValues` and a Zod schema as validator (Zod implements Standard Schema, no adapter needed):
  ```tsx
  const form = useForm({
    defaultValues: { title: '' },
    validators: { onChange: todoFormSchema },
    onSubmit: async ({ value }) => addTodo.mutateAsync(value),
  });
  ```
- Render fields with `form.Field`; show `field.state.meta.errors` next to the input, linked with `aria-describedby`
- `<form onSubmit>` calls `e.preventDefault()` then `form.handleSubmit()`
- When a repo has several forms, create shared field components with `createFormHook` in `components/ui/form`

### TanStack Table (v9)

- Use for tabular data with sorting, filtering or pagination; a simple list is a `<ul>`, not a table
- v9 API: `useTable` + `tableFeatures(...)` + `createColumnHelper<typeof features, TData>()`. Don't use v8 APIs (`useReactTable`, `getCoreRowModel`); read `node_modules/@tanstack/react-table/skills/` (incl. `migrate-v8-to-v9`)
- Register only the features the table uses
- `columns` defined outside the component or memoized; `data` stable (from a query or memoized)
- Render semantic `<table>`/`<thead>`/`<tbody>` markup; sortable headers are `<button>`s with `aria-sort` on the `<th>`

## Accessibility

- Semantic HTML: `<button>` for actions, `<a>` for navigation, `<ul>/<li>` for lists, `<form>` for forms
- Every input has a `<label>` (visible, or `aria-label` for icon-only controls)
- Everything usable with keyboard only; visible focus styles (`focus-visible:` utilities; never remove outlines without a replacement)
- Sufficient color contrast; don't use color as the only signal
- Loading and error messages are announced (`role="status"` / `role="alert"`)

## ESLint and Prettier

Lint rules live in config, not in prose. Use the templates in this skill's `templates/` folder:

- `templates/eslint.config.js` - flat config: `typescript-eslint` (type-checked), `react-hooks`, `react-refresh`, `jsx-a11y`, `@tanstack/eslint-plugin-query`, `eslint-config-prettier`
- `templates/.prettierrc.json` - Prettier settings with `prettier-plugin-tailwindcss`

When setting up a repo:

1. Copy both templates to the repo root
2. Install dev dependencies. ESLint is pinned to v9 because `eslint-plugin-jsx-a11y` doesn't support ESLint 10 yet:
   ```
   npm install -D eslint@^9 @eslint/js@^9 globals typescript-eslint eslint-plugin-react-hooks eslint-plugin-react-refresh eslint-plugin-jsx-a11y @tanstack/eslint-plugin-query eslint-config-prettier prettier prettier-plugin-tailwindcss
   ```
3. npm scripts:
   ```json
   "typecheck": "tsc -b",
   "lint": "eslint . --max-warnings 0",
   "lint:fix": "eslint . --fix",
   "format": "prettier --write .",
   "format:check": "prettier --check ."
   ```

Rules for all changes:

- `npm run typecheck`, `npm run lint` and `npm run build` must pass before committing
- Don't disable lint rules or use `@ts-expect-error` unless unavoidable, and always with a comment explaining why
- Don't change the shared ESLint/Prettier config in a single repo; change it in the conventions repo instead

## Reviewing React code

When reviewing, check in this order:

1. Bugs: incorrect state updates, missing/unstable keys, stale closures, effects with wrong dependencies, unhandled query/mutation errors
2. Type safety: `any`, unsafe casts, external data not validated with Zod
3. Conventions above: structure, library usage (Query/Form/Table/Zod/Lucide/Tailwind), derived state
4. Accessibility
5. Readability; skip pure style nits that Prettier handles
