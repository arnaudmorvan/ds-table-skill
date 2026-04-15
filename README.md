# ds-table

A skill that generates **data tables in Figma** via MCP — column-first architecture with native slots (v3).

## What It Does

A Copilot skill (GitHub Copilot Agent) that builds complete data tables in Figma. It uses the Figma MCP to create tables with:

- **6 column types**: checkbox, text, avatar+text, badge, action buttons, icons
- **Native Figma slots**: td cells stay as connected DS instances (zero `detachInstance()`)
- **Zebra striping**, **sort indicators**, **empty states**
- **Densities**: MD (default) and SM (compact)

## Example

![Team Members Table](assets/example-table.png)

> Table generated with ds-table: checkbox, avatar+name, status badges, role, email, team badges, action icons — with zebra striping and sort indicator.

## Prerequisites

- A Figma account with a personal access token — get one from Figma → Settings → Security → Personal access tokens
- A **Figma Design System** with the Component Sets: `th`, `td`, `badge`, `button`, `checkbox`, `avatar-profile-photo`
- An AI IDE with agent capabilities and MCP support (VS Code + GitHub Copilot, Cursor, Windsurf, Claude Code, etc.)
- The Figma MCP server connected to your AI agent
- The Node IDs in `table.yaml` must match your Figma file

## Installation

### GitHub Copilot (VS Code)

1. Clone the repo into your skills directory:

```bash
cd ~/.copilot/skills
git clone https://github.com/arnaudmorvan/ds-table-skill.git ds-table
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
git clone https://github.com/arnaudmorvan/ds-table-skill.git ds-table
```

> **Figma token:** Go to Figma → Settings → Security → Personal access tokens → Generate new token.

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
git clone https://github.com/arnaudmorvan/ds-table-skill.git ds-table
```

### Alternative — Knowledge Base only

Copy `knowledge-base/` into your project folder and reference the YAML specs from your workflow.

## Structure

```
ds-table/
  SKILL.md                              ← Skill entry point
  README.md                             ← This file
  assets/
    example-table.png                   ← Example output screenshot
  knowledge-base/
    cspec/
      builders/
        table-row.yaml                  ← Row-first builder (default, recommended)
        table-column.yaml               ← Column-first builder (legacy)
      components/
        th.yaml                         ← Table Header Cell (variants, layout)
        td.yaml                         ← Table Data Cell (native slot pattern)
```

## How to Use

Ask your AI agent:

> "Create a Team Members table in Figma with 10 rows, columns: checkbox, name, status, role, email, teams, actions."

The skill picks the row-first layout by default. To use column-first, specify it:

> "Create an Orders table in Figma using column-first layout."

The skill will:
1. Read the appropriate YAML builder (row or column)
2. Generate 2 MCP calls (structure + complex content)
3. Build the table with zebra striping and sort indicators

## MCP Tools Used

- `mcp_figma_get_design_context` — Read DS component structure (th, td, badges)
- `mcp_figma_use_figma` — Generate and inject table via Figma Plugin API

## License

MIT
