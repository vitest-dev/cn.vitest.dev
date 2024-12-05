---
outline: deep
---

# Test API 索引

下面的类型签名中使用了以下类型：

```ts
type Awaitable<T> = T | PromiseLike<T>
type TestFunction = () => Awaitable<void>

interface TestOptions {
  /**
   * 如果执行时间过长，测试将失败
   */
  timeout?: number
  /**
   * 如果测试失败，将重试特定次数
   *
   * @default 0
   */
  retry?: number
  /**
   * 即使每次都失败，也会重复多次相同的测试
   * 如果有 "retry" 选项并且失败，它将在每个周期中使用每次重试
   * 对于调试随机故障很有用
   *
   * @default 0
   */
  repeats?: number
}
```

当测试函数返回一个 promise 时，运行器会等待它解结束收集异步的结果。如果 promise 被拒绝，测试就会失败。

::: tip
在 Jest 中，`TestFunction` 也可以是 `(done: DoneCallback) => void` 类型。如果使用这种形式，测试将在调用 `done` 之前不会结束。也可以使用 `async` 函数来实现相同的效果，请参阅[迁移指南中的回调完成部分](/guide/migration#回调完成)。

:::

大多数选项都支持点语法和对象语法，允许您使用您喜欢的任何样式。

:::code-group
```ts [dot-syntax]
import { test } from 'vitest'

test.skip('skipped test', () => {
  // 一些现在失败的逻辑
})
```
```ts [object-syntax]
import { test } from 'vitest'

test('skipped test', { skip: true }, () => {
  // 一些现在失败的逻辑
})
```

:::

## test

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number | TestOptions) => void`
- **别名:** `it`

`test` 定义了一组相关的期望。 它接收测试名称和保存测试期望的函数。

或者，我们可以提供超时（以毫秒为单位）来指定终止前等待的时间。 默认为 5 秒，可以通过 [testTimeout](/config/#testtimeout) 进行全局配置

```ts
import { expect, test } from 'vitest'

test('should work as expected', () => {
  expect(Math.sqrt(4)).toBe(2)
})
```

### test.extend {#test-extended}

- **类型:** `<T extends Record<string, any>>(fixtures: Fixtures<T>): TestAPI<ExtraContext & T>`
- **别名:** `it.extend`

使用 `test.extend` 来使用自定义的 fixtures 扩展测试上下文。这将返回一个新的 `test`，它也是可扩展的，因此可以根据需要扩展更多的 fixtures 或覆盖现有的 fixtures。有关更多信息，请参阅[扩展测试上下文](/guide/test-context.html#test-extend)。

```ts
import { expect, test } from 'vitest'

const todos = []
const archive = []

const myTest = test.extend({
  todos: async ({ task }, use) => {
    todos.push(1, 2, 3)
    await use(todos)
    todos.length = 0
  },
  archive,
})

myTest('add item', ({ todos }) => {
  expect(todos.length).toBe(3)

  todos.push(4)
  expect(todos.length).toBe(4)
})
```

### test.skip

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number | TestOptions) => void`
- **别名:** `it.skip`

如果想跳过运行某些测试，但又不想删代码，可以使用 `test.skip` 来跳过这些测试。

```ts
import { assert, test } from 'vitest'

test.skip('skipped test', () => {
  // Test skipped, no error
  assert.equal(Math.sqrt(4), 3)
})
```

还可以通过在 [context](/guide/test-context) 上动态调用 `skip` 来跳过测试：

```ts
import { assert, test } from 'vitest'

test('skipped test', (context) => {
  context.skip()
  // Test skipped, no error
  assert.equal(Math.sqrt(4), 3)
})
```

### test.skipIf

- **类型:** `(condition: any) => Test`
- **别名:** `it.skipIf`

在某些情况下，可能会需要在不同的环境下多次运行测试，而且某些测试可能是特定于环境的。我们这时候可以通过使用 `test.skipIf` 来跳过测试，而不是用 `if` 来封装测试代码。

