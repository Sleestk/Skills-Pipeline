# Fast Refresh (v16.2.1)

Fast Refresh is a React feature integrated into Next.js that live-reloads the browser while **preserving temporary client-side state** on file save. Enabled by default in all Next.js apps v9.4+. Most edits are visible within a second.

---

## How It Works

Three behaviors depending on what you edit:

1. **File exports only React components** → Fast Refresh updates only that file and re-renders the component. Styles, rendering logic, event handlers, effects — all safe.

2. **File has non-React exports** → Fast Refresh re-runs that file AND all files importing it. E.g. editing `theme.js` (imported by `Button.js` and `Modal.js`) updates both components.

3. **File is imported by files outside the React tree** → Falls back to a **full reload**. This happens when a file exports both a React component and a value used by a non-React utility. Fix: extract the shared value to a separate file imported by both.

---

## Error Resilience

### Syntax errors
Fix and save → error disappears automatically. **Component state is preserved.** No reload needed.

### Runtime errors
A contextual overlay appears. Fix the error → overlay dismisses automatically, no reload.

- If error occurred **outside** rendering: state is retained.
- If error occurred **during** rendering: React remounts the app with updated code.

**Error boundaries**: They retry rendering on the next edit after a rendering error — which can prevent getting reset to root app state. Don't make them too granular; design them intentionally.

---

## Limitations — When State Resets

Fast Refresh preserves local React state only when safe. State **will reset** if:

- The component is a **class component** (only function components + Hooks preserve state)
- The file has **other exports** besides a React component
- The file exports the result of a **higher-order component** (`HOC(WrappedComponent)`) that returns a class
- The component is an **anonymous arrow function**: `export default () => <div />;`
  - Fix: use the [`name-default-component` codemod](https://nextjs.org/docs/pages/guides/upgrading/codemods#name-default-component)

As more of your codebase moves to function components + Hooks, state preservation improves.

---

## Tips

- Add `// @refresh reset` anywhere in a file to **force remount** on every edit — useful when tweaking mount-only animations.
- `console.log` and `debugger;` work fine inside components during development.
- **Imports are case-sensitive.** `'./header'` vs `'./Header'` will cause Fast Refresh (and full refresh) to fail.

---

## Fast Refresh and Hooks

- `useState` and `useRef` **preserve their previous values** across edits, as long as you don't change argument count or Hook order.
- Hooks with dependencies (`useEffect`, `useMemo`, `useCallback`) **always re-run** during Fast Refresh — dependency lists are ignored. This ensures edits reflect on screen immediately.
- A `useEffect` with an empty dependency array will still re-run once during Fast Refresh. Writing code resilient to occasional re-runs is good practice — and is enforced by React Strict Mode (recommended to enable).