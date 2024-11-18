# 扩展默认报告器 (Extending Reporters)

你可以从 `vitest/reporters` 导入报告器并扩展它们以创建你的自定义报告器。

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

```ts
// ./custom-reporter.js
import { BaseReporter } from 'vitest/reporters'

export default class CustomReporter extends BaseReporter {
  onCollected() {
    const files = this.ctx.state.getFiles(this.watchFilters)
    this.reportTestSummary(files)
  }
}
```

或者实现 `Reporter` 接口：

```ts
// ./custom-reporter.js
import { Reporter } from 'vitest/reporters'

export default class CustomReporter implements Reporter {
  onCollected() {
    // print something
  }
}
```

然后你可以在 `vitest.config.ts` 文件中使用自定义报告器：

```ts
import { defineConfig } from 'vitest/config'
import CustomReporter from './custom-reporter.js'

export default defineConfig({
  test: {
    reporters: [new CustomReporter()],
  },
})
```

## 报告的任务(Reported Tasks)

::: warning
这是一个试验性 API。破坏性更改可能不会跟进 SemVer。使用时请使用 Vitest 的版本。

你可以通过调用 `vitest.state.getReportedEntity(runnerTask)` 访问此 API：

```ts twoslash
// @noErrors
import type { Vitest } from 'vitest/node'
import type { RunnerTestFile } from 'vitest'
import type { Reporter, TestModule } from 'vitest/reporters'

class MyReporter implements Reporter {
  ctx!: Vitest

  onInit(ctx: Vitest) {
    this.ctx = ctx
  }

  onFinished(files: RunnerTestFile[]) {
    for (const fileTask of files) {
      // note that the old task implementation uses "file" instead of "module"
      const testModule = this.ctx.state.getReportedEntity(fileTask) as TestModule
      for (const task of testModule.children) {
        //                          ^?
        console.log('finished', task.type, task.fullName)
      }
    }
  }
}
```
<<<<<<< HEAD

我们计划在 Vitest 2.1 中稳定该应用程序接口。
=======
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9
:::

### TestCase

`TestCase` 代表一次测试.

```ts
declare class TestCase {
  readonly type = 'test' | 'custom'
  /**
   * Task instance.
   * @experimental Public task API 是试验性的，并不遵循 semver。
   */
  readonly task: RunnerTestCase | RunnerCustomCase
  /**
   * 与测试相关的项目。
   */
  readonly project: TestProject
  /**
   * 直接引用定义测试的测试文件。
   */
  readonly module: TestModule
  /**
   * 测试名称。
   */
  readonly name: string
  /**
   * 测试的全名，包括用 `>` 分隔的所有父套件
   */
  readonly fullName: string
  /**
   * 唯一标识符。
   * 该 ID 是确定的，多次运行时同一测试的 ID 将是相同的。
   * ID 基于项目名称、文件路径和测试位置。
   */
  readonly id: string
  /**
   * 文件中定义测试的位置。
   * 只有在配置中启用  `includeTaskLocation` 时，才会收集位置信息。
   */
  readonly location: { line: number, column: number } | undefined
  /**
   * 如果测试是在文件中直接调用的，则父套件将是文件。
   */
  readonly parent: TestSuite | TestModule
  /**
   * 启动测试时使用的选项。
   */
  readonly options: TaskOptions
  /**
   * 检查测试是否没有失败。
   * 如果测试尚未完成或被跳过，则返回 `true`。
   */
  ok(): boolean
  /**
   * 执行测试时附加到测试中的自定义元数据。
   */
  meta(): TaskMeta
  /**
   * 测试结果。如果测试尚未完成或刚刚收集，则将为`undefined`。
   */
  result(): TestResult | undefined
  /**
   * 有关测试的有用信息，如持续时间、内存使用情况等。
   */
  diagnostic(): TestDiagnostic | undefined
}

export type TestResult = TestResultPassed | TestResultFailed | TestResultSkipped

export interface TestResultPassed {
  /**
   * 测试已成功通过。
   */
  state: 'passed'
  /**
   * 测试执行过程中出现的错误。
   *
   * **Note**: 如果测试重试成功，仍会报告错误。
   */
  errors: TestError[] | undefined
}

export interface TestResultFailed {
  /**
   * 测试执行失败。
   */
  state: 'failed'
  /**
   * 测试执行过程中出现的错误。
   */
  errors: TestError[]
}

export interface TestResultSkipped {
  /**
   * 使用`only`、`skip`或`todo`标志跳过测试。点击并应用。
   * 你可以在 `mode` 选项中看到使用的是哪一种。
   */
  state: 'skipped'
  /**
   * 跳过的测试没有错误。
   */
  errors: undefined
}

export interface TestDiagnostic {
  /**
   * If the duration of the test is above `slowTestThreshold`.
   */
  slow: boolean
  /**
   * 测试使用的内存量（字节）。
   * 只有使用 `logHeapUsage` 标志执行测试时，该值才可用。
   */
  heap: number | undefined
  /**
   * 执行测试所需的时间（毫秒）。
   */
  duration: number
  /**
   * 测试开始的时间（毫秒）。
   */
  startTime: number
  /**
   * 测试重试的次数。
   */
  retryCount: number
  /**
   * 重复测试的次数，由 `repeats` 选项设置。
   * 如果测试在重复过程中失败，且未配置 `retry`，则该值可以更小。
   */
  repeatCount: number
  /**
   * 如果第二次重试时测试通过。
   */
  flaky: boolean
}
```

### TestSuite

`TestSuite` 表示包含测试和其他套件的单一套件。

