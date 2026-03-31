# Accessibility

> Next.js 16.2.1 — https://nextjs.org/docs/architecture/accessibility

---

## Route Announcements

Next.js includes a **route announcer** by default for client-side transitions via `next/link`. Screen readers and assistive technology are notified on every navigation.

The announcer checks for the page name in this order:
1. `document.title`
2. The `<h1>` element
3. The URL pathname

**Best practice:** Give every page a unique, descriptive `<title>` for the best screen reader experience.

---

## Linting (eslint-plugin-jsx-a11y)

Next.js includes [`eslint-plugin-jsx-a11y`](https://github.com/jsx-eslint/eslint-plugin-jsx-a11y) out of the box. It warns on common accessibility issues:

- `aria-props` — Invalid ARIA attributes
- `aria-proptypes` — Invalid ARIA attribute values
- `aria-unsupported-elements` — ARIA on unsupported elements
- `role-has-required-aria-props` — Missing required ARIA props for a role
- `role-supports-aria-props` — Incompatible ARIA props for a role

Also enforces: `alt` text on `<img>`, correct `aria-*` usage, correct `role` attributes.

---

## Accessibility Resources

- [WebAIM WCAG Checklist](https://webaim.org/standards/wcag/checklist)
- [WCAG 2.2 Guidelines](https://www.w3.org/TR/WCAG22/)
- [The A11y Project](https://www.a11yproject.com/)
- [Color contrast ratios (MDN)](https://developer.mozilla.org/docs/Web/Accessibility/Understanding_WCAG/Perceivable/Color_contrast)
- [`prefers-reduced-motion`](https://web.dev/prefers-reduced-motion/) — Respect reduced motion preferences in animations