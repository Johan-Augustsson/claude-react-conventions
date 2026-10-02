# claude-react-conventions

Shared React conventions for Claude Code, packaged as a plugin marketplace. One place to maintain React conventions used across all React repos.

**Stack:** Vite, React, TypeScript, Tailwind CSS v4, shadcn/ui, TanStack Router, Lucide icons, Zod, TanStack Query, TanStack Form, TanStack Table v9, ESLint 9, Prettier.

## Contents

- `plugins/react-conventions/skills/react-conventions/SKILL.md`: the conventions
- `plugins/react-conventions/skills/react-conventions/templates/`: shared `eslint.config.js`, `.prettierrc.json` and `.prettierignore`

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

Change the skill or templates here and bump `version` in `plugins/react-conventions/.claude-plugin/plugin.json`. Repos pick up the change on the next plugin update.