```ts
import { assert, test } from 'vitest'

const isDev = process.env.NODE_ENV === 'development'

test.skipIf(isDev)('prod only test', () => {
  // this test only runs in production
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### test.runIf

- **类型:** `(condition: any) => Test`
- **别名:** `it.runIf`

与 [test.skipIf](#test-skipif) 相反。

```ts
import { assert, test } from 'vitest'

const isDev = process.env.NODE_ENV === 'development'

test.runIf(isDev)('dev only test', () => {
  // this test only runs in development
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### test.only

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number) => void`
- **别名:** `it.only`

使用 `test.only` 仅运行给定 测试套件 中的某些测试。这在调试时非常有用。

可选择提供超时（以毫秒为单位），用于指定终止前的等待时间。默认值为 5 秒，可通过 [testTimeout](/config/#testtimeout) 进行全局配置。

```ts
import { assert, test } from 'vitest'

test.only('test', () => {
  // Only this test (and others marked with only) are run
  assert.equal(Math.sqrt(4), 2)
})
```

有时，只运行某个文件中的 "测试"，而忽略整个 测试套件 中的所有其他测试是非常有用的，因为这些测试会污染输出。

为此，请使用包含相关测试的特定文件运行 `vitest`。

```
# vitest interesting.test.ts
```

### test.concurrent

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number) => void`
- **别名:** `it.concurrent`

`test.concurrent` 标记并行运行的连续测试。它接收测试名称、包含要收集的测试的异步函数以及可选的超时（以毫秒为单位）。

```ts
import { describe, test } from 'vitest'

// The two tests marked with concurrent will be run in parallel
describe('suite', () => {
  test('serial test', async () => {
    /* ... */
  })
  test.concurrent('concurrent test 1', async () => {
    /* ... */
  })
  test.concurrent('concurrent test 2', async () => {
    /* ... */
  })
})
```

`test.skip`、 `test.only` 和 `test.todo` 适用于并发测试。以下所有组合均有效：

```ts
test.concurrent(/* ... */)
test.skip.concurrent(/* ... */) // or test.concurrent.skip(/* ... */)
test.only.concurrent(/* ... */) // or test.concurrent.only(/* ... */)
test.todo.concurrent(/* ... */) // or test.concurrent.todo(/* ... */)
```

运行并发测试时，快照和断言必须使用本地[测试上下文](/guide/test-context.md)中的 `expect`，以确保检测到正确的测试。

```ts
test.concurrent('test 1', async ({ expect }) => {
  expect(foo).toMatchSnapshot()
})
test.concurrent('test 2', async ({ expect }) => {
  expect(foo).toMatchSnapshot()
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### test.sequential

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number) => void`

`test.sequential` 标记一个测试为顺序测试。如果要在 `describe.concurrent` 中或使用 `--sequence.concurrent` 命令选项按顺序运行测试，这一点非常有用。

```ts
import { describe, test } from 'vitest'

// with config option { sequence: { concurrent: true } }
test('concurrent test 1', async () => {
  /* ... */
})
test('concurrent test 2', async () => {
  /* ... */
})

test.sequential('sequential test 1', async () => {
  /* ... */
})
test.sequential('sequential test 2', async () => {
  /* ... */
})

// within concurrent suite
describe.concurrent('suite', () => {
  test('concurrent test 1', async () => {
    /* ... */
  })
  test('concurrent test 2', async () => {
    /* ... */
  })

  test.sequential('sequential test 1', async () => {
    /* ... */
  })
  test.sequential('sequential test 2', async () => {
    /* ... */
  })
})
```

### test.todo

- **类型:** `(name: string | Function) => void`
- **别名:** `it.todo`

使用 `test.todo` 来存根测试，以便稍后实施。测试报告中将显示一个条目，以便知道还有多少测试需要执行。

```ts
// An entry will be shown in the report for this test
test.todo('unimplemented test')
```

### test.fails

- **类型:** `(name: string | Function, fn: TestFunction, timeout?: number) => void`
- **别名:** `it.fails`

使用 `test.fails` 明确表示断言将失败。

```ts
import { expect, test } from 'vitest'

function myAsyncFunc() {
  return new Promise(resolve => resolve(1))
}
test.fails('fail test', async () => {
  await expect(myAsyncFunc()).rejects.toBe(1)
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### test.each

- **类型:** `(cases: ReadonlyArray<T>, ...args: any[]) => void`
- **别名:** `it.each`

::: tip
 `test.each` 是为了与 Jest 兼容而提供的，Vitest 还提供了 [`test.for`](#test-for)，并集成了 [`TestContext`](/guide/test-context)。
:::

当需要使用不同变量运行同一测试时，请使用 `test.each`。
我们可以按照测试功能参数的顺序，在测试名称中注入带有 [printf formatting](https://nodejs.org/api/util.html#util_util_format_format_args) 的参数。

- `%s`: string
- `%d`: number
- `%i`: integer
- `%f`: floating point value
- `%j`: json
- `%o`: object
- `%#`: index of the test case
- `%%`: single percent sign ('%')

```ts
import { expect, test } from 'vitest'

test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('add(%i, %i) -> %i', (a, b, expected) => {
  expect(a + b).toBe(expected)
})

// this will return
// ✓ add(1, 1) -> 2
// ✓ add(1, 2) -> 3
// ✓ add(2, 1) -> 3
```

如果使用对象作为参数，也可以使用前缀 `$` 访问对象属性：

```ts
test.each([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 1, expected: 3 },
])('add($a, $b) -> $expected', ({ a, b, expected }) => {
  expect(a + b).toBe(expected)
})

// this will return
// ✓ add(1, 1) -> 2
// ✓ add(1, 2) -> 3
// ✓ add(2, 1) -> 3
```

如果使用对象作为参数，也可以使用 `.` 访问对象属性：

```ts
import { expect, test } from 'vitest'

test.each`
  a             | b      | expected
  ${{ val: 1 }} | ${'b'} | ${'1b'}
  ${{ val: 2 }} | ${'b'} | ${'2b'}
  ${{ val: 3 }} | ${'b'} | ${'3b'}
`('add($a.val, $b) -> $expected', ({ a, b, expected }) => {
  expect(a.val + b).toBe(expected)
})

// this will return
// ✓ add(1, b) -> 1b
// ✓ add(2, b) -> 2b
// ✓ add(3, b) -> 3b
```

从 Vitest 0.25.3 开始，还可以使用模板字符串表。

- 第一行应为列名，用 `|` 分隔；
- 使用 `${value}` 语法，以模板字面表达式的形式提供后面一行或多行数据。
-

```ts
test.each`
  a             | b      | expected
  ${1}          | ${1}   | ${2}
  ${'a'}        | ${'b'} | ${'ab'}
  ${[]}         | ${'b'} | ${'b'}
  ${{}}         | ${'b'} | ${'[object Object]b'}
  ${{ asd: 1 }} | ${'b'} | ${'[object Object]b'}
`('returns $expected when $a is added $b', ({ a, b, expected }) => {
  expect(a + b).toBe(expected)
})
```

::: tip
Vitest 使用 chai `format` 方法处理 `$values`。如果数值太短，可以在配置文件中增加 [chaiConfig.truncateThreshold](/config/#chaiconfig-truncatethreshold)。
:::

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### test.for

- **Alias:** `it.for`

作为 `test.each` 的替代，提供 [`TestContext`](/guide/test-context)。

与 `test.each` 的区别在于如何在参数中提供数组情况。
其他非数组情况（包括模板字符串的使用）完全相同。

```ts
// `each` spreads array case
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('add(%i, %i) -> %i', (a, b, expected) => { // [!code --]
  expect(a + b).toBe(expected)
})

// `for` doesn't spread array case
test.for([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('add(%i, %i) -> %i', ([a, b, expected]) => { // [!code ++]
  expect(a + b).toBe(expected)
})
```

第二个参数是 [`TestContext`](/guide/test-context)，可用于并发快照等

```ts
test.concurrent.for([
  [1, 1],
  [1, 2],
  [2, 1],
])('add(%i, %i)', ([a, b], { expect }) => {
  expect(a + b).matchSnapshot()
})
```

## bench

- **类型:** `(name: string | Function, fn: BenchFunction, options?: BenchOptions) => void`

`bench` 定义了一个基准。在 Vitest 术语中，基准是定义一系列操作的函数。Vitest 会多次运行该函数，以显示不同的性能结果。

Vitest 使用了 [`tinybench`](https://github.com/tinylibs/tinybench)库，继承其所有可用作第三个参数的选项。

```ts
import { bench } from 'vitest'

bench(
  'normal sorting',
  () => {
    const x = [1, 5, 4, 2, 3]
    x.sort((a, b) => {
      return a - b
    })
  },
  { time: 1000 }
)
```

```ts
export interface Options {
  /**
   * 运行基准任务所需时间（毫秒）
   * @default 500
   */
  time?: number

  /**
   * 如果连时间选项都已完成，任务应运行的次数
   * @default 10
   */
  iterations?: number

  /**
   * 函数以毫秒为单位获取当前时间戳
   */
  now?: () => number

  /**
   * 用于中止基准测试的中止信号
   */
  signal?: AbortSignal

  /**
   * 任务失败时抛出（如果为 true，事件将不起作用）
   */
  throws?: boolean

  /**
   * 预热时间（毫秒）
   * @default 100ms
   */
  warmupTime?: number

  /**
   * 热身迭代
   * @default 5
   */
  warmupIterations?: number

  /**
   * 在每个基准任务（周期）之前运行的设置函数
   */
  setup?: Hook

  /**
   * 在每个基准任务（周期）之后运行的拆机函数
   */
  teardown?: Hook
}
```
测试用例运行后，输出结构信息如下：

```
  name                      hz     min     max    mean     p75     p99    p995    p999     rme  samples
· normal sorting  6,526,368.12  0.0001  0.3638  0.0002  0.0002  0.0002  0.0002  0.0004  ±1.41%   652638
```
```ts
export interface TaskResult {
  /*
   * 运行任务时发生的最后一次错误
   */
  error?: unknown

  /**
   * 以毫秒为单位的基准任务运行时间（周期）。
   */
  totalTime: number

  /**
   * 样本中的最小值
   */
  min: number
  /**
   * 样本中的最大值
   */
  max: number

  /**
   * 每秒的操作次数
   */
  hz: number

  /**
   * 每个操作需要多长时间（毫秒）
   */
  period: number

  /**
   * 每个任务的任务样本迭代时间（毫秒）
   */
  samples: number[]

  /**
   * 样本平均数/平均值（总体平均数的估计值）
   */
  mean: number

  /**
   * 样本方差（总体方差的估计值）
   */
  variance: number

  /**
   * 样本标准差（总体标准差的估计值）
   */
  sd: number

  /**
   * 平均值的标准误差（又称样本平均值的抽样分布标准差）
   */
  sem: number

  /**
   * 自由度
   */
  df: number

  /**
   * 样本临界值
   */
  critical: number

  /**
   * 误差率
   */
  moe: number

  /**
   * 相对误差
   */
  rme: number

  /**
   * 中位绝对偏差
   */
  mad: number

  /**
   * P50/中位百分位数
   */
  p50: number

  /**
   * p75 百分位数
   */
  p75: number

  /**
   * p99 百分位数
   */
  p99: number

  /**
   * p995 百分位数
   */
  p995: number

  /**
   * p999 百分位数
   */
  p999: number
}
```

### bench.skip

- **类型:** `(name: string | Function, fn: BenchFunction, options?: BenchOptions) => void`

可以使用 "bench.skip "语法跳过运行某些基准。

```ts
import { bench } from 'vitest'

bench.skip('normal sorting', () => {
  const x = [1, 5, 4, 2, 3]
  x.sort((a, b) => {
    return a - b
  })
})
```

### bench.only

- **类型:** `(name: string | Function, fn: BenchFunction, options?: BenchOptions) => void`

使用 `bench.only` 仅运行给定测试套件中的某些基准。这在调试时非常有用。

```ts
import { bench } from 'vitest'

bench.only('normal sorting', () => {
  const x = [1, 5, 4, 2, 3]
  x.sort((a, b) => {
    return a - b
  })
})
```

### bench.todo

- **类型:** `(name: string | Function) => void`

使用 `bench.todo` 来存根基准，以便以后实施。

```ts
import { bench } from 'vitest'

bench.todo('unimplemented test')
```

## describe

当在文件的顶层使用 `test` 或 `bench` 时，它们会作为隐式套件的一部分被收集起来。使用 `describe` 可以在当前上下文中定义一个新的测试套件，作为一组相关测试或基准以及其他嵌套测试套件。测试套件可让组织测试和基准，使报告更加清晰。

```ts
// basic.spec.ts
// organizing tests

import { describe, expect, test } from 'vitest'

const person = {
  isActive: true,
  age: 32,
}

describe('person', () => {
  test('person is defined', () => {
    expect(person).toBeDefined()
  })

  test('is active', () => {
    expect(person.isActive).toBeTruthy()
  })

  test('age limit', () => {
    expect(person.age).toBeLessThanOrEqual(32)
  })
})
```

```ts
// basic.bench.ts
// organizing benchmarks

import { bench, describe } from 'vitest'

describe('sort', () => {
  bench('normal', () => {
    const x = [1, 5, 4, 2, 3]
    x.sort((a, b) => {
      return a - b
    })
  })

  bench('reverse', () => {
    const x = [1, 5, 4, 2, 3]
    x.reverse().sort((a, b) => {
      return a - b
    })
  })
})
```

如果测试或基准具有层次结构，还可以嵌套描述块：

```ts
import { describe, expect, test } from 'vitest'

function numberToCurrency(value: number | string) {
  if (typeof value !== 'number') {
    throw new TypeError('Value must be a number')
  }

  return value
    .toFixed(2)
    .toString()
    .replace(/\B(?=(\d{3})+(?!\d))/g, ',')
}

describe('numberToCurrency', () => {
  describe('given an invalid number', () => {
    test('composed of non-numbers to throw error', () => {
      expect(() => numberToCurrency('abc')).toThrowError()
    })
  })

  describe('given a valid number', () => {
    test('returns the correct currency format', () => {
      expect(numberToCurrency(10000)).toBe('10,000.00')
    })
  })
})
```

### describe.skip

- **类型:** `(name: string | Function, fn: TestFunction, options?: number | TestOptions) => void`

在套件中使用 `describe.skip` 可避免运行特定的 describe 块。

```ts
import { assert, describe, test } from 'vitest'

describe.skip('skipped suite', () => {
  test('sqrt', () => {
    // Suite skipped, no error
    assert.equal(Math.sqrt(4), 3)
  })
})
```

### describe.skipIf

- **类型:** `(condition: any) => void`

在某些情况下，可能会在不同的环境下多次运行套件，其中一些测试套件可能是特定于环境的。可以使用 `describe.skipIf` 来跳过条件为真时的套件，而不是使用 `if` 来封装套件。

```ts
import { describe, test } from 'vitest'

const isDev = process.env.NODE_ENV === 'development'

describe.skipIf(isDev)('prod only test suite', () => {
  // this test suite only runs in production
})
```

::: warning
将 Vitest 用作[类型检查器](/guide/testing-types)时，你不能使用此语法。
:::

### describe.runIf

- **类型:** `(condition: any) => void`

与 [describe.skipIf](#describe-skipif) 相反。

```ts
import { assert, describe, test } from 'vitest'

const isDev = process.env.NODE_ENV === 'development'

describe.runIf(isDev)('dev only test suite', () => {
  // this test suite only runs in development
})
```

::: warning
将 Vitest 用作[类型检查器](/guide/testing-types)时，你不能使用此语法。
:::

### describe.only

- **类型:** `(name: string | Function, fn: TestFunction, options?: number | TestOptions) => void`

使用 `describe.only` 只运行某些测试套件

```ts
import { assert, describe, test } from 'vitest'

// Only this suite (and others marked with only) are run
describe.only('suite', () => {
  test('sqrt', () => {
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('other suite', () => {
  // ... will be skipped
})
```

In order to do that run `vitest` with specific file containing the tests in question.

有时，只运行某个文件中的测试套件，而忽略整个测试套件中的所有其他测试是非常有用的，因为这些测试会污染输出。

要做到这一点，请在包含相关测试的特定文件中运行 `vitest`。

```
# vitest interesting.test.ts
```

### describe.concurrent

- **类型:** `(name: string | Function, fn: TestFunction, options?: number | TestOptions) => void`

测试套件中的 `describe.concurrent` 会将所有测试标记为并发测试

```ts
import { describe, test } from 'vitest'

// All suites and tests within this suite will be run in parallel
describe.concurrent('suite', () => {
  test('concurrent test 1', async () => {
    /* ... */
  })
  describe('concurrent suite 2', async () => {
    test('concurrent test inner 1', async () => {
      /* ... */
    })
    test('concurrent test inner 2', async () => {
      /* ... */
    })
  })
  test.concurrent('concurrent test 3', async () => {
    /* ... */
  })
})
```

`.skip`、`.only`和`.todo`适用于并发测试套件。以下所有组合都有效：

```ts
describe.concurrent(/* ... */)
describe.skip.concurrent(/* ... */) // or describe.concurrent.skip(/* ... */)
describe.only.concurrent(/* ... */) // or describe.concurrent.only(/* ... */)
describe.todo.concurrent(/* ... */) // or describe.concurrent.todo(/* ... */)
```

运行并发测试时，快照和断言必须使用本地[测试上下文](/guide/test-context.md)中的 `expect` ，以确保检测到正确的测试。

```ts
describe.concurrent('suite', () => {
  test('concurrent test 1', async ({ expect }) => {
    expect(foo).toMatchSnapshot()
  })
  test('concurrent test 2', async ({ expect }) => {
    expect(foo).toMatchSnapshot()
  })
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### describe.sequential

- **类型:** `(name: string | Function, fn: TestFunction, options?: number | TestOptions) => void`

测试套件中的 `describe.sequential` 会将每个测试标记为顺序测试。如果需要在 `describe.concurrent` 中或使用 `--sequence.concurrent` 命令选项按顺序运行测试，这一点非常有用。

```ts
import { describe, test } from 'vitest'

describe.concurrent('suite', () => {
  test('concurrent test 1', async () => {
    /* ... */
  })
  test('concurrent test 2', async () => {
    /* ... */
  })

  describe.sequential('', () => {
    test('sequential test 1', async () => {
      /* ... */
    })
    test('sequential test 2', async () => {
      /* ... */
    })
  })
})
```

### describe.shuffle

- **类型:** `(name: string | Function, fn: TestFunction, options?: number | TestOptions) => void`

Vitest 通过 CLI 标志 [`--sequence.shuffle`](/guide/cli) 或配置选项 [`sequence.shuffle`](/config/#sequence-shuffle)，提供了一种以随机顺序运行所有测试的方法，但如果只想让测试套件的一部分以随机顺序运行测试，可以用这个标志来标记它。

```ts
import { describe, test } from 'vitest'

// or describe('suite', { shuffle: true }, ...)
describe.shuffle('suite', () => {
<<<<<<< HEAD
  test('random test 1', async () => {
    /* ... */
  })
  test('random test 2', async () => {
    /* ... */
  })
  test('random test 3', async () => {
    /* ... */
=======
  test('random test 1', async () => { /* ... */ })
  test('random test 2', async () => { /* ... */ })
  test('random test 3', async () => { /* ... */ })

  // `shuffle` is inherited
  describe('still random', () => {
    test('random 4.1', async () => { /* ... */ })
    test('random 4.2', async () => { /* ... */ })
  })

  // disable shuffle inside
  describe('not random', { shuffle: false }, () => {
    test('in order 5.1', async () => { /* ... */ })
    test('in order 5.2', async () => { /* ... */ })
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f
  })
})
// order depends on sequence.seed option in config (Date.now() by default)
```

`.skip`、`.only`和`.todo`适用于随机测试套件。

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

### describe.todo

- **类型:** `(name: string | Function) => void`

使用 `describe.todo` 来暂存待以后实施的套件。测试报告中会显示一个条目，这样就能知道还有多少测试需要执行。

```ts
// An entry will be shown in the report for this suite
describe.todo('unimplemented suite')
```

### describe.each

- **类型:** `(cases: ReadonlyArray<T>, ...args: any[]): (name: string | Function, fn: (...args: T[]) => void, options?: number | TestOptions) => void`

如果有多个测试依赖于相同的数据，请使用 `describe.each` 。

```ts
import { describe, expect, test } from 'vitest'

describe.each([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 1, expected: 3 },
])('describe object add($a, $b)', ({ a, b, expected }) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected)
  })

  test(`returned value not be greater than ${expected}`, () => {
    expect(a + b).not.toBeGreaterThan(expected)
  })

  test(`returned value not be less than ${expected}`, () => {
    expect(a + b).not.toBeLessThan(expected)
  })
})
```

从 Vitest 0.25.3 开始，还可以使用模板字符串表。

- 第一行应为列名，用 `|` 分隔；
- 使用 `${value}` 语法，以模板字面表达式的形式提供后面一行或多行数据。

```ts
import { describe, expect, test } from 'vitest'

describe.each`
  a             | b      | expected
  ${1}          | ${1}   | ${2}
  ${'a'}        | ${'b'} | ${'ab'}
  ${[]}         | ${'b'} | ${'b'}
  ${{}}         | ${'b'} | ${'[object Object]b'}
  ${{ asd: 1 }} | ${'b'} | ${'[object Object]b'}
`('describe template string add($a, $b)', ({ a, b, expected }) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected)
  })
})
```

::: warning
在将 Vitest 用作[类型检查器](/guide/testing-types)时，不能使用此语法。
:::

## Setup and Teardown

通过这些函数，我们可以挂钩测试的生命周期，避免重复设置和拆卸代码。它们适用于当前上下文：如果在顶层使用，则适用于文件；如果在 `describe` 块内使用，则适用于当前测试套件。
将 Vitest 作为类型检查器运行时，不会调用这些钩子。

### beforeEach

- **类型:** `beforeEach(fn: () => Awaitable<void>, timeout?: number)`

注册一个回调函数，在当前上下文中的每个测试运行前调用。
如果函数返回一个 Promise ，Vitest 会等待承诺解析后再运行测试。

作为选项，可以传递一个超时（以毫秒为单位），定义终止前需要等待的时间。默认值为 5 秒。

```ts
import { beforeEach } from 'vitest'

