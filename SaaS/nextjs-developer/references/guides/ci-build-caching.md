# CI Build Caching

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/ci-build-caching

Cache `.next/cache` between builds to speed up compilation. Without it you may see a "No Cache Detected" error.

## Vercel
Automatically configured. No action needed.

## CircleCI
```yaml
steps:
  - save_cache:
      key: dependency-cache-{{ checksum "yarn.lock" }}
      paths:
        - ./node_modules
        - ./.next/cache
```

## Travis CI
```yaml
cache:
  directories:
    - $HOME/.cache/yarn
    - node_modules
    - .next/cache
```

## GitLab CI
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .next/cache/
```

## Netlify CI
Use `@netlify/plugin-nextjs`.

## AWS CodeBuild
```yaml
cache:
  paths:
    - 'node_modules/**/*'
    - '.next/cache/**/*'
```

## GitHub Actions
```yaml
uses: actions/cache@v4
with:
  path: |
    ~/.npm
    ${{ github.workspace }}/.next/cache
  key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
  restore-keys: |
    ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
```

## Bitbucket Pipelines
```yaml
definitions:
  caches:
    nextcache: .next/cache
# In pipeline step:
# caches: [node, nextcache]
```

## Heroku
```json
"cacheDirectories": [".next/cache"]
```

## Azure Pipelines
```yaml
- task: Cache@2
  inputs:
    key: next | $(Agent.OS) | yarn.lock
    path: '$(System.DefaultWorkingDirectory)/.next/cache'
```

## Jenkins (Pipeline)
```groovy
cache(caches: [
  arbitraryFileCache(path: ".next/cache", includes: "**/*", cacheValidityDecidingFile: "next-lock.cache")
]) { sh "npm run build" }
```