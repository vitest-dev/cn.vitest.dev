---
title: Vitest UI | 指南
---

# Vitest UI

Vitest 由 Vite 提供能力，在运行测试时有一个开发服务器。这允许 Vitest 提供一个漂亮的 UI 界面来查看并与测试交互。Vitest 的 UI 界面是可选的，你可以通过以下安装：

```bash
npm i -D @vitest/ui
```

接下来，你可以通过传入 `--ui` 参数来启动测试的 UI 界面：

```bash
vitest --ui
```

最后，你可以访问 Vitest UI 界面，通过 <a href="http://localhost:51204/__vitest__/">`http://localhost:51204/__vitest__/`</a>

<img alt="Vitest UI" img-light src="/ui-1-light.png">
<img alt="Vitest UI" img-dark src="/ui-1-dark.png">

UI 也可以用作测试报告器。 在 Vitest 配置中使用 `'html'` 报告器生成 HTML 输出并预览测试结果：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    reporters: ['html'],
  },
<<<<<<< HEAD
}
=======
})
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f
```

你可以在 Vitest UI 中查看覆盖率报告：查看 [Vitest UI 覆盖率](/guide/coverage#vitest-ui) 了解更多详情。

::: warning
如果你仍想在终端中实时查看测试的运行情况，请不要忘记将 `default` 报告器添加到 `reporters` 选项：`['default', 'html']`。
:::

::: tip
要预览你的 HTML 报告，可以使用 [vite preview](https://vitejs.dev/guide/cli.html#vite-preview) 命令：

```sh
npx vite preview --outDir ./html
```

你可以使用 [`outputFile`](/config/#outputfile) 配置选项配置输出。你需要在那里指定 `.html` 路径。例如，`./html/index.html` 是默认值。
:::
