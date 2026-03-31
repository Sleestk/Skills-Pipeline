# PPR Platform Guide

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/ppr-platform-guide

## How PPR Works

**Build time**: For each PPR route, Next.js produces:
- Static HTML shell (prerendered content + Suspense fallbacks)
- `postponedState` blob (opaque — never parse or modify)
- RSC payload for static portions

**Request time**: Server sends static shell immediately, resumes dynamic portions using `postponedState`, streams dynamic content.

**Shell + postponedState must always be updated atomically** — serving a new shell with old postponedState (or vice versa) produces incorrect output.

## Implementation Options

### 1. Origin-only (simplest — works everywhere that supports streaming)
All requests go to Next.js server. Server reads shell from local cache, sends it, then renders and streams dynamic content. This is what `next start` does. No extra infra needed.

### 2. CDN shell + origin compute (better TTFB)
1. CDN serves cached shell immediately (edge latency)
2. CDN sends resume request to origin in parallel
3. Origin renders only dynamic portions using postponedState
4. CDN concatenates and streams to client

## The Resume Protocol

Tells Next.js to skip the shell and render only dynamic portions.

### CDN-to-origin (HTTP)
Send a **POST** request to the route with:
- Header: `next-resume: 1`
- Body: the `postponedState` blob

For POST requests combining Server Action + PPR resume: the body contains `postponedState` followed by action body, and `x-next-resume-state-length` header carries the byte length of the postponedState prefix.

### Adapter-based (in-process)
Call the handler with `req.method = 'POST'`, `next-resume: 1` header, and `postponedState` as body. Or pass `requestMeta: { postponed: postponedState }` as third argument.

## Finding PPR Routes in Build Output

In adapter output, PPR routes have `renderingMode: 'PARTIALLY_STATIC'` in `outputs.prerenders`. Read `fallback.postponedState` from these entries. `pprChain.headers` contains `{ 'next-resume': '1' }`.

## Implementation Checklist

1. In `onBuildComplete`, find prerenders with `renderingMode: 'PARTIALLY_STATIC'`, store shell HTML + `postponedState`
2. At request time, serve cached shell immediately
3. Resume: POST to handler with `next-resume: 1` + postponedState body (or call handler directly)
4. Use `requestMeta.onCacheEntryV2` to capture new shell+postponedState after revalidation — update atomically
5. Fallback: if postponedState unavailable, do full server render

See [adapterPath API reference](https://nextjs.org/docs/app/api-reference/config/next-config-js/adapterPath#implementing-ppr-in-an-adapter) for code examples.