beforeEach(async () => {
  // Clear mocks and add some testing data after before each test run
  await stopMocking()
  await addUser({ name: 'John' })
})
```

这里， `beforeEach` 确保每次测试都会添加用户。

`beforeEach` 还接受一个可选的清理函数（相当于 `afterEach`）。

```ts
import { beforeEach } from 'vitest'

beforeEach(async () => {
  // called once before each test run
  await prepareSomething()

  // clean up function, called once after each test run
  return async () => {
    await resetSomething()
  }
})
```

### afterEach

- **类型 :** `afterEach(fn: () => Awaitable<void>, timeout?: number)`

注册一个回调函数，在当前上下文中的每个测试完成后调用。
如果函数返回一个承诺，Vitest 会等待承诺解析后再继续。

可以选择提供一个超时（毫秒），用于指定终止前的等待时间。默认值为 5 秒。

```ts
import { afterEach } from 'vitest'

afterEach(async () => {
  await clearTestingData() // clear testing data after each test run
})
```

在这里，`afterEach` 可确保在每次测试运行后清除测试数据。

::: tip
Vitest 在 1.3.0 新增 [`onTestFinished`](#ontestfinished)。你可以在测试执行过程中调用它，以便在测试运行结束后清理任何状态。
:::

### beforeAll

- **类型:** `beforeAll(fn: () => Awaitable<void>, timeout?: number)`

注册一个回调函数，在开始运行当前上下文中的所有测试之前调用一次。
如果函数返回一个 Promise ，Vitest 会等待承诺解析后再运行测试。

可以选择提供一个超时（毫秒），用于指定终止前的等待时间。默认值为 5 秒。

```ts
import { beforeAll } from 'vitest'

