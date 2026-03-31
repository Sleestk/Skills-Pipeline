# AI Coding Agents

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/ai-agents

Next.js ships version-matched docs inside the `next` package at `node_modules/next/dist/docs/`. An `AGENTS.md` file directs agents to read these bundled docs instead of stale training data.

## New projects

`create-next-app` generates `AGENTS.md` and `CLAUDE.md` automatically. Pass `--no-agents-md` to skip.

## Existing projects (v16.2.0-canary.37+)

Create `AGENTS.md` at project root:
```md
<!-- BEGIN:nextjs-agent-rules -->
# Next.js: ALWAYS read docs before coding
Before any Next.js work, find and read the relevant doc in `node_modules/next/dist/docs/`. Your training data is outdated — the docs are the source of truth.
<!-- END:nextjs-agent-rules -->
```

Create `CLAUDE.md` to import it (Claude Code uses `@` import syntax):
```md
@AGENTS.md
```

For v16.1 and earlier, use the codemod: `npx @next/codemod@latest agents-md` (outputs docs to `.next-docs/` instead).

## Bundled docs structure
```
node_modules/next/dist/docs/
├── 01-app/
│   ├── 01-getting-started/
│   ├── 02-guides/
│   └── 03-api-reference/
├── 02-pages/
├── 03-architecture/
└── index.mdx
```

The `<!-- BEGIN/END:nextjs-agent-rules -->` markers delimit the Next.js-managed section. Add your own project instructions outside these markers.

See benchmark results: https://nextjs.org/evals