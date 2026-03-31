# Next.js MCP Server

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/mcp

Next.js 16+ includes a built-in MCP endpoint at `/_next/mcp`. The `next-devtools-mcp` package connects coding agents to your running dev server.

## Setup (Next.js 16+)

Add to `.mcp.json` at project root:
```json
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```

Start dev server (`pnpm dev`) — `next-devtools-mcp` auto-discovers and connects.

## Available Tools

- **`get_errors`** — build errors, runtime errors, type errors from dev server
- **`get_logs`** — path to dev log file (browser console + server output)
- **`get_page_metadata`** — routes, components, rendering info for specific pages
- **`get_project_metadata`** — project structure, config, dev server URL
- **`get_routes`** — all routes grouped by router type (appRouter, pagesRouter). Dynamic segments shown as `[param]` or `[...slug]`
- **`get_server_action_by_id`** — look up Server Actions by ID to find source file and function name

## Next.js Knowledge Base Tools
- Query Next.js documentation and best practices
- Migration helpers with codemods for upgrading to v16
- Caching guide for Cache Components setup
- Browser testing via Playwright MCP integration

## Example Workflow

```bash
User: "What errors are currently in my application?"
# Agent calls get_errors → retrieves current build/runtime/type errors → provides fixes
```

```bash
User: "Help me upgrade my Next.js app to version 16"
# Agent analyzes version, guides through codemods, handles breaking changes
```

## How It Works

Next.js 16 includes MCP endpoint at `/_next/mcp` in the dev server. `next-devtools-mcp` discovers Next.js instances on different ports and forwards tool calls to the appropriate server.

## Troubleshooting
- Ensure Next.js v16+
- Verify `.mcp.json` is configured
- Start dev server first
- Restart dev server if it was already running when config was added