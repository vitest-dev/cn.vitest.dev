---
title: 工作空间 | 指南
---

# 工作空间

::: tip Sample Project

[GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/workspace) - [Play Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/workspace?initialPath=__vitest__/)

:::

Vitest 提供了在单个 Vitest 进程中定义多个项目配置的方法。该功能对单核设置尤为有用，但也可用于运行不同配置的测试，如 `resolve.alias`、`plugins` 或 `test.browser` 等。

## 定义工作空间

工作区必须在其根目录中包含一个 `vitest.workspace` 或 `vitest.projects` 文件（位于与我们的根配置文件相同的文件夹中，或者如果不存在，则位于工作目录中）。请注意，`projects` 只是一个别名，不会改变此功能的行为或语义。Vitest 支持此文件的 `ts`、`js` 和 `json` 扩展名。

自Vitest 3起，我们也可以在根配置文件中定义工作区。在这种情况下，如果存在，Vitest将忽略根目录下的 `vitest.workspace` 文件。

::: tip NAMING
:::

工作区是一系列内联配置、文件或引用我们项目的全局模式的列表。例如，如果我们有一个名为 `packages` 的文件夹，其中包含了我们的项目，我们可以直接创建一个工作区文件，或者在根配置中定义一个数组：

:::code-group

```ts [vitest.workspace.ts]
export default ['packages/*']
```
```ts [vitest.config.ts <Version>3.0.0</Version>]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    workspace: ['packages/*'],
  },
})
```
:::

Vitest 会将 `packages` 中的每个文件夹视为一个独立的项目，即使它里面没有配置文件。如果这个全局模式匹配到任何文件，即使文件名中没有 `vitest`，也会被视作 Vitest 的配置文件。

::: warning
<<<<<<< HEAD
除非在工作区配置中明确指定，否则 Vitest 不会将根配置文件 `vitest.config` 视为工作区项目。因此，根配置只会影响全局选项，如 `reporters` 和 `coverage`。
=======
Vitest does not treat the root `vitest.config` file as a workspace project unless it is explicitly specified in the workspace configuration. Consequently, the root configuration will only influence global options such as `reporters` and `coverage`. Note that Vitest will always run certain plugin hooks, like `apply`, `config`, `configResolved` or `configureServer`, specified in the root config file. Vitest also uses the same plugins to execute global setups, workspace files and custom coverage provider.
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d
:::

你还可以使用项目的配置文件引用项目：

:::code-group

```ts [vitest.workspace.ts]
export default ['packages/*/vitest.config.{e2e,unit}.ts']
```
```ts [vitest.config.ts <Version>3.0.0</Version>]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    workspace: ['packages/*/vitest.config.{e2e,unit}.ts'],
  },
})
```
:::

该模式仅包括具有包含 `e2e` 或 `unit` 的 `vitest.config` 文件的项目。这些关键字需要在文件扩展名之前出现。

你也可以使用内联配置定义项目。工作区文件同时支持这两种语法。

:::code-group
```ts [vitest.workspace.ts]
import { defineWorkspace } from 'vitest/config'

// defineWorkspace 会提供一个很好的类型提示开发体验
export default defineWorkspace([
  // matches every folder and file inside the `packages` folder
  'packages/*',
  {
    // 添加 "extends" 将两个配置合并到一起
    extends: './vite.config.js',
    test: {
      include: ['tests/**/*.{browser}.test.{ts,js}'],
      // 在使用内联配置的时候，建议定义一个名称
      name: 'happy-dom',
      environment: 'happy-dom',
    },
  },
  {
    test: {
      include: ['tests/**/*.{node}.test.{ts,js}'],
      name: 'node',
      environment: 'node',
    },
  },
])
```
```ts [vitest.config.ts <Version>3.0.0</Version>]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    workspace: [
      // matches every folder and file inside the `packages` folder
      'packages/*',
      {
        // add "extends: true" to inherit the options from the root config
        extends: true,
        test: {
          include: ['tests/**/*.{browser}.test.{ts,js}'],
          // it is recommended to define a name when using inline configs
          name: 'happy-dom',
          environment: 'happy-dom',
        }
      },
      {
        test: {
          include: ['tests/**/*.{node}.test.{ts,js}'],
          name: 'node',
          environment: 'node',
        }
      }
    ]
  }
})
```
:::

