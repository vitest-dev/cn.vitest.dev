# 运行器 API

::: warning 注意
这是高级 API。如果你只需要[运行测试](/guide/)，你可能不需要这个。它主要被库的作者使用。
:::

你可以在你的配置文件中使用 `runner` 选项指定你的测试运行器的路径。这个文件应该有一个默认的导出，其中包含一个实现这些方法的类：

```ts
export interface VitestRunner {
  /**
   * 这是在实际收集和运行测试之前被调用的第一件事情。
   */
  onBeforeCollect?: (paths: string[]) => unknown
  /**
   * 这是在收集测试后、"onBeforeRun" 之前被调用的。
   */
  onCollected?: (files: File[]) => unknown

  /**
   * 当测试运行程序应该取消下一次测试运行时调用。
   * 运行程序应该监听此方法，并在“onBeforeRunSuite”和“onBeforeRunTest”中将测试和套件标记为跳过。
   */
  onCancel?: (reason: CancelReason) => unknown

  /**
   * 在运行单个测试之前调用。此时还没有“result”。
   */
  onBeforeRunTask?: (test: TaskPopulated) => unknown
  /**
   * 这是在实际运行测试函数之前被调用的。
   * 此时已经有了带有 "state" 和 "startTime" 属性的 "result" 对象。
   */
  onBeforeTryTask?: (
    test: TaskPopulated,
    options: { retry: number, repeats: number }
  ) => unknown
  /**
   * 这是在结果和状态都被设置之后被调用的。
   */
  onAfterRunTask?: (test: TaskPopulated) => unknown
  /**
   * 这是在运行测试函数后立即被调用的。此时还没有新的状态。
   * 如果测试函数抛出异常，将不会调用此方法。
   */
  onAfterTryTask?: (
    test: TaskPopulated,
    options: { retry: number, repeats: number }
  ) => unknown

  /**
   * 这是在运行单个测试套件之前被调用的，此时还没有测试结果。
   */
  onBeforeRunSuite?: (suite: Suite) => unknown
  /**
   * 这是在运行单个测试套件之后被调用的，此时已经有了状态和测试结果。
   */
  onAfterRunSuite?: (suite: Suite) => unknown

  /**
   * 如果定义了这个方法，它将会替代 Vitest 常规的测试套件分割和处理方式。
   * 但 "before" 和 "after" 钩子函数仍然会被执行。
   */
  runSuite?: (suite: Suite) => Promise<void>
  /**
   * 如果定义了这个方法，它将会替代 Vitest 常规的测试处理方式。
   * 如果你有自定义的测试函数，这个方法就很有用。
   * 但 "before" 和 "after" 钩子函数仍然会被执行。
   */
  runTask?: (test: TaskPopulated) => Promise<void>

  /**
   * 当一个任务被更新时被调用。与报告器中的 "onTaskUpdate" 方法相同。
   * 但该方法在同一个线程中运行，与测试运行在同一个线程中。
   */
  onTaskUpdate?: (task: [string, TaskResult | undefined][]) => Promise<void>

  /**
   * 这是在运行收集的所有测试之前被调用的。
   */
  onBeforeRunFiles?: (files: File[]) => unknown
  /**
   * 这是在运行收集的所有测试后立即被调用的。
   */
  onAfterRunFiles?: (files: File[]) => unknown
  /**
   * 这个方法被用于 "test" 和 "custom" 处理程序。
   * 你可以在 "setupFiles" 中使用 "beforeAll" 来定义自定义上下文，而不是使用 runner。
   * 更多信息请参考：https://vitest.dev/advanced/runner.html#your-task-function
   */
  extendTaskContext?: <T extends Test | Custom>(
    context: TaskContext<T>
  ) => TaskContext<T>
  /**
   * 当导入某些文件时被调用。在收集测试和导入设置文件时都可能会被调用。.
   */
  importFile: (filepath: string, source: VitestRunnerImportSource) => unknown
  /**
   * 公开可用的配置.
   */
  config: VitestRunnerConfig
}
```

当初始化这个类时，Vitest 会传递 Vitest 配置，你应该将它作为一个 `config` 属性暴露出来。

::: warning 注意
Vitest 还会将 `ViteNodeRunner` 的实例作为 `__vitest_executor` 属性注入。你可以使用它来处理 `importFile` 方法中的文件（这是 `TestRunner` 和 `BenchmarkRunner` 的默认行为）。

`ViteNodeRunner` 暴露了 `executeId` 方法，用于在适用于 Vite 的环境中导入测试文件。这意味着它将在运行时解析导入并转换文件内容，以便 Node 能够理解它。
:::

::: tip 提示
快照支持和其他功能是依赖于测试运行器的。如果你想保留这些功能，可以从 `vitest/runners` 导入 `VitestTestRunner` 并将你的测试运行器继承该类。它还暴露了 `BenchmarkNodeRunner`，如果你想扩展基准测试功能的话也可以继承它。
:::

## 你的任务函数

你可以通过扩展 `Vitest` 的任务系统来添加你自己的任务。一个任务是一个对象，是套件的一部分。它会自动通过 `suite.task` 方法添加到当前套件中：

```js [custom.js]
import { createTaskCollector, getCurrentSuite, setFn } from 'vitest/suite'

export { afterAll, beforeAll, describe } from 'vitest'

// 当 Vitest 收集任务时，将调用此函数
// createTaskCollector 只提供了所有的 "todo"/"each"/... 支持，你不必使用它
// 要支持自定义任务，你只需要调用 "getCurrentSuite().task()"
export const myCustomTask = createTaskCollector(function (name, fn, timeout) {
  getCurrentSuite().task(name, {
    ...this, // so "todo"/"skip" is tracked correctly
    meta: {
      customPropertyToDifferentiateTask: true,
    },
    handler: fn,
    timeout,
  })
})
```

```js [tasks.test.js]
import { afterAll, beforeAll, describe, myCustomTask } from './custom.js'
import { gardener } from './gardener.js'

describe('take care of the garden', () => {
  beforeAll(() => {
    gardener.putWorkingClothes()
  })

  myCustomTask('weed the grass', () => {
    gardener.weedTheGrass()
  })
  myCustomTask.todo('mow the lawn', () => {
    gardener.mowerTheLawn()
  })
  myCustomTask('water flowers', () => {
    gardener.waterFlowers()
  })

  afterAll(() => {
    gardener.goHome()
  })
})
```

```bash
vitest ./garden/tasks.test.js
```

::: warning 注意
如果你没有定义自定义运行器，也没有定义 `runTest` 方法，Vitest 将会尝试自动获取任务。如果你没有使用 `setFn` 添加一个函数，这个过程会失败。
:::

::: tip 提示
自定义任务系统支持钩子和上下文。如果你想支持属性链式调用（如 `only`、`skip` 和你自己的定制属性），你可以从 `vitest/suite` 导入 `createChainable` 并用它包装你的函数。如果你决定这样做，你需要将 `custom` 作为 `custom.call(this)` 来调用。
:::
