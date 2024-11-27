---
title: 常见错误 | 指南
---

# 常见错误

## Cannot find module './relative-path'

如果你收到一个 **module cannot be found** 的报错，则可能意味着几种不同情况：

- 1.你拼错了路径。确保路径正确。
- 2.你可能依赖于 `tsconfig.json` 中的 `baseUrl`。默认情况下，Vite 不考虑 `tsconfig.json`，因此如果你依赖此行为，你可能需要自己安装 [`vite-tsconfig-paths`](https://www.npmjs.com/package/vite-tsconfig-paths) 。

```ts
import { defineConfig } from 'vitest/config'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths()],
})
```

或者重写你的路径，使它不是相对于 root。

```diff
- import helpers from 'src/helpers'
+ import helpers from '../src/helpers'
```

- 3. 确保你没有使用相对路径的 [别名](/config/#alias)。Vite 将它们视为相对于导入所在的文件而不是根目录。

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    alias: {
      '@/': './src/', // [!code --]
      '@/': new URL('./src/', import.meta.url).pathname, // [!code ++]
    },
  },
})
```

## Cannot mock "./mocked-file.js" because it is already loaded

当对已加载的模块调用 `vi.mock` 方法时，会发生此错误。Vitest 抛出此错误，因为此调用没有效果，因为首选缓存模块。

请记住，`vi.mock` 总是挂起的 - 这意味着模块在测试文件开始执行之前就已经加载了 - 很可能是在安装文件中。要修复此错误，请删除导入或清除安装文件末尾的缓存 - 请注意，在这种情况下，安装文件和测试文件将引用不同的模块。

```ts [setupFile.js]
import { vi } from 'vitest'
import { sideEffect } from './mocked-file.js'

sideEffect()

vi.resetModules()
```

## Failed to terminate worker

当 NodeJS 的 fetch 与默认的 [`pool: 'threads'`](/config/#threads) 一起使用时，可能会发生此错误。问题可以在 [issue#3077](https://github.com/vitest-dev/vitest/issues/3077) 上进行持续更新。

作为解决方法，我们可以切换到 [`pool: 'forks'`](/config/#forks) 或 [`pool: 'vmForks'`](/config/#vmforks)。

::: code-group

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```

```bash [CLI]
vitest --pool=forks
```

:::

## Segfaults and native code errors

运行 [原生 NodeJS 模块](https://nodejs.org/api/addons.html)在 `pool: 'threads'` 中，可能会遇到来自原生代码的神秘错误。

- `Segmentation fault (core dumped)`
- `thread '<unnamed>' panicked at 'assertion failed`
- `Abort trap: 6`
- `internal error: entered unreachable code`

在这些情况下，原生模块可能不是为多线程安全而构建的。在解决方案中，你可以切换到 `pool: 'forks'`，它在多个 `node:child_process` 而不是多个 `node:worker_threads` 中运行测试用例。

::: code-group

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```

```bash [CLI]
vitest --pool=forks
```

:::
