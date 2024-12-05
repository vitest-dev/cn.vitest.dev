---
title: 测试覆盖率 | 指南
---

# 测试覆盖率

Vitest 通过 [`v8`](https://v8.dev/blog/javascript-code-coverage) 支持原生代码覆盖率，通过 [`istanbul`](https://istanbul.js.org/) 支持检测代码覆盖率。

## 测试覆盖率提供者


`v8` 和 `istanbul` 的支持都是可选的。 默认情况下，启用 `v8`。

你可以通过将 `test.coverage.provider` 设置为 `v8` 或 `istanbul` 来选择覆盖工具：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'istanbul', // or 'v8'
    },
  },
})
```

当你启动 Vitest 进程时，它会提示你自动安装相应的支持包。

或者，如果你更喜欢手动安装它们：

::: code-group
```bash [v8]
npm i -D @vitest/coverage-v8
```
```bash [istanbul]
npm i -D @vitest/coverage-istanbul
```
:::

## 覆盖率配置

:::tip
建议始终在配置文件中定义 [`coverage.include`](https://cn.vitest.dev/config/#coverage-include)。
这有助于 Vitest 减少 [`coverage.all`](https://cn.vitest.dev/config/#coverage-all) 选择的文件数量。
:::

要在启用的情况下进行测试，你可以在 CLI 中传递 `--coverage` 标志。
默认情况下, 将使用 `['text', 'html', 'clover', 'json']` 作为测试报告器。

```json [package.json]
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

要对其进行配置，需要在配置文件中设置 `test.coverage` 选项：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      reporter: ['text', 'json', 'html'],
    },
  },
})
```

## 自定义覆盖率的报告器

我们可以通过在 `test.coverage.reporter` 中传递软件包名称或绝对路径来使用自定义覆盖报告器：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      reporter: [
        // Specify reporter using name of the NPM package
        ['@vitest/custom-coverage-reporter', { someOption: true }],

        // Specify reporter using local path
        '/absolute/path/to/custom-reporter.cjs',
      ],
    },
  },
})
```

自定义报告器由 Istanbul 加载，必须与其报告器接口相匹配。查看 [built-in reporters' implementation](https://github.com/istanbuljs/istanbuljs/tree/master/packages/istanbul-reports/lib) 了解更多详情。

```js [custom-reporter.cjs]
const { ReportBase } = require('istanbul-lib-report')

module.exports = class CustomReporter extends ReportBase {
  constructor(opts) {
    super()

    // Options passed from configuration are available here
    this.file = opts.file
  }

  onStart(root, context) {
    this.contentWriter = context.writer.writeFile(this.file)
    this.contentWriter.println('Start of custom coverage report')
  }

  onEnd() {
    this.contentWriter.println('End of custom coverage report')
    this.contentWriter.close()
  }
}
```

## 自定义覆盖率的提供者

也可以通过将 `'custom'` 传递给 `test.coverage.provider` 来配置你的自定义覆盖率提供者：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'custom',
      customProviderModule: 'my-custom-coverage-provider',
    },
  },
})
```

自定义覆盖率提供者需要一个 `customProviderModule` 选项，它是一个模块名称或从中加载 `CoverageProviderModule` 的路径。 它必须将实现 `CoverageProviderModule` 的对象导出为默认导出：

```ts [my-custom-coverage-provider.ts]
import type {
  CoverageProvider,
  CoverageProviderModule,
  ResolvedCoverageOptions,
  Vitest
} from 'vitest'

const CustomCoverageProviderModule: CoverageProviderModule = {
  getProvider(): CoverageProvider {
    return new CustomCoverageProvider()
  },

  // Implements rest of the CoverageProviderModule ...
}

class CustomCoverageProvider implements CoverageProvider {
  name = 'custom-coverage-provider'
  options!: ResolvedCoverageOptions

  initialize(ctx: Vitest) {
    this.options = ctx.config.coverage
  }

  // Implements rest of the CoverageProvider ...
}

export default CustomCoverageProviderModule
```

请参阅类型定义查看有关详细信息。

## 更改默认覆盖文件夹位置

运行覆盖率报告时，会在项目的根目录中创建一个 `coverage` 文件夹。 如果你想将它移动到不同的目录，请使用 `vite.config.js` 文件中的 `test.coverage.reportsDirectory` 属性。

```js [vitest.config.js]
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    coverage: {
      reportsDirectory: './tests/unit/coverage',
    },
  },
})
```

## 代码忽略

两个覆盖率提供商都有自己的方法来忽略覆盖率报告中的代码：

- [`v8`](https://github.com/istanbuljs/v8-to-istanbul#ignoring-uncovered-lines)
- [`ìstanbul`](https://github.com/istanbuljs/nyc#parsing-hints-ignoring-lines)

使用 TypeScript 时，源代码使用 `esbuild` 进行转译，这会从源代码中删除所有注释([esbuild#516](https://github.com/evanw/esbuild/issues/516))。
被视为[合法注释](https://esbuild.github.io/api/#legal-comments)的注释将被保留。

对于 `istanbul` 测试提供者，你可以在忽略提示中包含 `@preserve` 关键字。
请注意，这些忽略提示现在也可能包含在最终的产品构建中。

```diff
-/* istanbul ignore if */
+/* istanbul ignore if -- @preserve */
if (condition) {
```

不幸的是，目前这在 `v8` 中不起作用。你通常可以在 TypeScript 使用 `v8 ignore` 注释：

<!-- eslint-skip -->

```ts
/* v8 ignore next 3 */
if (condition) {
```

## 其他选项

要查看有关覆盖率的所有可配置选项，请参见 [覆盖率配置参考](https://cn.vitest.dev/config/#coverage)。

## Vitest UI

你可以在 [Vitest UI](/guide/ui) 中查看你的覆盖率报告。

Vitest UI 将在显式启用覆盖率报告且存在 html 覆盖率报告器的情况下启用覆盖率报告，否则将不可用：

- 在配置中启用 `coverage.enabled=true` 或使用 `--coverage.enabled=true` 标志运行 Vitest
- 在 `coverage.reporter` 列表中添加 `html`：也可以启用 `subdir` 选项，将覆盖率报告放到子目录中

<img alt="html coverage activation in Vitest UI" img-light src="/vitest-ui-show-coverage-light.png">
<img alt="html coverage activation in Vitest UI" img-dark src="/vitest-ui-show-coverage-dark.png">

<img alt="html coverage in Vitest UI" img-light src="/ui-coverage-1-light.png">
<img alt="html coverage in Vitest UI" img-dark src="/ui-coverage-1-dark.png">
