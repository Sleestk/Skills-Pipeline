# Rendering Philosophy

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/rendering-philosophy

## Component-Level Static/Dynamic Boundary

Unlike traditional frameworks where the boundary is per-route (fully static OR fully dynamic), **Next.js places the boundary at the component level**. A single page can have:
- A static shell served from CDN instantly
- Cached components that revalidate independently
- Dynamic sections that stream at request time

This is what PPR, `'use cache'`, and on-demand revalidation make possible.

## Benefits
- **Faster perceived load times** — static shell renders immediately while dynamic content streams
- **Incremental caching** — cache any function with `'use cache'`, not just entire routes
- **Granular revalidation** — invalidate a tag, not a deployment

## Trade-offs vs Other Approaches

| Model | Boundary | Pros | Cons |
|---|---|---|---|
| Build-time prerendering | Route | Simplest to deploy; any CDN | Every change requires rebuild |
| Route-level static/dynamic | Route | Simple mental model; clean infra split | All-or-nothing per route; dynamic for one element = full dynamic |
| **Component-level (Next.js)** | Component | Maximum flexibility; optimal perf | More infrastructure complexity |

## Infrastructure Implications

The component-level model requires:
- **Streaming** — static + dynamic content in one response (chunked transfer encoding / HTTP/2)
- **Cache coordination** — `revalidateTag()` must propagate across instances
- **Cache consistency** — HTML + RSC payload regenerated together; must stay in sync
- **PPR shell delivery** — optional CDN integration for edge-latency static shell

## Functional vs Performance Fidelity

- **Functional fidelity** (binary): does every feature work? → passes adapter test suite
- **Performance fidelity** (spectrum): are features at optimal perf? (CDN latency for PPR shell, sub-second ISR propagation)

A platform with functional fidelity is fully supported. Performance fidelity is how platforms differentiate.

## Portability

A single `next start` Node.js process achieves functional fidelity for everything. Streaming enables progressive delivery; without it, responses are buffered but correct. Additional infra (CDN, edge compute, shared cache) improves performance fidelity incrementally.

See [Deploying to Platforms](/docs/app/guides/deploying-to-platforms) for the full feature matrix.