::: warning
所有项目都必须有唯一的名称，否则 Vitest 会出错。如果内联配置中没有提供名称，Vitest 将分配一个数字。对于使用 glob 语法定义的项目配置，Vitest 将默认使用最近的 `package.json` 文件中的 "name" 属性，如果不存在，则使用文件夹名称。
:::

如果我们不使用内联配置，我们可以在根目录创建一个小的 JSON 文件，或者仅仅在根配置中指定它：

:::code-group

```json [vitest.workspace.json]
["packages/*"]
```
```ts [vitest.config.ts <Version>3.0.0</Version>]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    workspace: ['packages/*'],
  },
})
```

:::

工作区项目不支持所有配置属性。为了提高类型安全性，请在项目配置文件中使用 `defineProject` 方法而不是 `defineConfig` 方法：

```ts twoslash [packages/a/vitest.config.ts]
// @errors: 2769
import { defineProject } from 'vitest/config'

export default defineProject({
  test: {
    environment: 'jsdom',
    // "reporters" 在项目配置中是不支持的，
    // 所以会报错
    reporters: ['json'],
  },
})
```

## 运行测试

要在工作区内运行测试，请在根目录 `package.json` 中定义一个脚本：

```json [package.json]
{
  "scripts": {
    "test": "vitest"
  }
}
```

现在可以使用 CLI 运行测试了：

::: code-group

```bash [npm]
npm run test
```

```bash [yarn]
yarn test
```

```bash [pnpm]
pnpm run test
```

```bash [bun]
bun test
```

:::

如果只需在单个项目内运行测试，使用 `--project` CLI 选项：

::: code-group
```bash [npm]
npm run test --project e2e
```
```bash [yarn]
yarn test --project e2e
```
```bash [pnpm]
pnpm run test --project e2e
```
```bash [bun]
bun test --project e2e
```
:::

::: tip
CLI 选项 `--project` 可多次使用，以筛选出多个项目：

::: code-group
```bash [npm]
npm run test --project e2e --project unit
```
```bash [yarn]
yarn test --project e2e --project unit
```
```bash [pnpm]
pnpm run test --project e2e --project unit
```
```bash [bun]
bun test --project e2e --project unit
```
:::

## 配置

<<<<<<< HEAD
没有任何配置选项从根级别的配置文件继承。你可以创建一个共享的配置文件，并将其与项目配置文件合并：
=======
None of the configuration options are inherited from the root-level config file, even if the workspace is defined inside that config and not in a separate `vitest.workspace` file. You can create a shared config file and merge it with the project config yourself:
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d

```ts [packages/a/vitest.config.ts]
import { defineProject, mergeConfig } from 'vitest/config'
import configShared from '../vitest.shared.js'

export default mergeConfig(
  configShared,
  defineProject({
    test: {
      environment: 'jsdom',
    },
  })
)
```

此外，在 `defineWorkspace` 层级，您可以使用 `extends` 选项来继承根级别的配置。所有选项将被合并。

::: code-group
```ts [vitest.workspace.ts]
import { defineWorkspace } from 'vitest/config'

export default defineWorkspace([
  {
    extends: './vitest.config.ts',
    test: {
      name: 'unit',
      include: ['**/*.unit.test.ts'],
    },
  },
  {
    extends: './vitest.config.ts',
    test: {
      name: 'integration',
      include: ['**/*.integration.test.ts'],
    },
  },
])
```
```ts [vitest.config.ts <Version>3.0.0</Version>]
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    pool: 'threads',
    workspace: [
      {
        // will inherit options from this config like plugins and pool
        extends: true,
        test: {
          name: 'unit',
          include: ['**/*.unit.test.ts'],
        },
      },
      {
        // won't inherit any options from this config
        // this is the default behaviour
        extends: false,
        test: {
          name: 'integration',
          include: ['**/*.integration.test.ts'],
        },
      },
    ],
  },
})
```
:::

某些配置选项不允许在项目配置中使用。其中最明显的是：

- `coverage`: 覆盖率是针对整个工作区进行的。
- `reporters`: 仅支持根级别的报告器。
- `resolveSnapshotPath`: 仅支持根级别的解析器。
- 所有其他不影响测试运行器的选项。

::: tip
所有不支持在项目配置中使用的配置选项，在 ["Config"](/config/) 指南中以 <NonProjectOption /> 标记。
:::
