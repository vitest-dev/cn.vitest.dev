---
title: 主要功能 | 指南
outline: deep
---

# 主要功能

<FeaturesList class="!gap-1 text-lg" />

## 一套配置可以运用在多种环境

<div h-2 />
<CourseLink href="https://vueschool.io/lessons/your-first-test?friend=vueuse">通过视频了解如何编写你的第一个测试</CourseLink>

与 Vite 的配置、转换器、解析器和插件通用，将会使用应用中的相同配置来运行测试。

了解更多信息 [配置 Vitest](/guide/#配置-vitest)

## 监听模式(watch mode)

```bash
$ vitest
```

当你修改源代码或测试文件时，Vitest 智能搜索模块依赖树并只重新运行相关测试，就像 HMR 在 Vite 中的工作方式一样!

`vitest` **在开发环境下默认** 启动时使用 `监听模式`，在 CI 环境（当 `process.env.CI` 出现时）中以 `运行模式(run mode)` 启动。你可以使用 `vitest watch` 或 `vitest run` 明确指定所需的模式。

使用 `--standalone` 标志启动 Vitest，使其在后台运行。它不会运行任何测试，直到测试发生变化。如果源代码发生变化，Vitest 不会运行测试，直到运行了导入源代码的测试为止

## 开箱即用的常见 Web 支持

开箱即用的 ES Module / TypeScript / JSX support / PostCSS

## 多线程

默认情况下，Vitest 通过 [Tinypool](https://github.com/tinylibs/tinypool)（[Piscina](https://github.com/piscinajs/piscina) 的轻量级分叉）使用 [`node:child_process`](https://nodejs.org/api/child_process.html)，在多个进程中运行测试文件，允许测试同时运行。如果想进一步加快测试套件的速度，可以考虑启用 `--pool=threads`，使用 [`node:worker_threads`](https://nodejs.org/api/worker_threads.html)来运行测试（注意，某些软件包可能无法使用此设置）。

要在单个线程或进程中运行测试，查看 [`poolOptions`](/config/#pooloptions) 了解更多消息。

Vitest 还隔离了每个测试文件的运行环境，因此一个文件中的运行环境改变不会影响其他文件。可以通过将 `--no-isolate` 传递给 CLI 来禁用隔离（以正确性换取运行性能）。

## 测试可过滤

Vitest 提供了许多缩小测试范围的方法，以便在开发过程中加快速度并集中精力。

了解更多信息 [测试筛选](guide/filtering)

## 同时运行多个测试

在连续的测试中使用 `.concurrent` 来并行运行它们。

```ts
import { describe, it } from 'vitest'

// The two tests marked with concurrent will be started in parallel
describe('suite', () => {
  it('serial test', async () => {
    /* ... */
  })
  it.concurrent('concurrent test 1', async ({ expect }) => {
    /* ... */
  })
  it.concurrent('concurrent test 2', async ({ expect }) => {
    /* ... */
  })
})
```

如果在测试套件中使用 `.concurrent`，则其中的每个测试用例都将并发运行。

```ts
import { describe, it } from 'vitest'

// All tests within this suite will be started in parallel
describe.concurrent('suite', () => {
  it('concurrent test 1', async ({ expect }) => {
    /* ... */
  })
  it('concurrent test 2', async ({ expect }) => {
    /* ... */
  })
  it.concurrent('concurrent test 3', async ({ expect }) => {
    /* ... */
  })
})
```

你还可以将 `.skip`、`.only` 和 `.todo` 用于并发测试套件和测试用例。

了解更多信息 [API 索引](../api/#concurrent)

::: warning
在异步并发测试中使用快照时，由于 JavaScript 的限制，你需要使用 [测试环境](/guide/test-context) 中的 `expect` 来确保检测到正确的测试。
:::

## 快照

兼容 [Jest 快照测试](https://jestjs.io/zh-Hans/docs/snapshot-testing) 功能。

```ts
import { expect, it } from 'vitest'
it('renders correctly', () => {
  const result = render()
  expect(result).toMatchSnapshot()
})
```

了解更多信息 [快照](/guide/snapshot)

## Chai 和 Jest 的 `expect` 语法兼容

内置 [Chai](https://www.chaijs.com/) 进行断言和与 [Jest expect](https://jestjs.io/docs/expect) 兼容的 APIs

注意，如果你正在使用添加匹配器的第三方库，将 [`test.globals`](/config/#globals) 设置为 `true` 将提供更好的兼容性。

## 对象模拟(Mocking)

内置 [Tinyspy](https://github.com/tinylibs/tinyspy) 用于在 `vi` 对象上使用 `jest` 兼容的 API 进行对象模拟。

```ts
import { expect, vi } from 'vitest'
const fn = vi.fn()
fn('hello', 1)
expect(vi.isMockFunction(fn)).toBe(true)
expect(fn.mock.calls[0]).toEqual(['hello', 1])

fn.mockImplementation((arg: string) => arg)

fn('world', 2)
expect(fn.mock.results[1].value).toBe('world')
```

Vitest 支持 [happy-dom](https://github.com/capricorn86/happy-dom) 或 [jsdom](https://github.com/jsdom/jsdom) 来模拟 DOM 和浏览器 API。Vitest 并不内置它们，所以你可能需要安装：

::: code-group
```bash [happy-dom]
$ npm i -D happy-dom
```
```bash [jsdom]
$ npm i -D jsdom
```
:::

然后，更改 `environment` 配置文件中的选项：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
export default defineConfig({
  test: {
    environment: 'happy-dom', // or 'jsdom', 'node'
  },
})
```

了解更多信息 [模拟对象](/guide/mocking)

## 测试覆盖率

Vitest 通过 [`v8`](https://v8.dev/blog/javascript-code-coverage) 支持原生代码覆盖率，通过 [`istanbul`](https://istanbul.js.org/) 支持检测代码覆盖率。

```json [package.json]
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

可以在配置文件中设置 `test.coverage` 选项来配置它：

```ts
// vite.config.ts
import { defineConfig } from 'vitest/config'
export default defineConfig({
  test: {
    coverage: {
      reporter: ['text', 'json', 'html'],
    },
  },
})
```

了解更多信息 [测试覆盖率](/guide/coverage)

## 源码内联测试

Vitest 还提供了一种方式，可以运行与你的代码实现放在一起的测试，类似 [Rust's 模块测试](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest).

这使得测试与实现共享相同的闭包，并且能够在不导出的情况下针对私有状态进行测试。同时，它也使开发更加接近反馈循环。

<<<<<<< HEAD
```ts
// src/index.ts
=======
```ts [src/index.ts]
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f
// the implementation
export function add(...args: number[]): number {
  return args.reduce((a, b) => a + b, 0)
}
// in-source test suites
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest
  it('add', () => {
    expect(add()).toBe(0)
    expect(add(1)).toBe(1)
    expect(add(1, 2, 3)).toBe(6)
  })
}
```

了解更多信息 [源码内联测试](/guide/in-source)

## 基准测试 <Badge type="warning">实验性</Badge> {#benchmarking}

你可以使用 [`bench`](/api/#bench) 运行基准测试通过 [Tinybench](https://github.com/tinylibs/tinybench) 函数来比较基准测试结果。

```ts [sort.bench.ts]
import { bench, describe } from 'vitest'

describe('sort', () => {
  bench('normal', () => {
    const x = [1, 5, 4, 2, 3]
    x.sort((a, b) => {
      return a - b
    })
  })

  bench('reverse', () => {
    const x = [1, 5, 4, 2, 3]
    x.reverse().sort((a, b) => {
      return a - b
    })
  })
})
```

<img alt="Benchmark report" img-dark src="https://github.com/vitest-dev/vitest/assets/4232207/6f0383ea-38ba-4f14-8a05-ab243afea01d">
<img alt="Benchmark report" img-light src="https://github.com/vitest-dev/vitest/assets/4232207/efbcb427-ecf1-4882-88de-210cd73415f6">

## 类型测试 <Badge type="warning">实验性</Badge> {#type-testing}

你可以 [编写测试](/guide/testing-types) 来捕获类型回归。 Vitest 附带 [`expect-type`](https://github.com/mmkal/expect-type) 包，为你提供类似且易于理解的 API。

```ts [types.test-d.ts]
import { assertType, expectTypeOf, test } from 'vitest'
import { mount } from './mount.js'

test('my types work properly', () => {
  expectTypeOf(mount).toBeFunction()
  expectTypeOf(mount).parameter(0).toMatchTypeOf<{ name: string }>()

  // @ts-expect-error name is a string
  assertType(mount({ name: 42 }))
})
```

## 分片

使用 [`--shard`](/guide/cli#shard) 和 [`--reporter=blob`](/guide/reporters#blob-reporter)标志在不同的计算机上运行测试。可以使用 `--merge-reports` 命令在 CI 管道的末尾合并所有测试结果：

```bash
vitest --shard=1/2 --reporter=blob
vitest --shard=2/2 --reporter=blob
vitest --merge-reports --reporter=junit --coverage.reporter=text
```

了解更多信息 [`性能优化 | 分片`](/guide/improving-performance#sharding)


## 环境变量

Vitest 只从 `.env` 文件中自动加载以 `VITE_` 为前缀的环境变量，以保持与前端相关测试的兼容性，并遵守 [Vite 的既定惯例](https://vitejs.dev/guide/env-and-mode.html#env-files)。要从 `.env` 文件加载所有环境变量，可以使用从 `vite` 导入的 `loadEnv` 方法：

```ts [vitest.config.ts]
import { loadEnv } from 'vite'
import { defineConfig } from 'vitest/config'

export default defineConfig(({ mode }) => ({
  test: {
    // mode defines what ".env.{mode}" file to choose if exists
    env: loadEnv(mode, process.cwd(), ''),
  },
}))
```
