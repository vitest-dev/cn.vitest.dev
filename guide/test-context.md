---
title: 测试上下文 | 指南
---

# 测试上下文

受 [Playwright Fixtures](https://playwright.dev/docs/test-fixtures) 的启发，Vitest 的测试上下文允许你定义可在测试中使用的工具(utils)、状态(states)和固定装置(fixtures)。

## 用法

第一个参数或每个测试回调是一个测试上下文。

```ts
import { it } from 'vitest'

it('should work', (ctx) => {
  // 打印测试的名称
  console.log(ctx.task.name)
})
```

## 内置测试上下文

#### `context.task`

包含关于测试的元数据的只读对象。

#### `context.expect`

绑定到当前测试的 `expect` API:

```ts
import { it } from 'vitest'

it('math is easy', ({ expect }) => {
  expect(2 + 2).toBe(4)
})
```

此 API 对于同时运行快照测试非常有用，因为全局 Expect 无法跟踪它们:

```ts
import { it } from 'vitest'

it.concurrent('math is easy', ({ expect }) => {
  expect(2 + 2).toMatchInlineSnapshot()
})

it.concurrent('math is hard', ({ expect }) => {
  expect(2 * 2).toMatchInlineSnapshot()
})
```

#### `context.skip`

跳过后续测试执行并将测试标记为已跳过：

```ts
import { expect, it } from 'vitest'

it('math is hard', ({ skip }) => {
  skip()
  expect(2 + 2).toBe(5)
})
```

## 扩展测试上下文

Vitest 提供了两种不同的方式来帮助你扩展测试上下文。

### `test.extend`

与 [Playwright](https://playwright.dev/docs/api/class-test#test-extend) 一样，你可以使用此方法通过自定义装置定义你自己的 `test` API，并在任何地方重复使用它。

例如，我们首先使用两个固定装置创建 `myTest`，`todos` 和 `archive`。

```ts
// my-test.ts
import { test } from 'vitest'

const todos = []
const archive = []

export const myTest = test.extend({

  todos: async ({}, use) => {
    // 在每次测试函数运行之前设置固定装置
    todos.push(1, 2, 3)

    // 使用固定装置的值
    await use(todos)

    // 在每次测试函数运行之后清除固定装置
    todos.length = 0
  },
  archive,
})
```

然后我们就可以导入使用了。

```ts
import { expect } from 'vitest'
import { myTest } from './my-test.js'

myTest('add items to todos', ({ todos }) => {
  expect(todos.length).toBe(3)

  todos.push(4)
  expect(todos.length).toBe(4)
})

myTest('move items from todos to archive', ({ todos, archive }) => {
  expect(todos.length).toBe(3)
  expect(archive.length).toBe(0)

  archive.push(todos.pop())
  expect(todos.length).toBe(2)
  expect(archive.length).toBe(1)
})
```

我们还可以通过扩展 `myTest` 添加更多的固定装置或覆盖现有的固定装置。

```ts
export const myTest2 = myTest.extend({
  settings: {
    // ...
  },
})
```

#### 固定装置初始化

Vitest 运行器将智能地初始化你的固定装置并根据使用情况将它们注入到测试上下文中。

```ts
import { test } from 'vitest'

async function todosFn({ task }, use) {
  await use([1, 2, 3])
}

const myTest = test.extend({
  todos: todosFn,
  archive: [],
})

// todosFn 不会运行
myTest('', () => {})
myTest('', ({ archive }) => {})

// todosFn 会运行
myTest('', ({ todos }) => {})
```

::: warning
在固定装置中使用 `test.extend()` 时，需要始终使用对象解构模式 `{ todos }` 来访问固定装置函数和测试函数中的上下文。
:::

#### 自动化装置

Vitest 还支持 fixture 的元组语法，允许你传递每个 fixture 的选项。例如，你可以使用它来显式初始化固定装置，即使它没有在测试中使用。

```ts
import { test as base } from 'vitest'

const test = base.extend({
  fixture: [
    async ({}, use) => {
      // this function will run
      setup()
      await use()
      teardown()
    },
    { auto: true }, // Mark as an automatic fixture
  ],
})

test('works correctly')
```

#### Default fixture

从 Vitest 2.2 开始，你可以在不同的 [projects](/guide/workspace) 中提供不同的值。要启用此功能，请将`{ injected: true }` 传递给选项。如果未在 [project configuration](/config/#provide) 中指定密钥，则将使用默认值。

:::code-group
```ts [fixtures.test.ts]
import { test as base } from 'vitest'

const test = base.extend({
  url: [
    // default value if "url" is not defined in the config
    'default',
    // mark the fixure as "injected" to allow the override
    { injected: true },
  ],
})

test('works correctly', ({ url }) => {
  // url is "/default" in "project-new"
  // url is "/full" in "project-full"
  // url is "/empty" in "project-empty"
})
```
```ts [vitest.workspace.ts]
import { defineWorkspace } from 'vitest/config'

export default defineWorkspace([
  {
    test: {
      name: 'project-new',
    },
  },
  {
    test: {
      name: 'project-full',
      provide: {
        url: '/full',
      },
    },
  },
  {
    test: {
      name: 'project-empty',
      provide: {
        url: '/empty',
      },
    },
  },
])
```
:::

#### TypeScript

要为所有自定义上下文提供固定装置类型，你可以将固定装置类型作为泛型(generic)传递。

```ts
interface MyFixtures {
  todos: number[]
  archive: number[]
}

const myTest = test.extend<MyFixtures>({
  todos: [],
  archive: [],
})

myTest('types are defined correctly', (context) => {
  expectTypeOf(context.todos).toEqualTypeOf<number[]>()
  expectTypeOf(context.archive).toEqualTypeOf<number[]>()
})
```

### `beforeEach` and `afterEach`

每个测试的上下文都不同。 你可以在 `beforeEach` 和 `afterEach` hooks 中访问和扩展它们。

```ts
import { beforeEach, it } from 'vitest'

beforeEach(async (context) => {
  // 扩展上下文
  context.foo = 'bar'
})

it('should work', ({ foo }) => {
  console.log(foo) // 'bar'
})
```

#### TypeScript

你可以通过添加聚合(aggregate)类型 `TestContext`, 为你的自定义上下文属性提供类型支持。

```ts
declare module 'vitest' {
  export interface TestContext {
    foo?: string
  }
}
```

如果你只想为特定的 `beforeEach`、`afterEach`、`it` 或 `test` hooks 提供属性类型，则可以将类型作为泛型(generic)传递。

```ts
interface LocalTestContext {
  foo: string
}

beforeEach<LocalTestContext>(async (context) => {
  // 上下文的类型是 'TestContext & LocalTestContext'
  context.foo = 'bar'
})

it<LocalTestContext>('should work', ({ foo }) => {
  // foo 的类型是 'string'
  console.log(foo) // 'bar'
})
```
