---
title: 源码内联测试 | 指南
---

# 源码内联测试

Vitest 还提供了一种方式，可以运行与你的代码实现放在一起的测试，就像是 [Rust 语言的模块测试一样](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest)。

这允许测试与实现共享相同的闭包，并且能够在不导出的情况下针对私有状态进行测试。同时，它也使开发更加接近反馈循环。

## 指引
::: warning
本指南介绍如何在源代码中编写测试。如果需要在单独的测试文件中编写测试，请参阅["编写测试"指南](/guide/#writing-tests)。
:::

首先，在 `if (import.meta.vitest)` 代码块内写一些测试代码并放在文件的末尾，例如:

<<<<<<< HEAD
```ts
// src/index.ts

// 函数实现
=======
```ts [src/index.ts]
// the implementation
>>>>>>> b629c6141185f4a49b4c443b5cdcfe1a9ddda849
export function add(...args: number[]) {
  return args.reduce((a, b) => a + b, 0)
}

// 源码内的测试套件
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest
  it('add', () => {
    expect(add()).toBe(0)
    expect(add(1)).toBe(1)
    expect(add(1, 2, 3)).toBe(6)
  })
}
```

更新 Vitest 配置文件内的 `includeSource` 以获取到 `src/` 下的文件：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    includeSource: ['src/**/*.{js,ts}'], // [!code ++]
  },
})
```

然后你就可以开始执行测试了!

```bash
$ npx vitest
```

## 生产环境构建

对于生产环境的构建，你需要设置配置文件内的 `define` 选项，让打包器清除无用的代码。例如，在 Vite 中

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    includeSource: ['src/**/*.{js,ts}'],
  },
  define: {
    // [!code ++]
    'import.meta.vitest': 'undefined', // [!code ++]
  }, // [!code ++]
})
```

### 其他的打包器

<details mt4>
<summary text-xl>unbuild</summary>

```ts [build.config.ts]
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig({
  replace: {
    // [!code ++]
    'import.meta.vitest': 'undefined', // [!code ++]
  }, // [!code ++]
  // other options
})
```

了解更多：[unbuild](https://github.com/unjs/unbuild)

</details>

<details my2>
<summary text-xl>Rollup</summary>

```ts [rollup.config.js]
import replace from '@rollup/plugin-replace' // [!code ++]

export default {
  plugins: [
    replace({
      // [!code ++]
      'import.meta.vitest': 'undefined', // [!code ++]
    }), // [!code ++]
  ],
  // other options
}
```

了解更多：[Rollup](https://rollupjs.org/)

</details>

## TypeScript

要获得对 `import.meta.vitest` 的 TypeScript 支持，添加 `vitest/importMeta` 到 `tsconfig.json`:

```json [tsconfig.json]
{
  "compilerOptions": {
    "types": [
      "vitest/importMeta" // [!code ++]
    ]
  }
}
```

完整的示例请参考 [`examples/in-source-test`](https://github.com/vitest-dev/vitest/tree/main/examples/in-source-test)。

## 说明

此功能可用于:

- 小范围的功能或 utils 工具的单元测试
- 原型设计
- 内联断言

对于更复杂的测试，比如组件测试或 E2E 测试，建议**使用单独的测试文件取而代之**。
