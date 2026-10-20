# claude-react-conventions

Shared React conventions for Claude Code, packaged as a plugin marketplace. One place to maintain React conventions used across all React repos.

**Stack:** Vite, React, TypeScript, Tailwind CSS, shadcn/ui, React Router, Lucide icons, Zod, TanStack Query, TanStack Form, TanStack Table, ESLint, Prettier.

## Contents

- `plugins/react-conventions/skills/react-conventions/SKILL.md`: the conventions
- `plugins/react-conventions/skills/react-conventions/CHANGELOG.md`: what changed and what existing repos need to do
- `plugins/react-conventions/skills/react-conventions/templates/`: shared `eslint.config.js` and `.prettierrc.json`
- `plugins/react-conventions/skills/react-conventions/templates/github-workflows/`: `claude.yml` (with the allowed tools Claude needs), `claude-code-review.yml` and `ci.yml`

## Use in a repo

Add to the repo's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "react-conventions": {
      "source": { "source": "github", "repo": "Johan-Augustsson/claude-react-conventions" }
    }
  },
  "enabledPlugins": {
    "react-conventions@react-conventions": true
  }
}
```

In GitHub workflows using `anthropics/claude-code-action`, add:

```yaml
plugin_marketplaces: https://github.com/Johan-Augustsson/claude-react-conventions.git
plugins: react-conventions@react-conventions
```

## Updating

Change the skill or templates here, add an entry to `plugins/react-conventions/skills/react-conventions/CHANGELOG.md` saying what existing repos need to do, and push to `main`. The plugin has no `version` field, so each commit counts as a new version and repos pick up the change on the next plugin update.
