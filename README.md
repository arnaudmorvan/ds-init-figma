# ds-init-figma

A skill that creates a complete Design System directly on the Figma canvas — variables, tokens, components, documentation pages, layouts, and showcase — all via the Figma MCP server.

## What It Does

You provide a project name and answer configuration questions (font, colors, radii, shadows, spacing, breakpoints). The skill then executes 8+ phases to build a full Figma Design System file:

1. **Setup** — Creates the Figma file, pages, color/typography/size/spacing/radius/shadow variables with Light and Dark modes
2. **Doc Components** — Builds reusable documentation components (Header, Footer, Section Metadata, Divider)
3. **Layouts & Templates** — Creates layout components (PageLayout, SectionLayout, GridLayout) and SaaS/Landing templates
4. **Icons** — Imports SVG icons as a Component Set with 5 sizes × 6 color variants
5. **Components** — Builds UI Component Sets (Button, TextField, Select, Checkbox, Radio, Alert, Card, Badge, Avatar, Tag, Tabs, Modal) with full variant matrices
6. **Foundations** — Generates visual documentation for Colors, Typography, Size, Border, Spacing, Radius, Shadow, and Breakpoints
7. **Showcase** — Creates per-component showcase pages with all variants, states, and composition examples
8. **Welcome** — Builds a polished welcome page with hero, stats, feature cards, component highlights, and navigation
9. **Examples** — Composes realistic page mockups (Login, Dashboard, Settings, User List) using DS components

Each phase is verified by screenshot before moving to the next.

## Prerequisites

- A Figma account with a personal access token — get one from Figma → Account Settings → Personal Access Tokens
- An AI IDE with agent capabilities and MCP support (VS Code + GitHub Copilot, Cursor, Windsurf, Claude Code, etc.)
- The Figma MCP server connected to your AI agent

> This skill uses the Figma MCP server, not a Figma plugin. The AI agent reads and writes directly to your Figma file through the MCP connection — no plugin panel or Figma Community install required.

## Connecting the Figma MCP Server

### Using VS Code + GitHub Copilot

Add to your `.vscode/mcp.json`:

```json
{
  "servers": {
    "figma": {
      "type": "sse",
      "url": "https://mcp.figma.com/sse",
      "headers": {
        "Authorization": "Bearer YOUR_FIGMA_ACCESS_TOKEN"
      }
    }
  }
}
```

### Using Claude Code (terminal)

Add to your Claude config file (`~/.claude.json` or `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_FIGMA_ACCESS_TOKEN"
      }
    }
  }
}
```

Replace `YOUR_FIGMA_ACCESS_TOKEN` with your personal access token.

## Installation

1. Clone the repo

```bash
git clone https://github.com/arnaudmorvan/ds-init-figma.git
cd ds-init-figma
```

2. Add the skill to your AI agent

Place the folder in your skills directory. For GitHub Copilot:

```
~/.copilot/skills/ds-init/
```

## How to Trigger It

Use any of these prompts in your AI agent:

- `"Create a design system called MyApp"`
- `"Initialize a Figma DS for my project"`
- `"Build a design system with Inter font and blue primary color"`

The skill will ask 13 configuration questions (DS name, platform, font, colors, radius, shadows, dark mode, spacing, component tier, templates) then build the Figma file phase by phase.

## Structure

```
SKILL.md                          # Skill entry point
phases/
  01-setup.md                     # Figma file creation, variables, tokens
  02-doc-components.md            # Documentation components (Header, Section, etc.)
  03-layouts-templates.md         # Page templates and layouts
  04-icons.md                     # Icon system
  05-components.md                # UI components (Button, Input, Alert, etc.)
  06-foundations.md                # Foundations pages (colors, typography, spacing)
  07-showcase.md                  # Showcase pages per component
  08-welcome.md                   # DS welcome page
  09-examples.md                  # Example page compositions
references/
  plugin-api-patterns.md          # Tested Figma Plugin API patterns
  rules.md                        # Rules and constraints
  style-guide.md                  # Visual style guide
```

## MCP Tools Used

- `mcp_figma_create_new_file` — Create the DS file
- `mcp_figma_use_figma` — Execute Figma Plugin API code
- `mcp_figma_get_screenshot` — Verify visual output

## Contributing

Contributions are welcome! Here's how to get started:

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-improvement`
3. Make your changes
4. Test by running the skill end-to-end on a new Figma file
5. Open a pull request with a clear description of what changed and why

### Guidelines

- Keep the phase structure intact — each phase is designed to be self-contained and verifiable
- Any new component must follow the existing pattern: Component Set with variants, semantic color binding, size variables
- Do not commit personal access tokens or generated artifacts
- All Figma Plugin API code must follow the rules in `references/rules.md`

## License

[MIT](LICENSE)