```ts
declare class TestSuite {
  readonly type = 'suite'
  /**
   * Task instance.
   * @experimental Public task API 是实验性的，并不遵循 semver。
   */
  readonly task: RunnerTestSuite
  /**
   * 与测试相关的项目。
   */
  readonly project: TestProject
  /**
   * 直接引用定义套件的测试文件。
   */
  readonly module: TestModule
  /**
   * Name of the suite.
   */
  readonly name: string
  /**
   * 套件的全名，包括用 `>` 分隔的所有父套件。
   */
  readonly fullName: string
  /**
   * 唯一标识符。
   * 该 ID 是确定的，多次运行时同一测试的 ID 将是相同的。
   * ID 基于项目名称、文件路径和测试位置。
   */
  readonly id: string
  /**
   * 文件中定义套件的位置。
   * 只有在配置中启用 `includeTaskLocation` 时，才会收集位置信息。
   */
  readonly location: { line: number, column: number } | undefined
  /**
   * 套件和属于该套件的测试的集合。
   */
  readonly children: TaskCollection
  /**
   * 启动套件时使用的选项。
   */
  readonly options: TaskOptions
}
```

### TestModule

`TestModule` 表示包含套件和测试的单个文件。

```ts
declare class TestModule extends SuiteImplementation {
  readonly type = 'module'
  /**
   * Task instance.
   * @experimental Public task API 是实验性的，并不遵循 semver。
   */
  readonly task: RunnerTestFile
  /**
   * 属于该文件的套件和测试集合。
   */
  readonly children: TestCollection
  /**
   * 这通常是一个绝对的 Unix 文件路径。
   * 如果文件不在磁盘上，它可以是一个虚拟 ID。
   * 该值对应于 Vite 的 `ModuleGraph` id。
   */
  readonly moduleId: string
  /**
   * 有关文件的有用信息，如持续时间、内存使用情况等。
   * 如果文件尚未执行，所有诊断值都将返回 `0`。
   */
  diagnostic(): ModuleDiagnostic
}

export interface ModuleDiagnostic {
  /**
   * 导入和启动环境所需的时间。
   */
  environmentSetupDuration: number
  /**
   * Vitest 设置测试工具（运行程序、模拟等）所需的时间。
   */
  prepareDuration: number
  /**
   * 导入测试文件所需的时间。
   * 这包括导入文件中的所有内容和执行套件回调。
   */
  collectDuration: number
  /**
   * 导入设置文件所需的时间。
   */
  setupDuration: number
  /**
   * 文件中所有测试和钩子的累计持续时间。
   */
  duration: number
}
```

### TestCollection

`TestCollection` 表示套件和测试的集合。它还提供了迭代自身的有用方法。

```ts
declare class TestCollection {
  /**
   * 返回数组中特定索引处的测试或套件。
   */
  at(index: number): TestCase | TestSuite | undefined
  /**
   * 集合中测试和套件的数量。
   */
  size: number
  /**
   * 以数组形式返回集合，以便于操作。
   */
  array(): (TestCase | TestSuite)[]
  /**
   * 过滤属于此集合及其子集合的所有套件。
   */
  allSuites(): IterableIterator<TestSuite>
  /**
   * 过滤属于此集合及其子集合的所有测试。
   */
  allTests(state?: TestResult['state'] | 'running'): IterableIterator<TestCase>
  /**
   * 只筛选属于该集合的测试。
   */
  tests(state?: TestResult['state'] | 'running'): IterableIterator<TestCase>
  /**
   * 仅筛选属于该系列的套件。
   */
  suites(): IterableIterator<TestSuite>;
  [Symbol.iterator](): IterableIterator<TestSuite | TestCase>
}
```

例如，你可以通过调用 `testFile.children.allTests()` 遍历文件中的所有测试：

```ts
function onFileCollected(testModule: TestModule): void {
  console.log('collecting tests in', testModule.moduleId)

  // iterate over all tests and suites in the module
  for (const task of testModule.children.allTests()) {
    console.log('collected', task.type, task.fullName)
  }
}
```

<<<<<<< HEAD
### TestProject

`TestProject` 是与文件相关联的项目。该文件中的每个测试和套件都将引用同一个项目。

项目可用于获取配置或提供的上下文。

```ts
declare class TestProject {
  /**
   * The global vitest instance.
   * @experimental The public Vitest API 是实验性的，并不遵循 semver。
   */
  readonly vitest: Vitest
  /**
   * 与该测试项目相关联的工作区项目。
   * @experimental The public Vitest API 是实验性的，并不遵循 semver。
   */
  readonly workspaceProject: WorkspaceProject
  /**
   * Vite 的开发服务器实例。每个工作区项目都有自己的服务器。
   */
  readonly vite: ViteDevServer
  /**
   * 已解决项目配置问题。
   */
  readonly config: ResolvedProjectConfig
  /**
   * 已解决的全局配置问题。如果没有工作区项目，则与 `config`相同。
   */
  readonly globalConfig: ResolvedConfig
  /**
   * 序列化的项目配置。这就是测试会收到的配置。
   */
  get serializedConfig(): SerializedConfig
  /**
   * 项目名称，如果未设置，则为空字符串。
   */
  name(): string
  /**
   * 为项目提供的自定义上下文。
   */
  context(): ProvidedContext
  /**
   * 为项目提供自定义的可序列化上下文。测试运行后，该上下文将可供测试使用。
   */
  provide<T extends keyof ProvidedContext & string>(key: T, value: ProvidedContext[T]): void
}
```

## 导出报告器 (Exported Reporters)
=======
## Exported Reporters
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

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
