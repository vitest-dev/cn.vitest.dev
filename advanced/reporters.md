# 扩展默认报告器 (Extending Reporters)

::: warning
这是一个高级 API。如果你只是想配置内置报告器，请阅读 ["Reporters"](/guide/reporters) 指南。
:::

你可以从 `vitest/reporters` 导入报告器并扩展它们来创建自定义报告器。

## 扩展内置报告器 (Extending Built-in Reporters)

一般来说，你不需要从头开始创建报告器。`vitest` 附带了几个可以扩展的默认报告程序。

```ts
import { DefaultReporter } from 'vitest/reporters'

export default class MyDefaultReporter extends DefaultReporter {
  // do something
}
```

当然，你可以从头开始创建报告器。只需扩展 `BaseReporter` 类并实现你需要的方法即可。

这是自定义报告器的示例：

```ts [custom-reporter.js]
import { BaseReporter } from 'vitest/reporters'

export default class CustomReporter extends BaseReporter {
  onCollected() {
    const files = this.ctx.state.getFiles(this.watchFilters)
    this.reportTestSummary(files)
  }
}
```

或者实现 `Reporter` 接口：

```ts [custom-reporter.js]
import type { Reporter } from 'vitest/node'

export default class CustomReporter implements Reporter {
  onCollected() {
    // print something
  }
}
```

然后你可以在 `vitest.config.ts` 文件中使用自定义报告器：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import CustomReporter from './custom-reporter.js'

export default defineConfig({
  test: {
    reporters: [new CustomReporter()],
  },
})
```

## 报告的任务(Reported Tasks)

建议使用 Reported Tasks API，而不是使用报告器接收到的任务。

你可以通过调用 `vitest.state.getReportedEntity(runnerTask)` 访问此 API：

```ts twoslash
<<<<<<< HEAD
// @noErrors
import type { Vitest } from 'vitest/node'
import type { RunnerTestFile } from 'vitest'
import type { Reporter, TestModule } from 'vitest/reporters'
=======
import type { Reporter, RunnerTestFile, TestModule, Vitest } from 'vitest/node'
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d

class MyReporter implements Reporter {
  private vitest!: Vitest

  onInit(vitest: Vitest) {
    this.vitest = vitest
  }

  onFinished(files: RunnerTestFile[]) {
    for (const file of files) {
      // note that the old task implementation uses "file" instead of "module"
      const testModule = this.vitest.state.getReportedEntity(file) as TestModule
      for (const task of testModule.children) {
        //                          ^?
        console.log('finished', task.type, task.fullName)
      }
    }
  }
}
```

## 导出报告器 (Exported Reporters)

`vitest` 附带了一些[内置报告器](/guide/reporters)，你可以开箱即用。

### 内置报告器:

1. `BasicReporter`
1. `DefaultReporter`
1. `DotReporter`
1. `JsonReporter`
1. `VerboseReporter`
1. `TapReporter`
1. `JUnitReporter`
1. `TapFlatReporter`
1. `HangingProcessReporter`

### 基础抽象报告器:

1. `BaseReporter`

### 接口报告器:

1. `Reporter`
