# Debugging

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/debugging

## VS Code

Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug server-side",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev -- --inspect"
    },
    {
      "name": "Next.js: debug client-side",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000"
    },
    {
      "name": "Next.js: debug full stack",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/next/dist/bin/next",
      "runtimeArgs": ["--inspect"],
      "skipFiles": ["<node_internals>/**"],
      "serverReadyAction": {
        "action": "debugWithEdge",
        "killOnServerStop": true,
        "pattern": "- Local:.+(https?://.+)",
        "uriFormat": "%s",
        "webRoot": "${workspaceFolder}"
      }
    }
  ]
}
```

- Firefox debugging requires the [Firefox Debugger extension](https://marketplace.visualstudio.com/items?itemName=firefox-devtools.vscode-firefox-debug)
- Change `debugWithEdge` to `debugWithChrome` for Chrome
- For Turborepo, add `"cwd": "${workspaceFolder}/apps/web"` to server-side config

**Debug panel:** `Ctrl+Shift+D` (Win/Linux) or `⇧+⌘+D` (macOS) → select config → `F5`

## Browser DevTools

### Client-side
Run `next dev`, open `http://localhost:3000`, open DevTools → **Sources** tab. Search files: `Ctrl+P` / `⌘+P`. Source files have paths starting with `webpack://_N_E/./`.

### Server-side
Start with `--inspect` flag:
```bash
npm run dev -- --inspect
# or: NODE_OPTIONS=--inspect-brk next dev
```

**Chrome:** `chrome://inspect` → Remote Target → **inspect** → Sources tab. Paths start with `webpack://{application-name}/./`.

**Firefox:** `about:debugging` → This Firefox → Remote Targets → **Inspect** → Debugger tab.

**Docker:** Use `--inspect=0.0.0.0` for remote debugging access.

### Inspect Server Errors in Browser
Click the Node.js icon on the error overlay to copy the DevTools URL.

## JetBrains WebStorm
Create a `JavaScript Debug` configuration with `http://localhost:3000`, run it — browser opens automatically.

## React Developer Tools
Install the browser extension for inspecting components, editing props/state, identifying performance issues.

## Windows: Disable Windows Defender
Windows Defender can significantly slow down Fast Refresh. Add project folder to exclusion list in Windows Security → Virus & threat protection → Manage settings → Add or remove exclusions.