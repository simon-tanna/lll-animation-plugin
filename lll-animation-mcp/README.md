# LLL Animation MCP — Notion Integration

Optional companion plugin for [`lll-animation`](../README.md). Adds the Notion MCP server and `/fetch-spec` command for pulling the canonical workshop spec directly into your Claude Code session.

## Installation

Requires the core plugin to be installed first.

```bash
claude /plugin install lll-animation-mcp@lll-animation-mcp-dev
```

Then restart Claude Code. Notion OAuth is handled automatically on first use.

## What's Included

- **`/fetch-spec [task-or-section]`** — Fetch the workshop spec from Notion. Use `/fetch-spec 1.3` for a specific task or `/fetch-spec` for the full overview.
- **Notion MCP server** — HTTP connection to [mcp.notion.com](https://mcp.notion.com/mcp).
