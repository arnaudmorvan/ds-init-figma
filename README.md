# initDsWithClaude

Copilot Skill to create a **complete Design System in Figma** via the Plugin API and MCP tools.

## Principle

This skill guides an AI agent (Copilot / Claude) through a series of phases to generate a structured Figma Design System: variables, tokens, components, documentation pages, layouts, and showcase.

## Structure

```
SKILL.md              # Skill entry point
phases/
  01-setup.md         # Figma file creation, variables, tokens
  02-doc-components.md # Documentation components (Header, Section, etc.)
  03-layouts-templates.md # Page templates and layouts
  04-icons.md         # Icon system
  05-components.md    # UI components (Button, Input, Alert, etc.)
  06-foundations.md   # Foundations pages (colors, typography, spacing)
  07-showcase.md      # Showcase pages per component
  08-welcome.md       # DS welcome page
references/
  plugin-api-patterns.md # Tested Figma Plugin API patterns
  rules.md            # Rules and constraints
  style-guide.md      # Visual style guide
```

## Usage

1. Add this folder to `~/.copilot/skills/ds-init/`
2. In VS Code with Copilot (or any AI IDE with MCP support), invoke the skill with the project name
3. The skill asks configuration questions (fonts, colors, radii, shadows, spacing, breakpoints) then creates the Figma file phase by phase

## Prerequisites

- AI IDE with agent capabilities (VS Code + GitHub Copilot, Cursor, Windsurf, etc.)
- Figma MCP server configured (Plugin API access)
