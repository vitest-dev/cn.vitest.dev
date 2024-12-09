---
outline: deep
---

# 配置索引

如果我们正在使用 Vite 并且拥有一个 `vite.config` 文件，Vitest 会读取它来匹配我们的 Vite 应用的插件和设置。如果我们想要为测试配置不同的设置，或者我们的并不特别依赖于 Vite，您我们可以选择：

- 创建 `vitest.config.ts`，它将具有更高的优先级，并且会**覆盖** `vite.config.ts` 中的配置（Vitest 支持所有传统的 JS 和 TS 文件扩展名，但不支持 `json`） - 这意味着我们在 `vite.config` 中的所有选项将被**忽略**。
- 向 CLI 传递 `--config` 选项，例如 `vitest --config ./path/to/vitest.config.ts`。
- 使用 `process.env.VITEST` 或在 `defineConfig` 上的 `mode` 属性（如果没有用 `--mode` 覆盖，默认设置为 `test`/`benchmark`）来在 `vite.config.ts` 中有条件地应用不同的配置。

要配置 Vitest 本身，请在我们的 Vite 配置中添加 `test` 属性。如果您是从 `vite` 本身导入 `defineConfig`，我们还需要在配置文件顶部使用[三斜杠指令](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html#-reference-types-)添加对 Vitest 类型引用。

::: details Open Config Examples
使用 `vite` 中的 `defineConfig` 时使用以下步骤：

```ts [vite.config.js]
/// <reference types="vitest" />
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    // ... Specify options here.
  },
})
```

`<reference types="vitest" />` 将在 Vitest 4 中停止工作，但我们已经可以开始迁移到 `vitest/config`：

```ts [vite.config.js]
/// <reference types="vitest/config" />
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    // ... Specify options here.
  },
})
```

使用 `vitest/config` 中的 `defineConfig` 时应遵循以下步骤：

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // ... Specify options here.
  },
})
```

我们可以检索 Vitest 的默认选项，以便在需要时扩展它们：

```ts [vitest.config.js]
import { configDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    exclude: [...configDefaults.exclude, 'packages/template/*'],
  },
})
```

当使用单独的 `vitest.config.js` 时，我们还可以根据需要从另一个配置文件扩展 Vite 的选项：

```ts [vitest.config.js]
import { defineConfig, mergeConfig } from 'vitest/config'
import viteConfig from './vite.config'

export default mergeConfig(viteConfig, defineConfig({
  test: {
    exclude: ['packages/template/*'],
  },
}))
```

如果我们的 Vite 配置定义为一个函数，我们可以像这样定义配置：

```ts [vitest.config.js]
import { defineConfig, mergeConfig } from 'vitest/config'
import viteConfig from './vite.config'

