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

## Installation

### GitHub Copilot (VS Code)

1. Clone the repo into your skills directory:

```bash
cd ~/.copilot/skills
git clone https://github.com/arnaudmorvan/ds-init-figma.git ds-init
```

2. Add the Figma MCP server to your `.vscode/mcp.json`:

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

3. The skill is available immediately — no restart needed.

### Claude Desktop

1. **Open the Claude Desktop config file:**

```bash
open ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

2. **Make sure the Figma MCP server is connected.** Add or verify:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/figma-mcp-server@latest"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "YOUR_FIGMA_ACCESS_TOKEN"
      }
    }
  }
}
```

3. **Clone the repo** into your skills directory (find the `skills/user` path in your config):

```bash
cd /path/to/your/skills/user
git clone https://github.com/arnaudmorvan/ds-init-figma.git ds-init
```

4. **Restart Claude Desktop.** Skills are loaded at startup.

### Claude Code (terminal)

1. Add the Figma MCP server to `~/.claude.json`:

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

2. Clone the repo:

```bash
cd ~/.copilot/skills
git clone https://github.com/arnaudmorvan/ds-init-figma.git ds-init
```

> **Figma token:** Go to Figma → Settings → Security → Personal access tokens → Generate new token.

## How to Trigger It

The skill supports **4 execution modes** depending on what you ask:

### Full creation (new DS from scratch)
- `"Create a design system called MyApp"`
- `"Initialize a Figma DS for my project"`
- `"Build a design system with Inter font and blue primary color"`

The skill will ask 13 configuration questions then build the Figma file phase by phase.

### Add a component to an existing DS
- `"Add a Tabs component to <figma-link>"`
- `"Create Card, Badge, and Avatar components"`
- `"Add all Tier 2 components to my DS"`

### Generate a specific page
- `"Generate the Welcome page on <figma-link>"`
- `"Build the Foundations page"`
- `"Create example pages (Login, Dashboard, Settings)"`

### Run a specific phase
- `"Run phase 6 (Foundations)"`
- `"Execute phase 3 (Layouts)"`

For partial execution, provide a Figma file link. The skill will discover existing content and only create what's missing.

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
