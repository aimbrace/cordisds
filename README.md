# cordisds

Standalone [Cordis](https://www.npmjs.com/package/@deepseek-ai/cordis) core,
extracted from the vendored fork inside
[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)
(`vendor/cordis`) and published as an independent NPM package so projects can
depend on the DeepSeek-era Cordis core without checking out the full harness
monorepo.

Cordis is a TypeScript plugin framework for applications that need explicit
dependency injection, scoped services, lifecycle-managed cleanup, and optional
configuration-driven loading.

- Upstream package: `@deepseek-ai/cordis` `4.0.4`
- Harness pin: `639ed015397290b3745d163aafe02ffee4aa3f84`
- Original author: Shigma (MIT, see `LICENSE`)

## Install

```sh
npm install cordisds
```

Requires Node 22 or newer. ESM-first (`"type": "module"`).

The only runtime dependency is `@deepseek-ai/cosmokit` (installed from NPM).
`@deepseek-ai/cordis-plugin-loader` and `@deepseek-ai/cordis-plugin-include`
remain optional peer dependencies for the `bin.js` scaffolder entry.

## Quick Start

```ts
import { Context, Service } from 'cordisds'

declare module 'cordisds' {
  interface Context {
    counter: Counter
  }

  interface Events {
    'app/ready'(message: string): void
  }
}

class Counter extends Service {
  value = 0

  constructor(ctx: Context) {
    super(ctx, 'counter')
  }

  next() {
    return ++this.value
  }
}

const greeter = Object.assign((ctx: Context) => {
  ctx.on('app/ready', (message) => {
    ctx.logger.info('%s #%d', message, ctx.counter.next())
  })
}, {
  inject: ['counter'],
})

const root = new Context()
await root.plugin(Counter)
await root.plugin(greeter)

root.emit('app/ready', 'started')
await root.fiber.dispose()
```

## Develop

```sh
npm install
npm run typecheck
npm run build
```

`src/` is the verbatim upstream core (9 modules); `tsc` emits ESM JS plus
declarations into `lib/`. The package also ships `src/` so consumers can use
`cordisds/src/*` subpath imports.

## Differences from `@deepseek-ai/cordis`

- Package renamed to `cordisds`; `bin.js` imports `cordisds` instead of
  `@deepseek-ai/cordis`.
- `workspace:~` links replaced with published NPM ranges
  (`@deepseek-ai/cosmokit ^1.8.5`, optional loader/include peers).
- Monorepo `tsconfig.base.json` inheritance and `../cosmokit` project
  reference replaced with a self-contained `tsconfig.json`.
- Docs pointing at harness-relative paths removed; behavior is unchanged.
