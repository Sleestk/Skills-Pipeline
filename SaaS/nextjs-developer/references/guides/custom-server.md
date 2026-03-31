# Custom Server

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/custom-server

> **Warning:** Custom servers remove Automatic Static Optimization. Use only if the built-in router can't meet your needs. Not compatible with `output: 'standalone'`.

```ts
// server.ts
import { createServer } from 'http'
import next from 'next'

const port = parseInt(process.env.PORT || '3000', 10)
const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare().then(() => {
  createServer((req, res) => {
    handle(req, res)
  }).listen(port)
  console.log(`> Server listening at http://localhost:${port}`)
})
```

```json
// package.json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

> `server.js` does NOT go through the Next.js Compiler. Use syntax compatible with your Node.js version.

## `next()` options
| Option | Type | Description |
|---|---|---|
| `conf` | Object | Same as `next.config.js`. Defaults to `{}` |
| `dev` | Boolean | Launch in dev mode. Defaults to `false` |
| `dir` | String | Project location. Defaults to `'.'` |
| `quiet` | Boolean | Hide server info in errors. Defaults to `false` |
| `hostname` | String | Hostname the server runs behind |
| `port` | Number | Port the server runs behind |
| `httpServer` | `node:http#Server` | HTTP Server Next.js runs behind |
| `turbopack` | Boolean | Enable Turbopack (enabled by default) |
| `webpack` | Boolean | Enable webpack |