# Changelog

Changes to the conventions that existing repos need to act on, newest first. Each repo records the date it last synced in its `CLAUDE.md` (`Conventions synced: YYYY-MM-DD`); apply every entry newer than that date, then update the date.

Entries say what changed and what to do in an existing repo. Changes that only affect new projects say so.

## 2026-10-20

- **Routing: TanStack Router → `react-router-dom` (data router).** Migrate:
  - `npm remove @tanstack/react-router @tanstack/router-plugin @tanstack/eslint-plugin-router` and `npm install react-router-dom`
  - Remove `tanstackRouter(...)` from `vite.config.ts`
  - Move the routes to route objects in `src/router.tsx` (`createBrowserRouter`, or `createHashRouter` for static hosting without SPA fallback) and render `<RouterProvider>` in `main.tsx`; route modules in `src/routes/` export the component and `loader` (see Routing in `SKILL.md`)
  - `beforeLoad` guards → `loader` with `throw redirect(...)`; `validateSearch` → `useSearchParams()` parsed with Zod; `notFoundComponent` → catch-all `*` route plus `errorElement` on the root route
  - Delete `src/routeTree.gen.ts` and `.prettierignore` (if it only listed the route tree)
  - `eslint.config.js`: remove the `@tanstack/eslint-plugin-router` import and config, and `src/routeTree.gen.ts` from `globalIgnores` (match `templates/eslint.config.js`)
  - `.github/workflows/claude.yml`: `Bash(npx tsr*)` can be removed from `--allowedTools` (a person must make this change; harmless if left)
- **Versions:** always install the latest version of each package (no version pins except where documented, e.g. `eslint@^9`) and write code for the installed version. Nothing to migrate.
- **Sync line:** add `Conventions synced: 2026-10-20` to the repo's `CLAUDE.md` once the entries above are applied.