beforeAll(async () => {
  await startMocking() // called once before all tests run
})
```

这里的 `beforeAll` 确保在测试运行前设置好模拟数据。

`beforeAll` 还接受一个可选的清理函数（相当于 `afterAll`）。

```ts
import { beforeAll } from 'vitest'

beforeAll(async () => {
  // called once before all tests run
  await startMocking()

  // clean up function, called once after all tests run
  return async () => {
    await stopMocking()
  }
})
```

### afterAll

- **类型:** `afterAll(fn: () => Awaitable<void>, timeout?: number)`

注册一个回调函数，以便在当前上下文中所有测试运行完毕后调用一次。
如果函数返回一个 Promise ，Vitest 会等待承诺解析后再继续。

你还可以选择提供超时（毫秒），以指定终止前的等待时间。默认值为 5 秒。

```ts
import { afterAll } from 'vitest'

afterAll(async () => {
  await stopMocking() // this method is called after all tests run
})
```

这里的 `afterAll` 确保在所有测试运行后调用 `stopMocking` 方法。

## Test Hooks

Vitest 提供了一些 hooks，你可以在 _测试执行期间_ 调用这些钩子，以便在测试运行结束后清理状态。

::: warning
如果在测试体之外调用这些 hooks ，则会出错。
:::

### onTestFinished {#ontestfinished}

<<<<<<< HEAD
这个 hook 总是在测试运行结束后调用。它在 `afterEach` 之后被调用，因为它们会影响测试结果。它接收一个包含当前测试结果的 `TaskResult` 。
=======
This hook is always called after the test has finished running. It is called after `afterEach` hooks since they can influence the test result. It receives an `ExtendedContext` object like `beforeEach` and `afterEach`.
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f

```ts {1,5}
import { onTestFinished, test } from 'vitest'