export default defineConfig(configEnv => mergeConfig(
  viteConfig(configEnv),
  defineConfig({
    test: {
      exclude: ['packages/template/*'],
    },
  })
))
```
:::

::: warning
本页上列出的 _所有选项_ 都位于配置内的 `test` 属性内：

```ts [vitest.config.js]
export default defineConfig({
  test: {
    exclude: [],
  },
})
```

由于 Vitest 使用 Vite 的配置，我们也可以使用 [Vite](https://vitejs.dev/config/) 中的任何配置选项。例如，使用 `define` 来定义全局变量，或者使用 `resolve.alias` 来定义别名——这些选项应该在顶级定义，而不是在 `test` 属性内部。

不支持在[工作区](/guide/workspace)项目配置中的配置选项旁边会有 <NonProjectOption /> 标志。
:::

### include

- **类型:** `string[]`
- **默认值:** `['**/*.{test,spec}.?(c|m)[jt]s?(x)']`

匹配包含测试文件的 glob 规则。

::: tip NOTE
使用 coverage 时，Vitest 会自动将测试文件的 `include` 模式添加到 coverage 的默认 `exclude` 模式中。请参见 [`coverage.exclude`](#coverage-exclude)。
:::

### exclude

- **类型:** `string[]`
- **默认值:** `['**/node_modules/**', '**/dist/**', '**/cypress/**', '**/.{idea,git,cache,output,temp}/**', '**/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build}.config.*']`

匹配排除测试文件的 glob 规则。

### includeSource

- **类型:** `string[]`
- **默认值:** `[]`

包括源代码中的测试文件的通配符。

当定义时，Vitest 将运行所有包含 `import.meta.vitest` 的匹配文件。

### server {#server}

- **类型:** `{ sourcemap?, deps?, ... }`
- **版本:** Since Vitest 0.34.0

Vite-Node 服务端选项。

#### server.sourcemap

- **类型:** `'inline' | boolean`
- **默认值:** `'inline'`

通过内联方式注入到模块。

#### server.debug

- **类型:** `{ dumpModules?, loadDumppedModules? }`

Vite-Node 调试器选项。

#### server.debug.dumpModules

- **类型:** `boolean | string`

将转换后的模块转储到文件系统。传递字符串将转储到指定路径。

#### server.debug.loadDumppedModules

- **类型:** `boolean`

不管是否存在，就从文件系统中读取转储的模块。通过修改文件系统的转储结果对于调试会有帮助。

#### server.deps

- **类型:** `{ external?, inline?, ... }`

对依赖关系进行内联或外联的处理

#### server.deps.external

- **类型:** `(string | RegExp)[]`
- **默认值:** `[/\/node_modules\//]`

外部化（Externalize）意味着 Vite 将绕过原生 Node 的包。外部化依赖不会应用于 Vite 的转换器和解析器，因此它们不支持重新加载时的 HMR。 `node_modules` 下的所有包都被外部化。

这些选项支持在 `node_modules` 中编写的包名称或在 [`deps.moduleDirectories`](#deps-moduledirectories) 中指定的包名称。例如，位于 `packages/some-name` 内的包`@company/some-name` 应指定为 `some-name`，并且 `packages` 应包含在 `deps.moduleDirectories` 中。基本上，Vitest 总是检查文件路径，而不是实际的包名称。

如果成功匹配，Vitest 会在 _file path_ 上调用它，而不是包名称。

#### server.deps.inline

- **类型:** `(string | RegExp)[] | true`
- **默认值:** `[]`

Vite 将处理内联模块。这可能有助于处理以 ESM 格式传送 `.js` 的包（Node 无法处理）。

如果设置为 `true`，则每个依赖项都将被内联。默认情况下，将内联 [`ssr.noExternal`](https://cn.vitejs.dev/guide/ssr.html#ssr-externals) 中指定的所有依赖项。

#### server.deps.fallbackCJS

- **类型** `boolean`
- **默认值:** `false`

当依赖项是有效的 ESM 包时，尝试根据路径猜测 cjs 版本。如果依赖项是有错误的 ESM 文件，这可能会有所帮助。

如果包在 ESM 和 CJS 模式下具有不同的逻辑，这可能会导致一些错位。

#### server.deps.cacheDir

- **类型** `string`
- **默认值**: `'node_modules/.vite'`

保存缓存文件的目录。

### deps

- **类型:** `{ optimizer?, ... }`

处理依赖关系解析。

#### deps.optimizer {#deps-optimizer}

- **类型:** `{ ssr?, web? }`
- **参考:** [依赖优化选项](https://cn.vitejs.dev/config/dep-optimization-options.html)

启用依赖优化。如果你有很多测试，这可能会提高它们的性能。

当 Vitest 遇到 `include` 中列出的外部库时，它将使用 esbuild 打包到单个文件中，并作为整个模块导入。这很好，原因如下：

- 导入大量导入的包很昂贵。通过将它们捆绑到一个文件中，我们可以节省大量时间
- 导入 UI 库很昂贵，因为它们并不意味着在 Node.js 中运行
- 你的 `alias` 配置现在在捆绑包中得到处理
- 测试中的代码更接近于它在浏览器中的运行方式

请注意，只有 `deps.experimentalOptimizer?.[mode].include` 选项中的包会被捆绑（一些插件会自动填充它，比如 Svelte）。 你可以在 [Vite](/config/dep-optimization-options) 文档中阅读有关可用选项的更多信息。默认情况，Vitest 的 `experimentalOptimizer.web` 用在 `jsdom` 和 `happy-dom`, 在 `node` 和 `edge` 环境下使用 `experimentalOptimizer.ssr`，但这可以在 [`transformMode`](#transformmode) 进行配置。

此选项还继承了你的 `optimizeDeps` 配置（对于 web 环境， Vitest 将会继承 `optimizeDeps`，对于 ssr 则是 `ssr.optimizeDeps`）。如果你在 `deps.experimentalOptimizer` 中重新定义 `include`/`exclude`/`entries` 选项，它将在运行测试时覆盖你的 `optimizeDeps`。如果它们在 `exclude` 中配置，Vitest 会自动从 `include` 中删除相同的选项。

::: tip
你将无法编辑用于调试的 `node_modules` 代码，因为该代码实际上位于你的 `cacheDir` 或 `test.cache.dir` 目录中。如果你想使用 `console.log` 语句进行调试，请直接编辑它或使用 `deps.experimentalOptimizer?.[mode].force` 选项强制重新绑定。
:::

#### deps.optimizer.{mode}.enabled

- **类型:** `boolean`
- **默认值:** `false`

启用依赖优化。

::: warning
此选项仅适用于 Vite 4.3.2 及更高版本。
:::

#### deps.web <Badge type="info">0.34.2+</Badge>

#### deps.web

- **类型:** `{ transformAssets?, ... }`

当转换模式设置为 `web` 时应用于外部文件的选项。默认情况下，`jsdom` 和 `happy-dom` 使用 `web` 模式，而 `node` 和 `edge` 环境使用 `ssr` 转换模式，因此这些选项不会影响这些环境中的文件。

通常，`node_modules` 内的文件是外部化的，但这些选项也会影响 [`server.deps.external`](#server-deps-external) 中的文件。

#### deps.web.transformAssets

- **类型:** `boolean`
- **默认值:** `true`

Vitest 是否应该像 Vite 在浏览器中一样处理静态资源（.png、.svg、.jpg 等）文件并解析它们。

如果未指定查询，此模块将具有等同于静态资源路径的默认导出。

::: warning
目前，此选项适用于 [`vmThreads`](#vmthreads) 和 [`vmForks`](#vmForks) 池。
:::

#### deps.web.transformCss

- **类型:** `boolean`
- **默认值:** `true`

Vitest 是否应该像 Vite 在浏览器中一样处理静态资源（.css, .scss, .sass 等）文件并解析它们。

如果使用 [`css`](#css) 选项禁用 CSS 文件，则此选项只会消除 `ERR_UNKNOWN_FILE_EXTENSION` 错误。

::: warning
目前，此选项仅适用于 [`vmThreads`](#vmthreads) 和 [`vmForks`](#vmForks) 池。
:::

#### deps.web.transformGlobPattern

- **类型:** `RegExp | RegExp[]`
- **默认值:** `[]`

正则表达式模式匹配应转换的外部文件。

默认情况下，`node_modules` 内的文件是外部化的，不会被转换，除非它是 CSS 或静态资源，并且相应的选项不会被禁用。

::: warning
目前，此选项仅适用于 [`vmThreads`](#vmthreads) 和 [`vmForks`](#vmForks) 池。
:::

#### deps.interopDefault

- **类型:** `boolean`
- **默认值:** `true`

将 CJS 模块的默认值视为命名导出。某些依赖项仅捆绑 CJS 模块，不使用命名导出，Node.js 可以在使用 `import` 语法而不是 `require` 导入包时对其进行静态分析。使用命名导出在 Node 环境中导入此类依赖项时，你将看到此错误：

```
import { read } from 'fs-jetpack';
         ^^^^
SyntaxError: Named export 'read' not found. The requested module 'fs-jetpack' is a CommonJS module, which may not support all module.exports as named exports.
CommonJS modules can always be imported via the default export.
```

Vitest 不进行静态分析，并且不会在你运行代码之前失败，因此当该特性禁用时你在运行测试时很可能会看到此错误：

```
TypeError: createAsyncThunk is not a function
TypeError: default is not a function
```

如果你使用的是绕过此 Node.js 限制的捆绑器或转译器，则可以手动启用此选项。默认情况下，当 `environment` 为 `node` 时，Vitest 假定你使用的是 Node ESM 语法，并且不关心命名导出。

#### deps.moduleDirectories

- **类型:** `string[]`
- **默认值**: `['node_modules']`

配置一个视为模块目录的目录列表。此配置选项会影响 [`vi.mock`](/api/vi#vi-mock) 的行为：当未提供工厂并且你正在模拟的路径与 `moduleDirectories` 值之一匹配时，Vitest 将尝试 通过在项目的 [root](/config/#root) 中查找 `__mocks__` 文件夹来解析 mock。

此选项还将影响在外部化依赖项时是否应将文件视为模块。默认情况下，Vitest 绕过 Vite 转换步骤导入带有原生 Node.js 的外部模块。

设置此选项将 _覆盖_ 默认值，如果你仍希望搜索 `node_modules` 包包括它连同任何其他选项：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    deps: {
      moduleDirectories: ['node_modules', path.resolve('../../packages')],
    },
  },
})
```

### runner

- **类型**: `VitestRunnerConstructor`
- **默认值**: `node`, when running tests, or `benchmark`, when running benchmarks

自定义测试运行程序的路径。这是一项高级功能，应与自定义库运行器一起使用。你可以在 [文档](/advanced/runner) 中阅读更多相关信息。

### benchmark

- **类型:** `{ include?, exclude?, ... }`

运行 `vitest bench` 时使用的选项。

#### benchmark.include

- **类型:** `string[]`
- **默认值:** `['**/*.{bench,benchmark}.?(c|m)[jt]s?(x)']`

匹配包含基准测试文件的 glob 规则。

#### benchmark.exclude

- **类型:** `string[]`
- **默认值:** `['node_modules', 'dist', '.idea', '.git', '.cache']`

匹配排除基准测试文件的 glob 规则。

#### benchmark.includeSource

- **类型:** `string[]`
- **默认值:** `[]`

匹配包含内联基准测试文件的 glob 规则。此选项类似于 [`includeSource`](#includesource)。

定义后，Vitest 将运行所有匹配的文件，其中包含 `import.meta.vitest`。

#### benchmark.reporters

- **类型:** `Arrayable<BenchmarkBuiltinReporters | Reporter>`
- **默认值:** `'default'`

用于定义输出的自定义报告器。它可以包含一个或多个内置报告名称、报告实例和(或)自定义报告的路径。

#### benchmark.outputFile

已弃用，尝试使用 `benchmark.outputJson`。

#### benchmark.outputJson {#benchmark-outputJson}

- **类型:** `string | undefined`
- **默认值:** `undefined`

存储基准测试结果的文件路径，可用于稍后的 `--compare` 选项。

例如:

```sh
# save main branch's result
git checkout main
vitest bench --outputJson main.json

# change a branch and compare against main
git checkout feature
vitest bench --compare main.json
```

#### benchmark.compare {#benchmark-compare}

- **类型:** `string | undefined`
- **默认值:** `undefined`

与当前运行结果进行比较的以前基准结果的文件路径。

### alias

- **类型:** `Record<string, string> | Array<{ find: string | RegExp, replacement: string, customResolver?: ResolverFunction | ResolverObject }>`

在测试内部运行时定义自定义别名。它们将与来自 `resolve.alias` 的别名合并。

::: warning
Vitest 使用 Vite SSR 基元来运行测试，这有[一定的缺陷](https://vitejs.dev/guide/ssr.html#ssr-externals)。

1. 别名只影响由[inlined](#server-deps-inline)模块直接用`import`关键字导入的模块（默认情况下所有源代码都是内联的）。
2. Vitest 不支持对 `require` 调用进行别名。
3. 如果您要别名外部依赖（例如，`react` -> `preact`），您可能需要别名实际的 `node_modules` 包，以使其适用于外部依赖。[Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias) 和 [pnpm](https://pnpm.io/aliases/) 都支持通过 `npm:` 前缀进行别名。
   :::

### globals

- **类型:** `boolean`
- **默认值:** `false`
- **命令行终端:** `--globals`, `--globals=false`

默认情况下，`vitest` 不显式提供全局 API。如果你更倾向于使用类似 jest 中的全局 API，可以将 `--globals` 选项传递给 CLI 或在配置中添加 `globals: true`。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
  },
})
```

为了可以让全局 API 支持 TypeScript，请将 `vitest/globals` 添加到 `tsconfig.json` 中的 `types` 选项中

```json [tsconfig.json]
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

如果你已经在项目中使用 [`unplugin-auto-import`](https://github.com/antfu/unplugin-auto-import)，你也可以直接用它来自动导入这些 API。

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    AutoImport({
      imports: ['vitest'],
      dts: true, // generate TypeScript declaration
    }),
  ],
})
```

### environment

- **类型:** `'node' | 'jsdom' | 'happy-dom' | 'edge-runtime' | string`
- **默认值:** `'node'`
- **命令行终端:** `--environment=<env>`

Vitest 中的默认测试环境是一个 Node.js 环境。如果你正在构建 Web 端应用，你可以使用 [`jsdom`](https://github.com/jsdom/jsdom) 或 [`happy-dom`](https://github.com/capricorn86/happy-dom) 这种类似浏览器(browser-like)的环境来替代 Node.js。
如果你正在构建边缘计算函数，你可以使用 [`edge-runtime`](https://edge-runtime.vercel.app/packages/vm) 环境

::: tip
你还可以使用 [浏览器模式](/guide/browser/) 在浏览器中运行集成或单元测试，而无需模拟环境。
:::

你可以通过在文件顶部添加包含 `@vitest-environment` 的文档块或注释，为某个测试文件中的所有测试指定环境：

文档块格式:

```js
/**
 * @vitest-environment jsdom
 */

test('use jsdom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

注释格式:

```js
// @vitest-environment happy-dom

test('use happy-dom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

为了与 Jest 兼容，还存在一个配置 `@jest-environment`：

```js
/**
 * @jest-environment jsdom
 */

test('use jsdom in this test file', () => {
  const element = document.createElement('div')
  expect(element).not.toBeNull()
})
```

如果使用 [`--isolate=false`](#isolate) 运行 Vitest，测试将按以下顺序运行：`node`、`jsdom`、`happy-dom`、`edge-runtime`、`custom environments`。也就是说，具有相同环境的每个测试都会被分组，但仍会按顺序运行。

从 0.23.0 开始，你还可以定义自定义环境。 当使用非内置环境时，Vitest 将尝试加载包 `vitest-environment-${name}`。 该包应导出一个具有 `Environment` 属性的对象：

```ts [environment.js]
import type { Environment } from 'vitest'

export default <Environment>{
  name: 'custom',
  transformMode: 'ssr',
  setup() {
    // custom setup
    return {
      teardown() {
        // called after all tests with this env have been run
      },
    }
  },
}
```

Vitest 还通过 `vitest/environments` 入口导出 `builtinEnvironments`，以防你只想扩展它。 你可以在 [测试环境指南](/guide/environment) 中阅读有关扩展测试环境的更多信息。

::: tip
jsdom 环境变量导出了等同于当前[JSDOM](https://github.com/jsdom/jsdom) 的 `jsdom` 全局变量实例。如果你想让 TypeScript 识别它，可以在使用此环境时将 `vitest/jsdom`添加到 `tsconfig.json` 中：

```json [tsconfig.json]
{
  "compilerOptions": {
    "types": ["vitest/jsdom"]
  }
}
```

:::

### environmentOptions

- **类型:** `Record<'jsdom' | string, unknown>`
- **默认值:** `{}`

这些选项被传递给当前 [`environment`](#environment) 的 `setup` 方法。 默认情况下，如果你将其用作测试环境，则只能配置 JSDOM 选项。

### environmentMatchGlobs

- **类型:** `[string, EnvironmentName][]`
- **默认值:** `[]`

::: danger DEPRECATED
此 API 在 Vitest 3 中已弃用。请使用 [workspace](/guide/workspace) 来定义不同的配置。

```ts
export default defineConfig({
  test: {
    environmentMatchGlobs: [ // [!code --]
      ['./*.jsdom.test.ts', 'jsdom'], // [!code --]
    ], // [!code --]
    workspace: [ // [!code ++]
      { // [!code ++]
        extends: true, // [!code ++]
        test: { // [!code ++]
          environment: 'jsdom', // [!code ++]
        }, // [!code ++]
      }, // [!code ++]
    ], // [!code ++]
  },
})
```
:::

基于 globs 自动匹配执行环境。将使用第一个匹配项。

例如：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environmentMatchGlobs: [
      // all tests in tests/dom will run in jsdom
      ['tests/dom/**', 'jsdom'],
      // all tests in tests/ with .edge.test.ts will run in edge-runtime
      ['**/*.edge.test.ts', 'edge-runtime'],
      // ...
    ],
  },
})
```

### poolMatchGlobs {#poolmatchglobs}

- **类型:** `[string, 'threads' | 'forks' | 'vmThreads' | 'vmForks' | 'typescript'][]`
- **默认值:** `[]`

::: danger DEPRECATED
此 API 在 Vitest 3 中已被弃用。请使用 [workspace](/guide/workspace) 来定义不同的配置：

```ts
export default defineConfig({
  test: {
    poolMatchGlobs: [ // [!code --]
      ['./*.threads.test.ts', 'threads'], // [!code --]
    ], // [!code --]
    workspace: [ // [!code ++]
      { // [!code ++]
        test: { // [!code ++]
          extends: true, // [!code ++]
          pool: 'threads', // [!code ++]
        }, // [!code ++]
      }, // [!code ++]
    ], // [!code ++]
  },
})
```
:::

基于 globs 模式来匹配运行池中的测试并运行，将使用第一个匹配项。

例如:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    poolMatchGlobs: [
      // all tests in "worker-specific" directory will run inside a worker as if you enabled `--pool=threads` for them,
      ['**/tests/worker-specific/**', 'threads'],
      // run all tests in "browser" directory in an actual browser
      ['**/tests/browser/**', 'browser'],
      // all other tests will run based on "browser.enabled" and "threads" options, if you didn't specify other globs
      // ...
    ],
  },
})
```

### update<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `false`
- **命令行终端:** `-u`, `--update`, `--update=false`

更新快照文件。这将更新所有更改的快照并删除过时的快照。

### watch<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `!process.env.CI`
- **命令行终端:** `-w`, `--watch`, `--watch=false`

启动监听模式

### root

- **类型:** `string`
- **命令行终端:** `-r <path>`, `--root=<path>`

项目的根目录

### dir

- **类型:** `string`
- **命令行终端:** `--dir=<path>`
- **默认值:** same as `root`

扫描测试文件的基本目录。如果您的根目录覆盖整个项目，您可以指定此选项以加快测试发现速度

### reporters<NonProjectOption />

- **类型:** `Reporter | Reporter[]`
- **默认值:** `'default'`
- **命令行终端:** `--reporter=<name>`, `--reporter=<name1> --reporter=<name2>`

用于输出的自定义 reporters 。 Reporters 可以是 [一个 Reporter 实例](https://github.com/vitest-dev/vitest/blob/main/packages/vitest/src/node/types/reporter.ts) 或选择内置的 reporters 字符串：

- `'default'` - 当他们经过测试套件
- `'basic'` - 给定一个类似于 CI 中的默认报告实例
- `'verbose'` - 保持完整的任务树可见
- `'dot'` - 将每个任务显示为一个点
- `'junit'` - JUnit XML 报告器（你可以使用 `VITEST_JUNIT_SUITE_NAME` 环境变量配置 `test suites` 标签名称）
- `'json'` - 给出一个简单的 JSON 总结
- `'html'` - 根据 [`@vitest/ui`](/guide/ui) 输出 HTML 报告
- `'hanging-process'` - 如果 Vitest 无法安全退出进程，则显示挂起进程列表。 这可能是一个复杂的操作，只有在 Vitest 始终无法退出进程时才启用它
- 自定义报告的路径 (例如 `'./path/to/reporter.ts'`, `'@scope/reporter'`)

### outputFile<NonProjectOption />

- **类型:** `string | Record<string, string>`
- **命令行终端:** `--outputFile=<path>`, `--outputFile.json=./path`

当指定 `--reporter=json`、`--reporter=html` 或 `--reporter=junit` 时，将测试结果写入一个文件。通过提供对象而不是字符串，你可以在使用多个报告器时定义单独的输出。

### pool<NonProjectOption /> {#pool}

- **类型:** `'threads' | 'forks' | 'vmThreads' | 'vmForks'`
- **默认值:** `'forks'`
- **命令行终端:** `--pool=threads`

用于运行测试的线程池。

#### threads<NonProjectOption />

使用 [tinypool](https://github.com/tinylibs/tinypool)（一个轻量级的 [Piscina](https://github.com/piscinajs/piscina) 分支）来启用多线程。当使用线程时，你无法使用与进程相关的 API，如 `process.chdir()` 。一些使用原生语言编写的库，如 Prisma 、`bcrypt` 和 `canvas` ，在多线程环境下可能会遇到问题并导致段错误。在这些情况下，建议使用 `forks` 线程池。

#### forks<NonProjectOption />

与 `threads` 线程池类似，但是使用 `child_process` 而不是 `worker_threads` ，通过 [tinypool](https://github.com/tinylibs/tinypool) 实现。与 `threads` 线程池相比，测试与主进程之间的通信速度不够快。在 `forks` 线程池中，可以使用与进程相关的 API ，如 `process.chdir()` 。

#### vmThreads<NonProjectOption />

在 `threads` 线程池中使用[ VM 上下文](https://nodejs.org/api/vm.html)（在受限环境中）运行测试。

这样可以加快测试速度，但是当运行[ ESM 代码](https://github.com/nodejs/node/issues/37648)时，VM 模块可能不稳定。你的测试可能会[泄漏内存](https://github.com/nodejs/node/issues/33439)，为了解决这个问题，考虑手动编辑 [`poolOptions.vmThreads.memoryLimit`](#pooloptions-vmthreads-memorylimit) 的值。

::: warning
在沙箱中运行代码有一些优点（测试速度更快），但也有许多缺点。

- 原生模块中的全局变量，例如（`fs`、`path`等），与测试环境中存在的全局变量不同。因此，这些原生模块引发的任何错误都将引用与代码中使用的错误构造函数不同的错误构造函数：

```ts
try {
  fs.writeFileSync('/doesnt exist')
}
catch (err) {
  console.log(err instanceof Error) // false
}
```

- 导入 ES 模块会无限期地缓存它们，如果你有很多上下文（测试文件），这会导致内存泄漏。Node.js 中没有可以清除该缓存的 API。
- 在沙盒环境中访问全局变量[需要更长的时间](https://github.com/nodejs/node/issues/31658)。

使用此选项时请注意这些问题。Vitest 团队无法解决我们这边的任何问题。
:::

#### vmForks<NonProjectOption />

与 `vmThreads` 池类似，但通过 [tinypool](https://github.com/tinylibs/tinypool) 使用 `child_process` 而不使用 `worker_threads`。测试与主进程之间的通信速度虽然不如 `vmThreads` 快。但进程相关的 API（如 `process.chdir()` ）在 `vmForks` 中却可以使用。请注意，这个与 `vmThreads` 中列出的池具有相同的缺陷。

### poolOptions<NonProjectOption /> {#pooloptions}

- **类型:** `Record<'threads' | 'forks' | 'vmThreads' | 'vmForks', {}>`
- **默认值:** `{}`

#### poolOptions.threads

`threads` 池的选项。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    poolOptions: {
      threads: {
        // Threads related options here
      },
    },
  },
})
```

##### poolOptions.threads.maxThreads<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最大线程数或百分比。还可以使用`VITEST_MAX_THREADS`环境变量进行设置。

##### poolOptions.threads.minThreads<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最小线程数或百分比。还可以使用`VITEST_MIN_THREADS`环境变量进行设置。

##### poolOptions.threads.singleThread

- **类型:** `boolean`
- **默认值:** `false`

在单个工作线程内使用相同的环境运行所有测试。 这将禁用内置模块隔离（我们的源代码或 [inlined](#server-deps-inline) 代码仍将针对每个测试重新评估），但可以提高测试性能。

::: warning
尽管此选项将强制测试一个接一个地运行，但此选项与 Jest 的 `--runInBand` 不同。 Vitest 使用工作线程不仅可以并行运行测试，还可以提供隔离。 通过禁用此选项，你的测试将按顺序运行，但在相同的全局上下文中，因此你必须自己提供隔离。

如果你依赖全局状态（前端框架通常这样做）或者你的代码依赖于为每个测试单独定义的环境，这可能会导致各种问题。 但可以提高你的测试速度（最多快 3 倍），这不一定依赖于全局状态，也可以轻松绕过它。
:::

##### poolOptions.threads.useAtomics<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `false`

使用 Atomics 来同步线程。

这在某些情况下可以提高性能，但可能会导致旧 Node 版本中出现段错误。

##### poolOptions.threads.isolate

- **类型:** `boolean`
- **默认值:** `true`

隔离每个测试文件的环境。

##### poolOptions.threads.execArgv<NonProjectOption />

- **类型:** `string[]`
- **默认值:** `[]`

在线程中向 `node` 传递附加参数。更多信息，具体可以浏览 [Command-line API | Node.js](https://nodejs.org/docs/latest/api/cli.html) 。

::: warning
使用时要小心，因为某些选项（如--prof、--title）可能会导致 worker 崩溃。具体信息可以浏览 https://github.com/nodejs/node/issues/41103。
:::

#### poolOptions.forks

`forks` 池的选项。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    poolOptions: {
      forks: {
        // Forks related options here
      },
    },
  },
})
```

##### poolOptions.forks.maxForks<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最大分支数量或百分比。

##### poolOptions.forks.minForks<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最小分支数量或百分比。

##### poolOptions.forks.isolate

- **类型:** `boolean`
- **默认值:** `true`

隔离每个测试文件的环境。

##### poolOptions.forks.singleFork

- **类型:** `boolean`
- **默认值:** `false`

在单个子进程中使用相同的环境运行所有测试。 这将禁用内置模块隔离（你的源代码或 [inlined](#server-deps-inline) 代码仍将针对每个测试重新评估），但可以提高测试性能。

::: warning
尽管此选项将强制测试一个接一个地运行，但此选项与 Jest 的 `--runInBand` 不同。 Vitest 使用子进程不仅可以并行运行测试，还可以提供隔离。 通过禁用此选项，你的测试将按顺序运行，但在相同的全局上下文中，因此你必须自己提供隔离。

如果你依赖全局状态（前端框架通常这样做）或者你的代码依赖于为每个测试单独定义的环境，这可能会导致各种问题。 但可以提高你的测试速度（最多快 3 倍），这不一定依赖于全局状态，也可以轻松绕过它。
:::

##### poolOptions.forks.execArgv<NonProjectOption />

- **类型:** `string[]`
- **默认值:** `[]`

向子进程中的 `node` 进程传递附加参数。更多信息，详细信息可以浏览 [Command-line API | Node.js](https://nodejs.org/docs/latest/api/cli.html) 。

:::warning
使用时要小心，因为某些选项（如 --prof、--title ）可能会导致 worker 崩溃。详细信息可以浏览 https://github.com/nodejs/node/issues/41103。
:::

#### poolOptions.vmThreads

`vmThreads` 池的选项。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    poolOptions: {
      vmThreads: {
        // VM threads related options here
      },
    },
  },
})
```

##### poolOptions.vmThreads.maxThreads<NonProjectOption />

- **类型:** `number | string`
- **默认:** _available CPUs_

最大线程数或百分比。还可以使用`VITEST_MAX_THREADS`环境变量进行设置。

##### poolOptions.vmThreads.minThreads<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最小线程数或百分比。还可以使用`VITEST_MIN_THREADS`环境变量进行设置。

##### poolOptions.vmThreads.memoryLimit<NonProjectOption />

- **类型:** `string | number`
- **命令行终端:** `1 / CPU Cores`

指定工作线程被回收之前的内存限制。该值在很大程度上取决于你的运行环境，因此最好手动指定它，而不是依赖默认值。

::: tip
该实现基于 Jest 的 [`workerIdleMemoryLimit`](https://jestjs.io/docs/configuration#workeridlememorylimit-numberstring)。

可以通过多种不同的方式指定限制，无论结果是什么，`Math.floor` 都用于将其转换为整数值：

- `<= 1` - 该值假定为系统内存的百分比。所以 0.5 将 worker 的内存限制设置为系统总内存的一半。
- `\> 1` - 假设是固定字节值。由于之前的规则，如果你想要 1 字节的值（我不知道为什么），你可以使用 1.1。
- 有单位时
  - `50%` - 如上，占系统总内存的百分比
  - `100KB`, `65MB`, 等 - 用单位表示固定的内存限制
    - `K` / `KB` - Kilobytes (x1000)
    - `KiB` - Kibibytes (x1024)
    - `M` / `MB` - Megabytes - `MiB` - Mebibytes
    - `G` / `GB` - Gigabytes - `GiB` - Gibibytes

:::

::: warning
由于系统内存报告不正确，基于百分比的内存限制[在 Linux CircleCI 上不起作用](https://github.com/jestjs/jest/issues/11956#issuecomment-1212925677)。
:::

##### poolOptions.vmThreads.useAtomics<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `false`

使用 Atomics 来同步线程。

这在某些情况下可以提高性能，但可能会在旧的 Node 版本中抛出错误。

##### poolOptions.vmThreads.execArgv<NonProjectOption />

- **类型:** `string[]`
- **默认值:** `[]`

将附加参数传递给虚拟机上下文中的 `node` 进程。更多信息，详细信息可以浏览 [Command-line API | Node.js](https://nodejs.org/docs/latest/api/cli.html) 。

:::warning
使用时要小心，因为某些选项（如 --prof、--title ）可能会导致 worker 崩溃。详细信息可以浏览 https://github.com/nodejs/node/issues/41103。
:::

#### poolOptions.vmForks<NonProjectOption />

`vmForks` 池的选项

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    poolOptions: {
      vmForks: {
        // VM forks related options here
      },
    },
  },
})
```

##### poolOptions.vmForks.maxForks<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最大线程数或百分比。也可以使用 `VITEST_MAX_FORKS` 环境变量。

##### poolOptions.vmForks.minForks<NonProjectOption />

- **类型:** `number | string`
- **默认值:** _available CPUs_

最小线程数或百分比。也可以使用 `VITEST_MIN_FORKS` 环境变量。

##### poolOptions.vmForks.memoryLimit<NonProjectOption />

- **类型:** `string | number`
- **默认值:** `1 / CPU Cores`

指定 Worker 被回收前的内存限制。该值在很大程度上取决于环境，因此最好手动指定，而不是依赖默认值。该值的计算方法查看 [`poolOptions.vmThreads.memoryLimit`](#pooloptions-vmthreads-memorylimit)

##### poolOptions.vmForks.execArgv<NonProjectOption />

- **类型:** `string[]`
- **默认值:** `[]`

将附加参数传递给虚拟机上下文中的 `node` 进程。更多信息，查看 [Command-line API | Node.js](https://nodejs.org/docs/latest/api/cli.html) 了解更多详情。。

:::warning
使用时要小心，因为某些选项（如 `--prof` 、`--title`）可能会导致 worker 崩溃。查看 https://github.com/nodejs/node/issues/41103 了解更多详情。
:::

### fileParallelism<NonProjectOption /> {#fileparallelism}

- **类型:** `boolean`
- **默认值:** `true`
- **命令行终端:** `--no-file-parallelism`, `--fileParallelism=false`

所有测试文件应该并行运行。将其设置为 `false` 将覆盖 `maxWorkers` 和 `minWorkers` 选项为 `1`。

::: tip
此选项不会影响在同一文件中运行的测试。如果你想并行运行这些程序，请在[description](/api/#describe-concurrent)或通过[a config](#sequence-concurrent) 上使用 `concurrent` 选项。
:::

### maxWorkers<NonProjectOption /> {#maxworkers}

- **类型:** `number | string`

运行测试时设置的最大工作线程数或百分比。`poolOptions。｛threads，vmThreads｝.maxThreads `/`poolOptions.forks.maxForks` 具有更高的优先级。

### minWorkers<NonProjectOption /> {#minworkers}

- **类型:** `number | string`

运行测试时设置的最小工作线程数或百分比。`poolOptions.{threads,vmThreads}.minThreads`/`poolOptions.forks.minForks` 具有更高的优先级。

### testTimeout

- **类型:** `number`
- **默认值:** `5_000` in Node.js, `15_000` if `browser.enabled` is `true`
- **命令行终端:** `--test-timeout=5000`, `--testTimeout=5000`

测试的默认超时时间（以毫秒为单位）。

### hookTimeout

- **类型:** `number`
- **默认值:** `10_000` in Node.js, `30_000` if `browser.enabled` is `true`
- **命令行终端:** `--hook-timeout=10000`, `--hookTimeout=10000`

钩子(hook)的默认超时时间（以毫秒为单位）。

### teardownTimeout<NonProjectOption />

- **类型:** `number`
- **默认值:** `1000`
- **命令行终端:** `--teardown-timeout=5000`, `--teardownTimeout=5000`

Vitest 关闭时等待关闭的默认超时时间，以毫秒为单位

### silent<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `false`
- **命令行终端:** `--silent`, `--silent=false`

静默模式下启动测试。

### setupFiles

- **类型:** `string | string[]`

setup 文件的路径。它们将运行在每个测试文件之前。

::: info 提示
编辑设置文件将自动触发所有测试的重新运行。
:::

你可以在全局设置文件中使用 `process.env.VITEST_POOL_ID`（类似整数的字符串）来区分不同的线程。

:::tip
请注意，如果运行 [`--isolate=false`](#isolate) ，这个配置文件将在全局范围内多次运行。这意味着每次测试前都要访问同一个全局对象，因此请确保不要重复做同一件事。
:::

比如，你可能依赖于一个全局变量：

```ts
import { config } from '@some-testing-lib'

if (!globalThis.defined) {
  config.plugins = [myCoolPlugin]
  computeHeavyThing()
  globalThis.defined = true
}

// hooks are reset before each suite
afterEach(() => {
  cleanup()
})

globalThis.resetBeforeEachTest = true
```

### provide <Version>2.1.0</Version> {#provide}

- **Type:** `Partial<ProvidedContext>`

使用 `inject` 方法定义可在测试中访问的值。

:::code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    provide: {
      API_KEY: '123',
    },
  },
})
```
```ts [api.test.js]
import { expect, inject, test } from 'vitest'

test('api key is defined', () => {
  expect(inject('API_KEY')).toBe('123')
})
```
:::

::: warning
属性必须是字符串，值必须是[可序列化](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)，因为该对象将在不同进程之间传输。
:::

::: tip
如果使用的是 TypeScript，则需要增强 `ProvidedContext` 类型，以实现类型安全访问：

```ts [vitest.shims.d.ts]
declare module 'vitest' {
  export interface ProvidedContext {
    API_KEY: string
  }
}

// mark this file as a module so augmentation works correctly
export {}
```
:::

### globalSetup

- **类型:** `string | string[]`

全局的 setup 文件的路径，相对于项目的根目录。

全局的 setup 文件可以导出命名函数 `setup` 和 `teardown` 或返回拆卸函数的 `default` 函数（[示例](https://github.com/vitest-dev/vitest/blob/main/test/global-setup/vitest.config.ts))。

::: info 提示
可以存在多个 globalSetup。setup 和 teardown 依次执行，而 teardown 则以相反的顺序执行。
:::

::: warning
全局设置只有在至少有一个正在运行的测试时才运行。这意味着在测试文件更改后，全局安装程序可能会在监视模式下开始运行（测试文件将等待全局安装程序完成后再运行）。

请注意，全局设置在不同的全局范围内运行，因此你的测试无法访问此处定义的变量。悬停，从 1.0.0 开始，你可以通过 [`provide`](#provide) 方法将可序列化数据传递给测试：

:::code-group
```ts [example.test.js]
import { inject } from 'vitest'

inject('wsPort') === 3000
```
```ts [globalSetup.ts <Version>3.0.0</Version>]
import type { TestProject } from 'vitest/node'

export default function setup(project: TestProject) {
  project.provide('wsPort', 3000)
}

declare module 'vitest' {
  export interface ProvidedContext {
    wsPort: number
  }
}
```
```ts [globalSetup.ts <Version>2.0.0</Version>]
import type { GlobalSetupContext } from 'vitest/node'

export default function setup({ provide }: GlobalSetupContext) {
  provide('wsPort', 3000)
}

declare module 'vitest' {
  export interface ProvidedContext {
    wsPort: number
  }
}
```
:::
自 Vitest  3起，我们可以定义一个自定义回调函数，在 Vitest 重新运行测试时被调用。如果该函数是异步的，运行器将在执行测试前等待其完成。请注意，您不能像 `{ onTestsRerun }` 那样解构 `project` ，因为它依赖于上下文环境。

```ts [globalSetup.ts]
import type { TestProject } from 'vitest/node'

export default function setup(project: TestProject) {
  project.onTestsRerun(async () => {
    await restartDb()
  })
}
```

### forceRerunTriggers<NonProjectOption />

- **类型**: `string[]`
- **默认值:** `['**/package.json/**', '**/vitest.config.*/**', '**/vite.config.*/**']`

将触发整个套件重新运行的文件路径的全局 glob 模式。 如果在 git diff 中找到触发器，则与 --changed 参数配对时，将运行整个测试套件。

如果你正在测试调用 CLI 命令时很有用，因为 Vite 无法构建模块依赖树:

```ts
test('execute a script', async () => {
  // Vitest cannot rerun this test, if content of `dist/index.js` changes
  await execa('node', ['dist/index.js'])
})
```

:::tip 提醒
请确保您的文件没有被 [`server.watch.ignored`](https://vitejs.dev/config/server-options.html#server-watch)排除在外。
:::

### coverage<NonProjectOption />

- **类型:** `CoverageC8Options | CoverageIstanbulOptions`
- **默认值:** `undefined`

你可以使用点符号向 CLI 提供覆盖选项：

```sh
npx vitest --coverage.enabled --coverage.provider=istanbul --coverage.all
```

::: warning
如果你使用带点符号的覆盖选项，请不要忘记指定 `--coverage.enabled`。 在这种情况下，不要提供单个 --coverage 选项。
:::

#### coverage.provider

- **类型:** `'v8' | 'istanbul' | 'custom'`
- **默认值:** `'v8'`
- **命令行终端:** `--coverage.provider=<provider>`

使用 `provider` 选择收集测试覆盖率的工具。

#### coverage.enabled

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.enabled`, `--coverage.enabled=false`

是否启用收集测试覆盖率。可以使用 `--coverage` 覆盖 CLI 选项。

#### coverage.include

- **类型:** `string[]`
- **默认值:** `['**']`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.include=<path>`, `--coverage.include=<path1> --coverage.include=<path2>`

匹配包含测试覆盖率的 glob 规则

#### coverage.extension

- **类型:** `string | string[]`
- **默认值:** `['.js', '.cjs', '.mjs', '.ts', '.mts', '.tsx', '.jsx', '.vue', '.svelte', '.marko', '.astro']`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.extension=<extension>`, `--coverage.extension=<extension1> --coverage.extension=<extension2>`

#### coverage.exclude

- **类型:** `string[]`
- **默认值:**

```js
[
  'coverage/**',
  'dist/**',
  '**/node_modules/**',
  '**/[.]**',
  'packages/*/test?(s)/**',
  '**/*.d.ts',
  '**/virtual:*',
  '**/__x00__*',
  '**/\x00*',
  'cypress/**',
  'test?(s)/**',
  'test?(-*).?(c|m)[jt]s?(x)',
  '**/*{.,-}{test,spec,bench,benchmark}?(-d).?(c|m)[jt]s?(x)',
  '**/__tests__/**',
  '**/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*',
  '**/vitest.{workspace,projects}.[jt]s?(on)',
  '**/.{eslint,mocha,prettier}rc.{?(c|m)js,yml}',
]
```

- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.exclude=<path>`, `--coverage.exclude=<path1> --coverage.exclude=<path2>`

使用全局模式排除在覆盖范围之外的文件列表。

该选项覆盖所有默认选项。添加新的忽略模式时，扩展默认选项：

```ts
import { coverageConfigDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      exclude: ['**/custom-pattern/**', ...coverageConfigDefaults.exclude],
    },
  },
})
```

::: tip NOTE
Vitest 会自动将测试文件的 `include` 模式添加到 `coverage.exclude` 的默认值中。
:::

#### coverage.all

- **类型:** `boolean`
- **默认值:** `true`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.all`, `--coverage.all=false`

是否将所有文件（包括未测试的文件）包括在报告中。

#### coverage.clean

- **类型:** `boolean`
- **默认值:** `true`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.clean`, `--coverage.clean=false`

运行测试之前是否清除覆盖率结果

#### coverage.cleanOnRerun

- **类型:** `boolean`
- **默认值:** `true`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.cleanOnRerun`, `--coverage.cleanOnRerun=false`

监视重新运行时是否清除覆盖率报告。设置为 `false` 可保留观察模式下上次运行的覆盖结果。

#### coverage.reportsDirectory

- **类型:** `string`
- **默认值:** `'./coverage'`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.reportsDirectory=<path>`

::: warning
如果启用了 `coverage.clean`（默认值），Vitest 会在运行测试前删除此目录。
:::

配置测试覆盖率报告写入的目录。

要预览覆盖范围报告，请使用 [HTML reporter](/guide/reporters.html#html-reporter), 该选项必须设置为 html 报告目录的子目录 (比如 `./html/coverage`).

#### coverage.reporter

- **类型:** `string | string[] | [string, {}][]`
- **默认值:** `['text', 'html', 'clover', 'json']`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.reporter=<reporter>`, `--coverage.reporter=<reporter1> --coverage.reporter=<reporter2>`

配置要使用的测试覆盖率报告器。查看 [istanbul 文档](https://istanbul.js.org/docs/advanced/alternative-reporters/) 来了解报告详情。有关报告特定选项的详细信息，请参阅 [`@types/istanbul-reporter`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/276d95e4304b3670eaf6e8e5a7ea9e265a14e338/types/istanbul-reports/index.d.ts)。

该报告器支持三种不同的类型:

- 单个报告器: `{ reporter: 'html' }`
- 无配置的多个报告器: `{ reporter: ['html', 'json'] }`
- 有配置的单个或多个报告器:
  <!-- eslint-skip -->
  ```ts
  {
    reporter: [
      ["lcov", { projectRoot: "./src" }],
      ["json", { file: "coverage.json" }],
      ["text"],
    ];
  }
  ```

我们还可以传递自定义覆盖报告器。查看[自定义覆盖报告器](/guide/coverage#%E8%87%AA%E5%AE%9A%E4%B9%89%E8%A6%86%E7%9B%96%E7%8E%87%E6%8F%90%E4%BE%9B%E8%80%85)了解更多详情。

<!-- eslint-skip -->

```ts
{
  reporter: [
    // Specify reporter using name of the NPM package
    "@vitest/custom-coverage-reporter",
    ["@vitest/custom-coverage-reporter", { someOption: true }],

    // Specify reporter using local path
    "/absolute/path/to/custom-reporter.cjs",
    ["/absolute/path/to/custom-reporter.cjs", { someOption: true }],
  ];
}
```

我们可以在 Vitest UI 中查看覆盖率报告：查看 [Vitest UI Coverage](/guide/coverage#vitest-ui) 了解更多详情。

#### coverage.reportOnFailure {#coverage-reportonfailure}

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.reportOnFailure`, `--coverage.reportOnFailure=false`

即使测试失败也会生成覆盖率报告。

#### coverage.allowExternal

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.allowExternal`, `--coverage.allowExternal=false`

收集 [项目`root`](#root) 之外文件的覆盖率。

#### coverage.excludeAfterRemap <Version>2.1.0</Version> {#coverage-exclude-after-remap}

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.excludeAfterRemap`, `--coverage.excludeAfterRemap=false`

在覆盖范围重新映射到原始源后再次应用排除。
当你的源文件被转译并且可能包含非源文件的源映射时，这很有用。

当你看到报告中显示的文件与你的 `coverage.exclude` 模式匹配时，请使用此选项。

#### coverage.skipFull

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.skipFull`, `--coverage.skipFull=false`

是否显示具有 100% 语句、分支和函数的测试覆盖率的文件。

#### coverage.thresholds

覆盖范围阈值选项

##### coverage.thresholds.lines

- **类型:** `number`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.lines=<number>`

lines 的全局阈值。
更多信息请查看 [istanbul documentation](https://github.com/istanbuljs/nyc#coverage-thresholds)。

##### coverage.thresholds.functions

- **类型:** `number`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.functions=<number>`

functions 的全局阈值。
更多信息请查看 [istanbul documentation](https://github.com/istanbuljs/nyc#coverage-thresholds)。

##### coverage.thresholds.branches

- **类型:** `number`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.branches=<number>`

branches 的全局阈值。
更多信息请查看 [istanbul documentation](https://github.com/istanbuljs/nyc#coverage-thresholds)。

##### coverage.thresholds.statements

- **类型:** `number`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.statements=<number>`

statements 的全局阈值。
更多信息请查看 [istanbul documentation](https://github.com/istanbuljs/nyc#coverage-thresholds)。

##### coverage.thresholds.perFile

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.perFile`, `--coverage.thresholds.perFile=false`

检查每个文件的阈值。

##### coverage.thresholds.autoUpdate

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.autoUpdate=<boolean>`

如果当前覆盖率高于配置的阈值时，将所有阈值 `lines` 、`functions` 、`branches` 和 `statements` 更新到配置文件中。
此选项有助于在覆盖率提高时保持阈值不变。

##### coverage.thresholds.100

- **类型:** `boolean`
- **默认值:** `false`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.thresholds.100`, `--coverage.thresholds.100=false`

将全局阈值设置为 100。
这是 `--coverage.thresholds.lines 100 --coverage.thresholds.functions 100 --coverage.thresholds.branches 100 --coverage.thresholds.statements 100` 的快捷方式。

##### coverage.thresholds[glob-pattern]

- **类型:** `{ statements?: number functions?: number branches?: number lines?: number }`
- **默认值:** `undefined`
- **可用的测试提供者:** `'v8' | 'istanbul'`

设置与 glob 模式匹配的文件的阈值。

::: tip NOTE
Vitest counts all files, including those covered by glob-patterns, into the global coverage thresholds.
This is different from Jest behavior.
:::

<!-- eslint-skip -->

```ts
{
  coverage: {
    thresholds: {
      // Thresholds for all files
      functions: 95,
      branches: 70,

      // Thresholds for matching glob pattern
      'src/utils/**.ts': {
        statements: 95,
        functions: 90,
        branches: 85,
        lines: 80,
      },

      // Files matching this pattern will only have lines thresholds set.
      // Global thresholds are not inherited.
      '**/math.ts': {
        lines: 100,
      }
    }
  }
}
```

##### coverage.thresholds[glob-pattern].100 <Version>2.1.0</Version> {#coverage-thresholds-glob-pattern-100}

- **Type:** `boolean`
- **Default:** `false`
- **Available for providers:** `'v8' | 'istanbul'`

Sets thresholds to 100 for files matching the glob pattern.

<!-- eslint-skip -->
```ts
{
  coverage: {
    thresholds: {
      // Thresholds for all files
      functions: 95,
      branches: 70,

      // Thresholds for matching glob pattern
      'src/utils/**.ts': { 100: true },
      '**/math.ts': { 100: true }
    }
  }
}
```

#### coverage.ignoreEmptyLines

- **类型:** `boolean`
- **默认值:** `true` (`false` in v1)
- **可用的测试提供者:** `'v8'`
- **命令行终端:** `--coverage.ignoreEmptyLines=<boolean>`

忽略空行、注释和其他非运行时代码，如 Typescript 类型。

该选项只有在使用的编译器删除了转译代码中的注释和其他非运行时代码时才有效。
默认情况下，Vite 使用 ESBuild，它会删除 `.ts`、`.tsx` 和 `.jsx` 文件中的注释和 Typescript 类型。

如果还想将 ESBuild 应用于其他文件，请在 [`esbuild` options](https://cn.vitejs.dev/config/shared-options.html#esbuild) 中定义它们：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  esbuild: {
    // 使用 ESBuild 转换所有文件以删除代码覆盖率中的注释。
    // `test.coverage.ignoreEmptyLines` 需要工作：
    include: ['**/*.js', '**/*.jsx', '**/*.mjs', '**/*.ts', '**/*.tsx'],
  },
  test: {
    coverage: {
      provider: 'v8',
      ignoreEmptyLines: true,
    },
  },
})
```

#### coverage.ignoreClassMethods

- **类型:** `string[]`
- **默认值:** `[]`
- **可用的测试提供者:** `'istanbul'`
- **命令行终端:** `--coverage.ignoreClassMethods=<method>`

设置为要忽略覆盖率的类方法名称数组。参考 [istanbul 文档](https://github.com/istanbuljs/nyc#ignoring-methods) 来了解详情。

#### coverage.watermarks

- **类型:**
<!-- eslint-skip -->

```ts
{
  statements?: [number, number],
  functions?: [number, number],
  branches?: [number, number],
  lines?: [number, number]
}
```

- **默认值:**
<!-- eslint-skip -->

```ts
{
  statements: [50, 80],
  functions: [50, 80],
  branches: [50, 80],
  lines: [50, 80]
}
```

- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.watermarks.statements=50,80`， `--coverage.watermarks.branches=50,80`

语句、行、分支和函数的水印。有关更多信息，请参见 [istanbul 文档](https://github.com/istanbuljs/nyc#high-and-low-watermarks)。

#### coverage.processingConcurrency

- **类型:** `boolean`
- **默认值:** `Math.min(20, os.availableParallelism?.() ?? os.cpus().length)`
- **可用的测试提供者:** `'v8' | 'istanbul'`
- **命令行终端:** `--coverage.processingConcurrency=<number>`

处理覆盖率结果时使用的并发限制。

#### coverage.customProviderModule

- **类型:** `string`
- **可用的测试提供者:** `'custom'`
- **命令行终端:** `--coverage.customProviderModule=<path or module name>`

指定自定义覆盖率提供者的模块名称或路径。有关详细信息，请参阅[指南 - 自定义覆盖率提供者](/guide/coverage#custom-coverage-provider)。

### testNamePattern<NonProjectOption />

- **类型** `string | RegExp`
- **命令行终端:** `-t <pattern>`, `--testNamePattern=<pattern>`, `--test-name-pattern=<pattern>`

使用与模式匹配的全名运行测试。
如果你将 `OnlyRunThis` 添加到此属性，将跳过测试名称中不包含单词 `OnlyRunThis` 的测试。

```js
import { expect, test } from 'vitest'

// run
test('OnlyRunThis', () => {
  expect(true).toBe(true)
})

// skipped
test('doNotRun', () => {
  expect(true).toBe(true)
})
```

### open<NonProjectOption />

- **类型:** `boolean`
- **默认值:** `!process.env.CI`
- **命令行终端:** `--open`, `--open=false`

打开 Vitest UI (WIP: 赞助者计划可用)

### api

- **类型:** `boolean | number`
- **默认值:** `false`
- **命令行终端:** `--api`, `--api.port`, `--api.host`, `--api.strictPort`

提供 API 服务的端口。当设置为 true 时，默认端口为 51204

### browser {#browser}

- **类型:** `{ enabled?, name?, provider?, headless?, api? }`
- **默认值:** `{ enabled: false, headless: process.env.CI, api: 63315 }`
- **命令行终端:** `--browser`, `--browser=<name>`, `--browser.name=chrome --browser.headless`

在浏览器中运行 Vitest 测试。我们默认使用 [WebdriverIO](https://webdriver.io/) 来运行测试，但可以使用 [browser.provider](#browser-provider) 选项进行配置。

::: tip NOTE
在 [指南页面](/guide/browser/) 中阅读有关在真实浏览器中进行测试的更多信息。
:::

::: warning
这是一项实验性功能。重大更改可能不会遵循 semver，请在使用时锁定 Vitest 的版本。
:::

#### browser.enabled

- **类型:** `boolean`
- **默认值:** `false`
- **命令行终端:** `--browser`, `--browser.enabled=false`

默认在浏览器内运行所有测试。

#### browser&#46;name

- **类型:** `string`
- **命令行终端:** `--browser=safari`

在特定浏览器中运行所有测试。不同的浏览器提供商有以下选项：

- `webdriverio`: `firefox`, `chrome`, `edge`, `safari`
- `playwright`: `firefox`, `webkit`, `chromium`
- 自定义: 将传递给提供者的任何字符串

#### browser.headless

- **类型:** `boolean`
- **默认值:** `process.env.CI`
- **命令行终端:** `--browser.headless`, `--browser.headless=false`

以 `headless` 模式运行浏览器。如果你在 CI 中运行 Vitest，它将默认启用。

#### browser.isolate

- **类型:** `boolean`
- **默认值:** `true`
- **命令行终端:** `--browser.isolate`, `--browser.isolate=false`

在单独的 iframe 中运行每个测试。

#### browser.testerHtmlPath <Version>2.1.4</Version> {#browser-testerhtmlpath}

- **Type:** `string`
- **Default:** `@vitest/browser/tester.html`

A path to the HTML entry point. Can be relative to the root of the project. This file will be processed with [`transformIndexHtml`](https://vite.dev/guide/api-plugin#transformindexhtml) hook.

#### browser.api

- **类型:** `number | { port?, strictPort?, host? }`
- **默认值:** `63315`
- **命令行终端:** `--browser.api=63315`, `--browser.api.port=1234, --browser.api.host=example.com`

为在浏览器中提供代码的 Vite 服务器配置选项。它不影响 [`test.api`](#api) 选项。默认情况下，Vitest 会分配端口 `63315`，以避免与开发服务器冲突，从而允许同时运行这两个端口。

#### browser.provider

- **类型:** `'webdriverio' | 'playwright' | 'preview' | string`
- **默认值:** `'preview'`
- **命令行终端:** `--browser.provider=playwright`

设置运行浏览器测试时浏览器的路径。Vitest 提供了三个浏览器驱动选项: `preview`（默认）、`webdriverio` 和 `playwright` 。自定义提供商应该使用 `default` 进行导出，并具有如下类型签名:

```ts
export interface BrowserProvider {
  name: string
  getSupportedBrowsers: () => readonly string[]
  initialize: (
    ctx: Vitest,
    options: { browser: string, options?: BrowserProviderOptions }
  ) => Awaitable<void>
  openPage: (url: string) => Awaitable<void>
  close: () => Awaitable<void>
}
```

::: warning
这是一个对库作者友好的的高级 API。如果你只需要在浏览器中运行测试，请使用 [browser](#browser) 选项。
:::

#### browser.providerOptions {#browser-provideroptions}

- **类型:** `BrowserProviderOptions`

调用 `provider.initialize` 时将传递给提供程序的选项。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    browser: {
      providerOptions: {
        launch: {
          devtools: true,
        },
      },
    },
  },
})
```

::: tip
为了在使用内置提供程序时获得更好的类型安全性，我们应该在 [config 文件](/config/) 中引用以下类型之一（对于我们正在使用的提供程序）：

```ts
/// <reference types="@vitest/browser/providers/playwright" />
/// <reference types="@vitest/browser/providers/webdriverio" />
```

:::

#### browser.ui {#browser-ui}

- **类型:** `boolean`
- **默认值:** `!isCI`
- **命令行终端:** `--browser.ui=false`

是否应该将 Vitest UI 注入到页面中。默认情况下，在开发过程中注入 UI iframe。

#### browser.viewport {#browser-viewport}

- **Type:** `{ width, height }`
- **Default:** `414x896`

默认 iframe 的viewport。

#### browser.locators {#browser-locators}

Options for built-in [browser locators](/guide/browser/locators).

##### browser.locators.testIdAttribute

- **Type:** `string`
- **Default:** `data-testid`

Attribute used to find elements with `getByTestId` locator.

#### browser.screenshotDirectory {#browser-screenshotdirectory}

- **Type:** `string`
- **Default:** `__snapshots__` in the test file directory

相对于 `root` 的快照目录路径。

#### browser.screenshotFailures {#browser-screenshotfailures}

- **Type:** `boolean`
- **Default:** `!browser.ui`

如果测试失败，Vitest 是否应截图。

#### browser.orchestratorScripts {#browser-orchestratorscripts}

- **类型:** `BrowserScript[]`
- **默认值:** `[]`

在启动测试 iframe 之前，应注入orchestrator HTML 中的自定义脚本。此 HTML 文档仅设置 iframe，并不实际导入您的代码。

脚本 `src` 和 `content` 将由 Vite 插件处理。脚本应以以下格式提供：

```ts
export interface BrowserScript {
  /**
   * 如果提供了 "content"，且类型为 "module"，则这将是其标识符。
   *
   * 如果使用的是 TypeScript，可以在此处添加 `.ts` 扩展名，例如
   * @default `injected-${index}.js`
   */
  id?: string
  /**
   * 要注入的 JavaScript 内容。如果类型为 “module”，Vite 插件将处理此字符串。
   *
   * 你可以使用 `id` 给 Vite 有关文件扩展名的提示。
   */
  content?: string
  /**
   * 脚本的路径。该值由 Vite 解析，因此可以是节点模块或文件路径。
   */
  src?: string
  /**
   * 如果要异步加载脚本
   */
  async?: boolean
  /**
   * Script type.
   * @default 'module'
   */
  type?: string
}
```

#### browser.testerScripts {#browser-testerscripts}

- **类型:** `BrowserScript[]`
- **默认值:** `[]`

应在测试环境启动前注入测试器 HTML 的自定义脚本。这对注入 Vitest 浏览器实现所需的 polyfills 非常有用。建议在几乎所有情况下使用 [`setupFiles`](#setupfiles)，而不要使用此脚本。

脚本 `src` 和 `content` 将由 Vite 插件处理。

#### browser.commands {#browser-commands}

- **类型:** `Record<string, BrowserCommand>`
- **默认值:** `{ readFile, writeFile, ... }`

自定义[命令](/guide/browser/commands)，可在浏览器测试期间从 `@vitest/browser/commands` 导入。

### clearMocks

- **类型:** `boolean`
- **默认值:** `false`

每次测试前，都会对所有 spies 调用 [`.mockClear()`](/api/mock#mockclear)。
这将清除模拟历史记录，而不会影响模拟实现。

### mockReset

- **类型:** `boolean`
- **默认值:** `false`

每次测试前，都会对所有 spies 调用 [`.mockReset()`](/api/mock#mockreset)。
这将清除模拟历史记录，并将每个实现重置为原始状态。

### restoreMocks

- **类型:** `boolean`
- **默认值:** `false`

每次测试前，都会对所有 spies 调用 [`.mockRestore()`](/api/mock#mockrestore)。
这将清除模拟历史记录，将每个实现恢复为原始实现，并恢复被监视对象的原始描述符。

### unstubEnvs {#unstubenvs}

- **类型:** `boolean`
- **默认值:** `false`

将在每次测试前调用 [`vi.unstubAllEnvs`](/api/#vi-unstuballenvs)。

### unstubGlobals {#unstubglobals}

- **类型:** `boolean`
- **默认值:** `false`

将在每次测试前调用 [`vi.unstubAllGlobals`](/api/#vi-unstuballglobals)。

### testTransformMode {#testtransformmode}

- **类型:** `{ web?, ssr? }`

确定与测试中的 glob 模式匹配的所有导入模块的转换方法。默认情况下，依赖于环境。例如，使用 JSDOM 环境的测试将处理所有带有 `ssr: false` 标志的文件，而使用 Node 环境的测试将处理所有带有 `ssr: true` 的模块。

#### testTransformMode.ssr

- **类型:** `string[]`
- **默认值:** `[]`

对指定的文件使用 SSR 转换管道。<br>
Vite 插件在处理这些文件时会收到 `ssr: true` 标志。

#### testTransformMode&#46;web

- **类型:** `string[]`
- **默认值:** `[]`

首先会进行正常的转换管道（针对浏览器），然后进行 SSR 重写以在 Node 中运行代码。<br>
Vite 插件在处理这些文件时会收到 `ssr: false` 标志。

当你使用 JSX 作为 React 以外的组件模型（例如 Vue JSX 或 SolidJS）时，你可能需要进行如下配置以使 `.tsx` / `.jsx` 转换为客户端组件：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    transformMode: {
      web: [/\.[jt]sx$/],
    },
  },
})
```

### snapshotFormat<NonProjectOption />

- **类型:** `PrettyFormatOptions`

测试快照的格式选项。这些选项被传递给 [`pretty-format`](https://www.npmjs.com/package/pretty-format)。

::: tip
请注意，此对象上的 `plugins` 字段将被忽略。

如果你需要通过 pretty-format 插件扩展快照序列器，请使用 [`expect.addSnapshotSerializer`](/api/expect#expect-addsnapshotserializer) 或 [snapshotSerializers](#snapshotserializers) 选项。
:::

### snapshotSerializers<NonProjectOption /> {#snapshotserializers}

- **类型:** `string[]`
- **默认值:** `[]`

快照测试的快照序列化程序模块的路径列表，如果要添加自定义快照序列化器，则非常有用。有关详细信息，请参阅[自定义序列化器](/guide/snapshot#custom-serializer)。

### resolveSnapshotPath<NonProjectOption />

- **类型**: `(testPath: string, snapExtension: string, context: { config: SerializedConfig }) => string`
- **默认值**: stores snapshot files in `__snapshots__` directory

覆盖快照的默认路径。例如，要在测试文件旁边存储一下快照：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    resolveSnapshotPath: (testPath, snapExtension) => testPath + snapExtension,
  },
})
```

### allowOnly

- **类型**: `boolean`
- **默认值**: `!process.env.CI`
- **命令行终端:** `--allowOnly`, `--allowOnly=false`

允许标记为 only 的测试和套件。

### dangerouslyIgnoreUnhandledErrors<NonProjectOption />

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端:** `--dangerouslyIgnoreUnhandledErrors` `--dangerouslyIgnoreUnhandledErrors=false`

忽略发生的任何未处理的错误。

### passWithNoTests<NonProjectOption />

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端:** `--passWithNoTests`, `--passWithNoTests=false`

如果没有找到测试，Vitest 不会失败。

### logHeapUsage

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端:** `--logHeapUsage`, `--logHeapUsage=false`

每次测试后显示堆的使用情况。用于调试内存是否泄漏。

### css

- **类型**: `boolean | { include?, exclude?, modules? }`

配置是否应处理 CSS。 排除后，CSS 文件将被替换为空字符串以绕过后续处理。 CSS 模块将返回一个代理以不影响运行时。

#### css.include

- **类型**: `RegExp | RegExp[]`
- **默认值**: `[]`

将返回匹配正则表达式并将由 Vite 管道处理的实际 CSS 文件。

:::tip
如果需要处理所有 CSS 文件，请使用 `/.+/`。
:::

#### css.exclude

- **类型**: `RegExp | RegExp[]`
- **默认值**: `[]`

将返回匹配正则表达式的空 CSS 文件。

#### css.modules

- **类型**: `{ classNameStrategy? }`
- **默认值**: `{}`

#### css.modules.classNameStrategy

- **类型**: `'stable' | 'scoped' | 'non-scoped'`
- **默认值**: `'stable'`

如果你决定处理 CSS 文件，你可以配置 CSS 模块中的类名是否在限定范围内。 默认情况下，Vitest 会导出一个代理，绕过 CSS 模块处理。 你可以选择以下选项之一：

- `stable`: 类名将生成为`_${name}_${hashedFilename}`，这意味着如果 CSS 内容发生变化，生成的类将保持不变，但如果文件名被修改，或者文件名将发生变化 被移动到另一个文件夹。 如果你使用快照功能，此设置很有用。
- `scoped`: 类名将照常生成，遵照 `css.modules.generateScopedName` 方法，如果你有的话。 默认情况下，文件名将生成为`_${name}_${hash}`，其中 hash 包括文件名和文件内容。
- `non-scoped`: 类名将保留 CSS 中定义的名称。

::: warning
在默认的情况下，Vitest 导出代理会绕过 CSS 模块处理。 如果你依赖类的 CSS 属性，就必须使用 `include` 选项启用 CSS 处理。
:::

### maxConcurrency

- **类型**: `number`
- **默认值**: `5`
- **命令行终端**: `--max-concurrency=10`, `--maxConcurrency=10`

使用 `test.concurrent` 标记允许同时运行的最大测试数量。

当出现可用插槽时，超过此限制的测试将排队运行。

### cache<NonProjectOption />

- **类型**: `false`
- **命令行终端**: `--no-cache`, `--cache=false`

如果要禁用缓存功能，请使用此选项。目前，Vitest 会对测试结果进行缓存，优先运行时间较长和失败的测试。

缓存目录由 Vite 的 [`cacheDir`](https://vitejs.dev/config/shared-options.html#cachedir) 选项控制：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  cacheDir: 'custom-folder/.vitest'
})
```

您可以使用 `process.env.VITEST` 来限制目录，使其仅用于 Vitest：

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  cacheDir: process.env.VITEST ? 'custom-folder/.vitest' : undefined
})
```

### sequence

- **类型**: `{ sequencer?, shuffle?, seed?, hooks?, setupFiles? }`

配置测试运行顺序的选项。

你可以使用点符号向 CLI 提供序列选项：

```sh
npx vitest --sequence.shuffle --sequence.seed=1000
```

#### sequence.sequencer<NonProjectOption />

- **类型**: `TestSequencerConstructor`
- **默认值**: `BaseSequencer`

定义分片和排序的自定义类。你可以从 `vitest/node` 扩展 `BaseSequencer`，如果你只需要重新定义 `sort` 和 `shard` 方法之一，但两者都应该存在。

分片是在排序之前进行的，并且只有提供了 `--shard` 选项的情况下才会生效。

#### sequence.shuffle

- **类型**: `boolean | { files?, tests? }`
- **默认值**: `false`
- **命令行终端**: `--sequence.shuffle`, `--sequence.shuffle=false`

如果你希望测试随机运行，可以使用此选项或 CLI 参数 [`--sequence.shuffle`](/guide/cli) 启用它。

Vitest 通常使用缓存对测试进行排序，因此长时间运行的测试会更早开始 - 这会使测试运行得更快。 如果你的测试将以随机顺序运行，你将失去这种性能改进，但跟踪意外依赖于先前运行的测试可能很有用。

- **类型**: `boolean | { files?, tests? }`
- **默认值**: `false`
- **命令行终端**: `--sequence.shuffle`, `--sequence.shuffle=false`

#### sequence.shuffle.files {#sequence-shuffle-files}

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端**: `--sequence.shuffle.files`, `--sequence.shuffle.files=false`

是否随机化文件，注意如果启用此选项，长时间运行的测试将不会提前启动。

#### sequence.shuffle.tests {#sequence-shuffle-tests}

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端**: `--sequence.shuffle.tests`, `--sequence.shuffle.tests=false`

是否随机测试。

#### sequence.concurrent {#sequence-concurrent}

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端**: `--sequence.concurrent`, `--sequence.concurrent=false`

如果你希望测试并行运行，可以使用此选项或 CLI 参数 [`--sequence.concurrent`](/guide/cli) 启用它。

#### sequence.seed<NonProjectOption />

- **类型**: `number`
- **默认值**: `Date.now()`
- **命令行终端**: `--sequence.seed=1000`

如果测试以随机顺序运行，则设置随机化种子。

#### sequence.hooks

- **类型**: `'stack' | 'list' | 'parallel'`
- **默认值**: `'parallel'`
- **命令行终端**: `--sequence.hooks=<value>`

更改钩子的执行顺序。

- `stack` 将以相反的顺序排列 "after" 钩子，"before" 钩子将按照它们定义的顺序运行
- `list` 将按照定义的顺序对所有钩子进行排序
- `parallel` 将并行运行单个组中的钩子（父套件中的钩子仍将在当前套件的钩子之前运行）

::: tip
该选项不会影响 [`onTestFinished`](/api/#ontestfinished)。它总是以相反的顺序调用。
:::

#### sequence.setupFiles {#sequence-setupfiles}

- **类型**: `'list' | 'parallel'`
- **默认值**: `'parallel'`
- **命令行终端**: `--sequence.setupFiles=<value>`

更改安装文件的执行顺序。

- `list` 将按照定义的顺序运行安装文件
- `parallel` 将并行运行设置文件

### typecheck

用于配置 [typechecking](/guide/testing-types) 测试环境的选项。

#### typecheck.enabled {#typecheck-enabled}

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端**: `--typecheck`, `--typecheck.enabled`

常规测试时是否进行类型检查。

#### typecheck.only {#typecheck-only}

- **类型**: `boolean`
- **默认值**: `false`
- **命令行终端**: `--typecheck.only`

启用类型检查时，仅运行类型检查测试。使用 CLI 时，此选项将自动启用类型检查。

#### typecheck.checker

- **类型**: `'tsc' | 'vue-tsc' | string`
- **默认值**: `tsc`

设置类型检查的检测器。Vitest 将根据类型生成具有某些参数的进程，以便于解析。 Checker 应该实现与 `tsc` 相同的输出格式。

你需要安装一个包才能使用 typecheker:

- `tsc` requires `typescript` package
- `vue-tsc` requires `vue-tsc` package

你还可以将路径传递到自定义二进制文件或命令名称，该路径会产生与 `tsc --noEmit --pretty false` 相同的输出。

#### typecheck.include

- **类型**: `string[]`
- **默认值**: `['**/*.{test,spec}-d.?(c|m)[jt]s?(x)']`

匹配包含测试文件的 glob 规则。

#### typecheck.exclude

- **类型**: `string[]`
- **默认值**: `['**/node_modules/**', '**/dist/**', '**/cypress/**', '**/.{idea,git,cache,output,temp}/**']`

匹配排除测试文件的 glob 规则。

#### typecheck.allowJs

- **类型**: `boolean`
- **默认值**: `false`

检查有 `@ts-check` 注释的 JS 文件。 如果你在 tsconfig 中启用它，则不会覆盖它。

#### typecheck.ignoreSourceErrors

- **类型**: `boolean`
- **默认值**: `false`

如果 Vitest 在测试文件之外发现错误，不要失败。 这根本不会向你显示非测试错误。

默认情况下，如果 Vitest 发现源错误，它将测试套件中抛出失败。

#### typecheck.tsconfig

- **类型**: `string`
- **默认值**: _tries to find closest tsconfig.json_

自定义 tsconfig 的路径，相对于项目根目录。

### slowTestThreshold<NonProjectOption />

- **类型**: `number`
- **默认值**: `300`
- **命令行终端:**:`--slow-test-threshold=<number>`, `--slowTestThreshold=<number>`

如果测试被认为是缓慢的，那么会在报告结果中显示毫秒值。

### chaiConfig {#chaiconfig}

- **类型:** `{ includeStack?, showDiff?, truncateThreshold? }`
- **默认值:** `{ includeStack: false, showDiff: true, truncateThreshold: 40 }`

等同于 [Chai 配置](https://github.com/chaijs/chai/blob/4.x.x/lib/chai/config.js)。

#### chaiConfig.includeStack

- **类型:** `boolean`
- **默认值:** `false`

影响断言错误消息中是否包含堆栈跟踪。默认值为 false，在错误消息中抑制堆栈跟踪。

#### chaiConfig.showDiff

- **类型:** `boolean`
- **默认值:** `true`

影响是否应在抛出的 AssertionErrors 中包含 `showDiff` 标志。`false` 始终为 `false`；`true` 将在断言请求显示差异时为 `true`。

#### chaiConfig.truncateThreshold

- **类型:** `number`
- **默认值:** `40`

设置断言错误中实际值和期望值的长度阈值。如果超过此阈值，例如对于大型数据结构，该值将被替换为类似 `[ Array(3) ]` 或 `{ Object (prop1, prop2) }` 的内容。如果要完全禁用截断，请将其设置为 `0`。

此配置选项影响在 `test.each` 标题和断言错误消息中截断值的方式。

### bail {#bail}

- **类型:** `number`
- **默认值:** `0`
- **命令行终端**: `--bail=<value>`

当给定数量的测试失败时停止测试执行。

默认情况下，即使其中一些测试失败，Vitest 也会运行你的所有测试用例。这可能不适用于 CI 构建，你只对 100% 成功的构建感兴趣，并且希望在测试失败时尽早停止测试执行。`bail` 选项可用于通过在发生故障时防止运行更多测试来加速 CI 运行。

### retry {#retry}

- **类型:** `number`
- **默认值:** `0`
- **命令行终端:** `--retry=<value>`

如果测试失败，请重试特定次数的测试。

### onConsoleLog<NonProjectOption />

- **类型**: `(log: string, type: 'stdout' | 'stderr') => boolean | void`

在测试自定义 `console.log` 的处理程序。如果返回 `false`，Vitest 将不会将日志打印到控制台上。

这在过滤掉来自第三方库的日志时会非常有用。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    onConsoleLog(log: string, type: 'stdout' | 'stderr'): boolean | void {
      return !(log === 'message from third party library' && type === 'stdout')
    },
  },
})
```

### onStackTrace<NonProjectOption /> {#onstacktrace}

- **类型**: `(error: Error, frame: ParsedStack) => boolean | void`

在处理错误时，对每个堆栈的每个帧应用过滤功能。第一个参数 `error` 是一个与标准 `Error` 具有相同属性的对象，但它不是实际实例。

可用于从第三方库中筛选堆栈跟踪帧。

```ts
import type { ParsedStack } from 'vitest'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    onStackTrace(error: Error, { file }: ParsedStack): boolean | void {
      // If we've encountered a ReferenceError, show the whole stack.
      if (error.name === 'ReferenceError') {
        return
      }

      // Reject all frames from third party libraries.
      if (file.includes('node_modules')) {
        return false
      }
    },
  },
})
```

### diff

- **类型:** `string`
- **命令行终端:** `--diff=<path>`

`DiffOptions` 对象或者是一个导出 `DiffOptions` 的模块路径。如果我们想要自定义差异显示，这将非常有用。

例如，作为一个配置对象：

```ts
import { defineConfig } from 'vitest/config'
import c from 'picocolors'

export default defineConfig({
  test: {
    diff: {
      aIndicator: c.bold('--'),
      bIndicator: c.bold('++'),
      omitAnnotationLines: true,
    },
  },
})
```

Or as a module:

:::code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    diff: './vitest.diff.ts',
  },
})
```

```ts [vitest.diff.ts]
import type { DiffOptions } from 'vitest'
import c from 'picocolors'

export default {
  aIndicator: c.bold('--'),
  bIndicator: c.bold('++'),
  omitAnnotationLines: true,
} satisfies DiffOptions
```
:::

#### diff.expand

- **类型**: `boolean`
- **默认值**: `true`
- **命令行终端:** `--diff.expand=false`

Expand all common lines.

#### diff.truncateThreshold

- **类型**: `number`
- **默认值**: `0`
- **命令行终端:** `--diff.truncateThreshold=<path>`

要显示的差异结果的最大长度。超过此阈值的差异将被截断。
默认值为 0 时，截断不会生效。

#### diff.truncateAnnotation

- **类型**: `string`
- **默认值**: `'... Diff result is truncated'`
- **命令行终端:** `--diff.truncateAnnotation=<annotation>`

在 diff 结果末尾输出的注释（如果被截断）。

#### diff.truncateAnnotationColor

- **类型**: `DiffOptionsColor = (arg: string) => string`
- **默认值**: `noColor = (string: string): string => string`

截断注释的颜色，默认为无色输出。

#### diff.printBasicPrototype

- **Type**: `boolean`
- **Default**: `false`

Print basic prototype `Object` and `Array` in diff output

### fakeTimers

- **类型:** `FakeTimerInstallOpts`

当使用 [`vi.useFakeTimers()`](/api/vi#vi-usefaketimers)时，Vitest 将向 [`@sinon/fake-timers`](https://www.npmjs.com/package/@sinonjs/fake-timers) 传递的选项。

#### fakeTimers.now

- **类型:** `number | Date`
- **默认值:** `Date.now()`

用指定的 unix 时间安装假计时器。

#### fakeTimers.toFake

- **类型:** `('setTimeout' | 'clearTimeout' | 'setImmediate' | 'clearImmediate' | 'setInterval' | 'clearInterval' | 'Date' | 'nextTick' | 'hrtime' | 'requestAnimationFrame' | 'cancelAnimationFrame' | 'requestIdleCallback' | 'cancelIdleCallback' | 'performance' | 'queueMicrotask')[]`
- **默认值:** everything available globally except `nextTick`

包含要伪造的全局方法和 API 名称的数组。

要只模拟 `setTimeout()` 和 `nextTick()` ，请将此属性指定为 `['setTimeout','nextTick']`。

使用 `--pool=forks` 在 `node:child_process` 内运行 Vitest 时，不支持模拟 `nextTick`。NodeJS 在 `node:child_process` 中内部使用了 `process.nextTick`，当模拟它时会挂起。使用 `--pool=threads` 运行 Vitest 时支持模拟 `nextTick`。

#### fakeTimers.loopLimit

- **类型:** `number`
- **默认值:** `10_000`

调用 [`vi.runAllTimers()`](/api/vi#vi-runalltimers) 时将运行的计时器的最大数量。

#### fakeTimers.shouldAdvanceTime

- **类型:** `boolean`
- **默认值:** `false`

告诉 @sinonjs/fake-timers 根据实际系统时间的变化自动递增模拟时间（例如，实际系统时间每变化 20 毫秒，模拟时间就递增 20 毫秒）。

#### fakeTimers.advanceTimeDelta

- **类型:** `number`
- **默认值:** `20`

只有在和 `shouldAdvanceTime: true` 一起使用时才相关。实际系统时间每发生一次 advanceTimeDelta ms 变化，模拟时间就增加一次 advanceTimeDelta ms。

#### fakeTimers.shouldClearNativeTimers

- **类型:** `boolean`
- **默认值:** `true`

通过委托各自的处理程序，告诉假冒计时器清除 "native"（即非假冒）计时器。禁用时，如果计时器在启动假计时器会话之前已经存在，则可能导致意外行为。

### workspace<NonProjectOption /> {#workspace}

- **类型:** `string | TestProjectConfiguration`
- **命令行终端:** `--workspace=./file.js`
- **默认值:** `vitest.{workspace,projects}.{js,ts,json}` close to the config file or root

相对于[root](#root) 的 [workspace](/guide/workspace) 配置文件的路径。

Since Vitest 3, you can also define the workspace array in the root config. If the `workspace` is defined in the config manually, Vitest will ignore the `vitest.workspace` file in the root.

### isolate

- **类型:** `boolean`
- **默认值:** `true`
- **命令行终端:** `--no-isolate`, `--isolate=false`

在隔离的环境中运行测试。此选项对 `vmThreads` 和 `vmForks` 池没有影响。

如果你的代码不依赖于副作用（对于具有 `node` 环境的项目通常如此），禁用此选项可能会[改进性能](/guide/improving-performance)。

::: tip
你可以使用 [`poolOptions`](#poolOptions) 属性禁用特定池的隔离。
:::

### includeTaskLocation {#includeTaskLocation}

- **类型:** `boolean`
- **默认值:** `false`

Vitest API 在 [reporters](#reporters) 中接收任务时是否应包含`location`属性。如果您有大量测试，这可能会导致性能小幅下降。

`location` 属性的 `列` 和 `行` 值与原始文件中的 `test` 或 `describe` 位置相对应。

如果您没有明确禁用该选项，并且在运行 Vitest 时使用了该选项，则该选项将自动启用：
- [Vitest UI](/guide/ui)
- 或使用不带 [headless](/guide/browser/#headless) 模式的 [浏览器模式](/guide/browser/)
- 或使用[HTML 报告器](/guide/reporters#html-reporter)

::: tip
如果不使用依赖于该选项的自定义代码，该选项将不起作用。
:::

### snapshotEnvironment {#snapshotEnvironment}

- **类型:** `string`

自定义快照环境实现的路径。如果在不支持 Node.js API 的环境中运行测试，该选项将非常有用。此选项对浏览器运行程序没有任何影响。

该对象应具有 `SnapshotEnvironment` 的形状，用于解析和读/写快照文件：

```ts
export interface SnapshotEnvironment {
  getVersion: () => string
  getHeader: () => string
  resolvePath: (filepath: string) => Promise<string>
  resolveRawPath: (testPath: string, rawPath: string) => Promise<string>
  saveSnapshotFile: (filepath: string, snapshot: string) => Promise<void>
  readSnapshotFile: (filepath: string) => Promise<string | null>
  removeSnapshotFile: (filepath: string) => Promise<void>
}
```

如果只需覆盖部分 API，可从 `vitest/snapshot` 入口扩展默认的 `VitestSnapshotEnvironment` 。

::: warning
这是一个低级选项，仅适用于无法访问默认 Node.js API 的高级情况。

如果只需要配置快照功能，请使用 [`snapshotFormat`](#snapshotformat)或 [`resolveSnapshotPath`](#resolvesnapshotpath)选项。
:::

### env {#env}

- **类型:** `Partial<NodeJS.ProcessEnv>`

测试期间在 `process.env` 和 `import.meta.env` 中可用的环境变量。这些变量在主进程中不可用（例如在 `globalSetup` 中）。

### expect

- **类型:** `ExpectOptions`

#### expect.requireAssertions

- **类型:** `boolean`
- **默认值:** `false`

与每次测试开始时调用 [`expect.hasAssertions()`](/api/expect#expect-hasassertions) 相同。这可确保不会意外通过任何测试。

::: tip
这仅适用于 Vitest 的`expect`。如果您使用`assert`或`.should`断言，它们将不计算在内，并且您的测试将因缺少 expect 断言而失败。

您可以通过调用 `vi.setConfig({ expect: { requireAssertions: false } })` 来更改此值。该配置将应用于每个后续 `expect` 调用，直到手动调用 `vi.resetConfig`。
:::

#### expect.poll

[`expect.poll`](/api/expect#poll) 的全局配置选项。这些选项与您可以传递给 `expect.poll(condition, options)` 的选项相同。

##### expect.poll.interval

- **类型:** `number`
- **默认值:** `50`

轮询间隔（以毫秒为单位）

##### expect.poll.timeout

- **类型:** `number`
- **默认值:** `1000`

轮询超时时间（以毫秒为单位）

### printConsoleTrace

- **类型:** `boolean`
- **默认值:** `false`

调用任何`console`方法时始终打印控制台跟踪。这对于调试很有用。
