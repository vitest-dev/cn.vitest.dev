---
outline: [2, 3]
---

# Node API

::: warning
Vitest 暴露了实验性的私有 API。由于可能不遵循语义化版本规范（SemVer），因此可能会出现不兼容的更改，请在使用 Vitest 时锁定版本。
:::

## 启动 Vitest

你可以使用 Vitest 的 Node API 开始运行 Vitest 测试：

```js
import { startVitest } from 'vitest/node'

const vitest = await startVitest('test')

await vitest?.close()
```

如果测试可以启动，则 `startVitest` 函数返回 `Vitest` 实例。 如果出现以下情况之一，则返回 `undefined`：

- Vitest 未找到 `vite` 包 (通常与 Vitest 一起安装)
- 如果启用了 `coverage`，并且运行模式为 "test"，但并未安装 "coverage" 包（`@vitest/coverage-v8` 或 `@vitest/coverage-istanbul`）
- 如果未安装环境包 (`jsdom`/`happy-dom`/`@edge-runtime/vm`)

如果在运行期间返回 `undefined` 或者测试失败, Vitest 会将 `process.exitCode` 设置为 `1`。

如果未启用监视模式，Vitest 将会调用 `close` 方法。

如果启用了监视模式并且终端支持 TTY, 则 Vitest 会注册控制台快捷键。

你可以将过滤器列表作为第二个参数传递下去。Vitest 将仅运行包含其文件路径中至少一个传递字符串的测试。

此外，你可以使用第三个参数传递 CLI 参数，这将覆盖任何测试配置选项。

或者，你可以将完整的 Vite 配置作为第四个参数传递进去，这将优先于任何其他用户定义的选项。

运行测试后，您可以从 `state.getFiles` API 获取结果：

```ts
const vitest = await startVitest('test')

console.log(vitest.state.getFiles()) // [{ type: 'file', ... }]
```

