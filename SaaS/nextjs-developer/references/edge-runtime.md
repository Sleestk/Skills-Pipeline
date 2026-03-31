# Next.js Edge Runtime Reference

> Version: 16.2.1 | Source: https://nextjs.org/docs/app/api-reference/edge

---

## Overview

Next.js has two server runtimes:

| | Node.js Runtime (default) | Edge Runtime |
|---|---|---|
| **APIs** | Full Node.js API surface | Limited web-standard subset |
| **Used in** | App Router rendering, Route Handlers | `proxy.ts` only |
| **ISR support** | ✅ | ❌ |
| **Streaming** | ✅ (adapter-dependent) | ✅ (adapter-dependent) |
| **Filesystem access** | ✅ | ❌ |
| **`require()`** | ✅ | ❌ (use ESM imports) |
| **`node_modules`** | ✅ | Only if ESM + no native Node.js APIs |

The Edge Runtime is intentionally restricted for fast cold starts and global distribution. Use it only in `proxy.ts` and only when you need sub-millisecond startup or geographic routing.

---

## Supported APIs

### Network
`Blob`, `fetch`, `FetchEvent`, `File`, `FormData`, `Headers`, `Request`, `Response`, `URLSearchParams`, `WebSocket`

### Encoding
`atob`, `btoa`, `TextDecoder`, `TextDecoderStream`, `TextEncoder`, `TextEncoderStream`

### Streams
`ReadableStream`, `ReadableStreamBYOBReader`, `ReadableStreamDefaultReader`, `TransformStream`, `WritableStream`, `WritableStreamDefaultWriter`

### Crypto
`crypto`, `CryptoKey`, `SubtleCrypto`

### Web Standards
`AbortController`, `Array`, `ArrayBuffer`, `Atomics`, `BigInt`, `BigInt64Array`, `BigUint64Array`, `Boolean`, `clearInterval`, `clearTimeout`, `console`, `DataView`, `Date`, `decodeURI`, `decodeURIComponent`, `DOMException`, `encodeURI`, `encodeURIComponent`, `Error`, `EvalError`, `Float32Array`, `Float64Array`, `Function`, `Infinity`, `Int8Array`, `Int16Array`, `Int32Array`, `Intl`, `isFinite`, `isNaN`, `JSON`, `Map`, `Math`, `Number`, `Object`, `parseFloat`, `parseInt`, `Promise`, `Proxy`, `queueMicrotask`, `RangeError`, `ReferenceError`, `Reflect`, `RegExp`, `Set`, `setInterval`, `setTimeout`, `SharedArrayBuffer`, `String`, `structuredClone`, `Symbol`, `SyntaxError`, `TypeError`, `Uint8Array`, `Uint8ClampedArray`, `Uint32Array`, `URIError`, `URL`, `URLPattern`, `URLSearchParams`, `WeakMap`, `WeakSet`, `WebAssembly`

### Next.js-Specific Polyfills
`AsyncLocalStorage` (Node.js API, polyfilled for Edge)

### Environment Variables
`process.env` is available for both `next dev` and `next build`.

---

## Unsupported APIs

### Hard restrictions
- **Native Node.js APIs** — no `fs`, `path`, `crypto` (Node), `child_process`, etc.
- **`require()`** — not allowed; use ESM `import` instead
- **`node_modules`** that use native Node.js APIs or CommonJS-only

### Disabled JavaScript features

| API | Why it's blocked |
|---|---|
| `eval` | Dynamic code evaluation |
| `new Function(evalString)` | Dynamic code evaluation |
| `WebAssembly.compile` | Compiles from buffer at runtime |
| `WebAssembly.instantiate` | Compiles and instantiates from buffer |

These are disabled because the Edge Runtime performs static analysis at build time to determine what code can run. Dynamic evaluation makes this impossible.

---

## Allowing Dynamic Code (escape hatch)

If a dependency contains unreachable dynamic evaluation that can't be tree-shaken, suppress the build error with `unstable_allowDynamic` in `proxy.ts`:

```ts
// proxy.ts
export const config = {
  unstable_allowDynamic: [
    '/lib/utilities.js',                      // single file
    '**/node_modules/function-bind/**',       // glob for a package
  ],
}
```

> ⚠️ If those statements actually execute at runtime, they will throw a runtime error. This only suppresses the build-time check.

---

## Setting the Runtime in Route Handlers

Route Handlers default to Node.js. To opt a Route Handler into the Edge Runtime:

```ts
// app/api/example/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  return new Response('Hello from Edge')
}
```

For `proxy.ts`, the Edge Runtime is always used — no config needed.

---

## Common Gotchas

- **No `fs` access** — can't read files at runtime; bundle static assets at build time instead
- **No `Buffer`** — use `Uint8Array` or `TextEncoder`/`TextDecoder`
- **No `process.nextTick` / `setImmediate`** — use `queueMicrotask` or `setTimeout`
- **No `crypto` (Node)** — use the Web Crypto API (`globalThis.crypto`)
- **Package compatibility** — many npm packages assume Node.js; check for edge-compatible builds (e.g., `jose` works, `jsonwebtoken` does not)
- **No ISR** — `revalidate` and `generateStaticParams` only work in the Node.js runtime