test('performs a query', () => {
  const db = connectDb()
  onTestFinished(() => db.close())
  db.query('SELECT * FROM users')
})
```

::: warning
如果要并发运行测试，应该始终使用测试上下文中的 `onTestFinished` ，因为 Vitest 不会在全局 hook 中跟踪并发测试：

```ts {3,5}
import { test } from 'vitest'

test.concurrent('performs a query', ({ onTestFinished }) => {
  const db = connectDb()
  onTestFinished(() => db.close())
  db.query('SELECT * FROM users')
})
```

:::

这个 hook 在创建可重复使用的逻辑时特别有用：

```ts
// 这可以是一个单独的文件
function getTestDb() {
  const db = connectMockedDb()
  onTestFinished(() => db.close())
  return db
}

test('performs a user query', async () => {
  const db = getTestDb()
  expect(await db.query('SELECT * from users').perform()).toEqual([])
})

test('performs an organization query', async () => {
  const db = getTestDb()
  expect(await db.query('SELECT * from organizations').perform()).toEqual([])
})
```

::: tip
该钩子总是以相反顺序调用，不受 [`sequence.hooks`](/config/#sequence-hooks) 选项的影响。
:::

### onTestFailed

<<<<<<< HEAD
只有在测试失败后才会调用这个 hook 。它在 `afterEach` 之后被调用，因为它们会影响测试结果。它将接收一个包含当前测试结果的 `TaskResult` 。这个 hook 对调试非常有用。
=======
This hook is called only after the test has failed. It is called after `afterEach` hooks since they can influence the test result. It receives an `ExtendedContext` object like `beforeEach` and `afterEach`. This hook is useful for debugging.
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f

```ts {1,5-7}
import { onTestFailed, test } from 'vitest'

test('performs a query', () => {
  const db = connectDb()
  onTestFailed((e) => {
    console.log(e.result.errors)
  })
  db.query('SELECT * FROM users')
})
```

::: warning
如果要并发运行测试，应始终使用测试上下文中的 `onTestFailed` ，因为 Vitest 不会在全局 hook 中跟踪并发测试：

```ts {3,5-7}
import { test } from 'vitest'

test.concurrent('performs a query', ({ onTestFailed }) => {
  const db = connectDb()
  onTestFailed((result) => {
    console.log(result.errors)
  })
  db.query('SELECT * FROM users')
})
```

:::