自 Vitest 2.1 起，建议使用["Reported Tasks" API](/advanced/reporters#reported-tasks) 和 `state.getFiles`。今后，Vitest 将直接返回这些对象：

```ts
const vitest = await startVitest('test')

const [fileTask] = vitest.state.getFiles()
const testFile = vitest.state.getReportedEntity(fileTask)
```

## 创建 Vitest

你可以使用 `createVitest` 函数创建自己的 Vitest 实例. 它返回与 `startVitest` 相同的 `Vitest` 实例, 但不会启动测试，也不会验证已安装的包。

```js
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test', {
  watch: false,
})
```

## parseCLI

你可以使用此方法来解析 CLI 参数。它接受字符串（其中参数由单个空格分隔）或与 Vitest CLI 使用的格式相同的 CLI 参数的字符串数组。它返回一个过滤器和`选项`，你可以稍后将其传递给 `createVitest` 或 `startVitest` 方法。

```ts
import { parseCLI } from 'vitest/node'

parseCLI('vitest ./files.ts --coverage --browser=chrome')
```

## Vitest

Vitest 实例需要当前的测试模式。它可以是以下之一：

- 运行运行时测试时为 `test`
- 运行基准测试时为 `benchmark`
- 运行类型测试时为 `typecheck`

### 模式

#### test

测试模式仅会调用 `test` 或 `it` 中的函数，并在遇到 `bench` 时抛出错误。此模式使用配置中的 `include` 和 `exclude` 选项查找测试文件。

#### benchmark

基准测试模式会调用 `bench` 函数，并在遇到 `test` 或 `it` 时抛出错误。此模式使用配置中的 `benchmark.include` 和 `benchmark.exclude` 选项查找基准测试文件。

#### typecheck

类型检查模式不会*运行*测试。它仅分析类型并提供摘要信息。此模式使用配置中的 `typecheck.include` 和 `typecheck.exclude` 选项查找要分析的文件。

### start

你可以使用 `start` 方法运行测试或者基准测试。你还可以传递一个字符串数组以筛选测试文件。


### `provide`

<<<<<<< HEAD
Vitest 暴露了`provide`方法，它是`vitest.getCoreWorkspaceProject().provide`的简写。使用该方法，您可以从主线程向测试传递值。所有值在存储前都会通过 `structuredClone`进行检查，但值本身不会被克隆。
=======
Vitest exposes `provide` method which is a shorthand for `vitest.getRootTestProject().provide`. With this method you can pass down values from the main thread to tests. All values are checked with `structuredClone` before they are stored, but the values themselves are not cloned.
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

要在测试中接收值，需要从 `vitest` entrypont 导入 `inject` 方法：

```ts
import { inject } from 'vitest'
const port = inject('wsPort') // 3000
```

为了提高类型安全性，我们鼓励您增强 `ProvidedContext` 的类型：

```ts
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test', {
  watch: false,
})
vitest.provide('wsPort', 3000)

declare module 'vitest' {
  export interface ProvidedContext {
    wsPort: number
  }
}
```

::: warning
<<<<<<< HEAD
从技术上讲，`provide`是`WorkspaceProject`的一个方法，因此仅限于特定的项目。不过，所有项目都继承了核心项目的值，这使得 `vitest.provide` 成为向测试传递值的通用方法。
:::

::: tip
在不想使用公共 API 的情况下，[全局设置文件](/config/#globalsetup) 也可以使用此方法：
=======
Technically, `provide` is a method of [`TestProject`](#testproject), so it is limited to the specific project. However, all projects inherit the values from the core project which makes `vitest.provide` universal way of passing down values to tests.
:::

::: tip
This method is also available to [global setup files](/config/#globalsetup) for cases where you cannot use the public API:
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

```js
export default function setup({ provide }) {
  provide('wsPort', 3000)
}
```
:::

## TestProject <Version>2.2.0</Version>

- **Alias**: `WorkspaceProject` before 2.2.0

### name

The name is a unique string assigned by the user or interpreted by Vitest. If user did not provide a name, Vitest tries to load a `package.json` in the root of the project and takes the `name` property from there. If there is no `package.json`, Vitest uses the name of the folder by default. Inline projects use numbers as the name (converted to string).

::: code-group
```ts [node.js]
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test')
vitest.projects.map(p => p.name) === [
  '@pkg/server',
  'utils',
  '2',
  'custom'
]
```
```ts [vitest.workspace.js]
export default [
  './packages/server', // has package.json with "@pkg/server"
  './utils', // doesn't have a package.json file
  {
    // doesn't customize the name
    test: {
      pool: 'threads',
    },
  },
  {
    // customized the name
    test: {
      name: 'custom',
    },
  },
]
```
:::

### vitest

`vitest` references the global [`vitest`](#vitest) process.

### serializedConfig

This is the test config that all tests will receive. Vitest [serializes config](https://github.com/vitest-dev/vitest/blob/main/packages/vitest/src/node/config/serializeConfig.ts) manually by removing all functions and properties that are not possible to serialize. Since this value is available in both tests and node, it is exported from the main entry point.

```ts
import type { SerializedConfig } from 'vitest'

const config: SerializedConfig = vitest.projects[0].serializedConfig
```

### globalConfig

The test config that `vitest` was initialized with. If this is the root project, `globalConfig` and `config` will reference the same object. This config is useful for values that cannot be set on the project level, like `coverage` or `reporters`.

```ts
import type { ResolvedConfig } from 'vitest/node'

vitest.config === vitest.projects[0].globalConfig
```

### config

This is the project's resolved test config.

### vite

This is project's `ViteDevServer`. All projects have their own Vite servers.

### browser

This value will be set only if tests are running in the browser. If `browser` is enabled, but tests didn't run yet, this will be `undefined`. If you need to check if the project supports browser tests, use `project.isBrowserSupported()` method.

::: warning
The browser API is even more experimental and doesn't follow SemVer. The browser API will be standardized separately from the rest of the APIs.
:::

### provide

A way to provide custom values to tests in addition to [`config.provide`](/config/#provide) field. All values are validated with [`structuredClone`](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone) before they are stored, but the values on `providedContext` themselves are not cloned.

::: code-group
```ts [node.js]
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test')
const project = vitest.projects.find(p => p.name === 'custom')
project.provide('key', 'value')
await vitest.start()
```
```ts [test.spec.js]
import { inject } from 'vitest'
const value = inject('key')
```
:::

The values can be provided dynamicaly. Provided value in tests will be updated on their next run.

### getProvidedContext

This returns the context object. Every project also inherits the global context set by `vitest.provide`.

```ts
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test')
vitest.provide('global', true)
const project = vitest.projects.find(p => p.name === 'custom')
project.provide('key', 'value')

// { global: true, key: 'value' }
const context = project.getProvidedContext()
```

::: tip
Project context values will always override global ones.
:::

### createSpecification

Create a test specification that can be used in `vitest.runFiles`. Specification scopes the test file to a specific `project` and `pool` (optionally).

```ts
import { createVitest } from 'vitest/node'
import { resolve } from 'node:path/posix'

const vitest = await createVitest('test')
const project = vitest.projects[0]
const specification = project.createSpecification(
  resolve('./basic.test.ts'),
  'threads', // optional override
)
await vitest.runFiles([specification], true)
```

::: warning
`createSpecification` expects an absolute file path. It doesn't resolve the file or check that it exists on the file system.
:::

### isRootProject

Checks if the current project is the root project. You can also get the root project by calling `vitest.getRootTestProject()`.

The root project generally doesn't run any tests and is not included in `vitest.projects` unless the user explicitly includes the root config in their workspace.

The primary goal of the root project is to setup the global config. In fact, `rootProject.config` references `rootProject.globalConfig` and `vitest.config` directly.

### globTestFiles

Globs all test files. This function returns an object with regular tests and typecheck tests:

```ts
interface GlobReturn {
  /**
   * Test files that match the filters.
   */
  testFiles: string[]
  /**
   * Typecheck test files that match the filters. This will be empty unless `typecheck.enabled` is `true`.
   */
  typecheckTestFiles: string[]
}
```

::: tip
Vitest uses [fast-glob](https://www.npmjs.com/package/fast-glob) to find test files. `test.dir`, `test.root`, `root` or `process.cwd()` define the `cwd` option.

This method looks at several config options:

- `test.include`, `test.exclude` to find regular test files
- `test.includeSource`, `test.exclude` to find in-source tests
- `test.typecheck.include`, `test.typecheck.exclude` to find typecheck tests
:::

### matchesTestGlob

This method checks if the file is a regular test file. It uses the same config properties that `globTestFiles` uses for validation.

This method also accepts a second parameter, which is the source code. This is used to validate if the file is an in-source test. If you are calling this method several times for several projects it is recommended to read the file once and pass it down directly.

```ts
import { createVitest } from 'vitest/node'
import { resolve } from 'node:path/posix'

const vitest = await createVitest('test')
const project = vitest.projects[0]

project.matchesTestGlob(resolve('./basic.test.ts')) // true
project.matchesTestGlob(resolve('./basic.ts')) // false
project.matchesTestGlob(resolve('./basic.ts'), `
if (import.meta.vitest) {
  // ...
}
`) // true if `includeSource` is set
```

### close

Closes the project and all associated resources. This can only be called once; the closing promise is cached until the server restarts. If the resources are needed again, create a new project.

In detail, this method closes the Vite server, stops the typechecker service, closes the browser if it's running, deletes the temporary directory that holds the source code, and resets the provided context.
