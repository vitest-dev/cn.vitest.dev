# TestCase

`TestCase` 类表示单个测试。此类仅在主线程中可用。如果您正在处理运行时任务，请参阅[“Runner API”](/advanced/runner#tasks)。

`TestCase` 实例始终有一个值为 `test` 的 `type` 属性。您可以使用它来区分不同的任务类型：

```ts
if (task.type === 'test') {
  task // TestCase
}
```

<<<<<<< HEAD
::: warning
我们计划引入一个新的 Reporter API，默认将使用此 API。目前，Reporter API 使用 [runner tasks](/advanced/runner#tasks)，但您仍然可以通过 `vitest.state.getReportedEntity` 方法访问 `TestCase`：

```ts
import type { RunnerTestFile, TestModule, Vitest } from 'vitest/node'

class Reporter {
  private vitest!: Vitest

  onInit(vitest: Vitest) {
    this.vitest = vitest
  }

  onFinished(files: RunnerTestFile[]) {
    for (const file of files) {
      const testModule = this.vitest.getReportedEntity(file) as TestModule
      for (const test of testModule.children.allTests()) {
        console.log(test) // TestCase
      }
    }
  }
}
```
:::

=======
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d
## project

这引用了测试所属的 [`TestProject`](/advanced/api/test-project)。

## module

这是对定义测试的 [`TestModule`](/advanced/api/test-module) 的直接引用。

## name

这是传递给 `test` 函数的测试名称。

```ts
import { test } from 'vitest'

// [!code word:'the validation works correctly']
test('the validation works correctly', () => {
  // ...
})
```

## fullName

包括所有父套件并用 `>` 符号分隔的测试名称。此测试的完整名称为 "the validation logic > the validation works correctly"：

```ts
import { describe, test } from 'vitest'

// [!code word:'the validation works correctly']
// [!code word:'the validation logic']
describe('the validation logic', () => {
  test('the validation works correctly', () => {
    // ...
  })
})
```

## id

这是测试的唯一标识符。此 ID 是确定性的，在多次运行中相同的测试将具有相同的 ID。ID 基于 [project](/advanced/api/test-project) 名称、模块 ID 和测试顺序。

ID 的格式如下：

```
1223128da3_0_0
^^^^^^^^^^ the file hash
           ^ suite index
             ^ test index
```

::: tip
你可以使用 `vitest/node` 中的 `generateFileHash` 函数来生成文件哈希，该函数自 Vitest 3 起可用：

```ts
import { generateFileHash } from 'vitest/node'

const hash = generateFileHash(
  '/file/path.js', // relative path
  undefined, // the project name or `undefined` is not set
)
```
:::

::: danger
不要尝试解析 ID。它可能以连字符开头，例如：`-1223128da3_0_0_0`。
:::

## location

测试在模块中定义的位置。只有在配置中启用了 [`includeTaskLocation`](/config/#includetasklocation) 时才会收集位置信息。请注意，如果使用了 `--reporter=html`、`--ui` 或 `--browser` 参数，此选项会自动启用。

此测试的位置将等于 `{ line: 3, column: 1 }`：

```ts:line-numbers {3}
import { test } from 'vitest'

test('the validation works correctly', () => {
  // ...
})
```

## parent

父级 [suite](/advanced/api/test-suite)。如果测试是直接在 [模块](/advanced/api/test-module) 内调用的，则父级将是模块本身。

## options

```ts
interface TaskOptions {
  readonly each: boolean | undefined
  readonly fails: boolean | undefined
  readonly concurrent: boolean | undefined
  readonly shuffle: boolean | undefined
  readonly retry: number | undefined
  readonly repeats: number | undefined
  readonly mode: 'run' | 'only' | 'skip' | 'todo'
}
```

收集测试时使用的选项。

## ok

```ts
function ok(): boolean
```

检查测试是否未使套件失败。如果测试尚未完成或被跳过，它将返回 `true`。

<<<<<<< HEAD
## skipped

```ts
function skipped(): boolean
```

检查测试是否在收集期间或通过 `ctx.skip()` 动态跳过。

=======
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d
## meta

```ts
function meta(): TaskMeta
```

在测试执行期间附加的自定义元数据。可以通过在测试运行期间将属性赋值给 `ctx.task.meta` 对象来附加元数据：

```ts {3,6}
import { test } from 'vitest'

test('the validation works correctly', ({ task }) => {
  // ...

  task.meta.decorated = false
})
```

如果测试尚未完成运行，元数据将是一个空对象。

## result

```ts
function result(): TestResult
```

<<<<<<< HEAD
测试结果。如果测试在收集期间被跳过、尚未完成或只是被收集，则结果将为 `undefined`。
=======
Test results. If test is not finished yet or was just collected, it will be equal to `TestResultPending`:

```ts
export interface TestResultPending {
  /**
   * The test was collected, but didn't finish running yet.
   */
  readonly state: 'pending'
  /**
   * Pending tests have no errors.
   */
  readonly errors: undefined
}
```
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d

如果测试被跳过，返回值将是 `TestResultSkipped`：

```ts
interface TestResultSkipped {
  /**
   * The test was skipped with `skip` or `todo` flag.
   * You can see which one was used in the `options.mode` option.
   */
  readonly state: 'skipped'
  /**
   * Skipped tests have no errors.
   */
  readonly errors: undefined
  /**
   * A custom note passed down to `ctx.skip(note)`.
   */
  readonly note: string | undefined
}
```

::: tip
如果测试因为其他测试有 `only` 标志而被跳过，则 `options.mode` 将等于 `skip`。
:::

如果测试失败，返回值将是 `TestResultFailed`：

```ts
interface TestResultFailed {
  /**
   * The test failed to execute.
   */
  readonly state: 'failed'
  /**
   * Errors that were thrown during the test execution.
   */
  readonly errors: ReadonlyArray<TestError>
}
```

<<<<<<< HEAD
如果测试通过，返回值将是 `TestResultPassed`：
=======
If the test passed, the return value will be `TestResultPassed`:
>>>>>>> dc3f007c37a718074d6b3b367159c7a198c04c0d

```ts
interface TestResultPassed {
  /**
   * The test passed successfully.
   */
  readonly state: 'passed'
  /**
   * Errors that were thrown during the test execution.
   */
  readonly errors: ReadonlyArray<TestError> | undefined
}
```

::: warning
请注意，状态为 `passed` 的测试仍可能附带有错误——如果 `retry` 至少触发了一次，这种情况就可能发生。
:::

## diagnostic

```ts
function diagnostic(): TestDiagnostic | undefined
```

有关测试的有用信息，例如持续时间、内存使用等：

```ts
interface TestDiagnostic {
  /**
   * If the duration of the test is above `slowTestThreshold`.
   */
  readonly slow: boolean
  /**
   * The amount of memory used by the test in bytes.
   * This value is only available if the test was executed with `logHeapUsage` flag.
   */
  readonly heap: number | undefined
  /**
   * The time it takes to execute the test in ms.
   */
  readonly duration: number
  /**
   * The time in ms when the test started.
   */
  readonly startTime: number
  /**
   * The amount of times the test was retried.
   */
  readonly retryCount: number
  /**
   * The amount of times the test was repeated as configured by `repeats` option.
   * This value can be lower if the test failed during the repeat and no `retry` is configured.
   */
  readonly repeatCount: number
  /**
   * If test passed on a second retry.
   */
  readonly flaky: boolean
}
```

::: info
`diagnostic()` will return `undefined` if the test was not scheduled to run yet.
:::
