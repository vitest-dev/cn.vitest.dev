---
title: 模拟对象 | 指南
---

# 模拟对象

在编写测试时，你可能会因为时间问题，需要创建内部或外部服务的 “假” 版本，这通常被称为 **对象模拟** 操作。Vitest 通过 **vi** 提供了一些实用的函数用于解决这个问题。你可以使用 `import { vi } from 'vitest'` 或者 **全局配置** 进行访问它 (当 **启用** [全局配置](/config/#globals) 时)。

::: warning
不要忘记在每次测试运行前后清除或恢复模拟对象，以撤消运行测试时模拟对象状态的更改！有关更多信息，请参阅 [`mockReset`](/api/mock.html#mockreset) 文档。
:::

如果你想从头开始，请查看 [API 部分](/api/#vi) 的 vi 部分，或者继续跟着文档深入了解一下这个对象模拟的世界。

## 日期

有些时候，你可能需要控制日期来确保测试时的一致性。Vitest 使用了 [`@sinonjs/fake-timers`](https://github.com/sinonjs/fake-timers) 库来操作计时器以及系统日期。可以在 [此处](/api/vi#vi-setsystemtime) 找到有关特定 API 的更多详细信息。

### 示例

```js
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest'

const businessHours = [9, 17]

function purchase() {
  const currentHour = new Date().getHours()
  const [open, close] = businessHours

  if (currentHour > open && currentHour < close) {
    return { message: 'Success' }
  }

  return { message: 'Error' }
}

describe('purchasing flow', () => {
  beforeEach(() => {
    // 告诉 vitest 我们使用模拟时间
    vi.useFakeTimers()
  })

  afterEach(() => {
    // 每次测试运行后恢复日期
    vi.useRealTimers()
  })

  it('allows purchases within business hours', () => {
    // 将时间设置在工作时间之内
    const date = new Date(2000, 1, 1, 13)
    vi.setSystemTime(date)

    // 访问 Date.now() 将生成上面设置的日期
    expect(purchase()).toEqual({ message: 'Success' })
  })

  it('disallows purchases outside of business hours', () => {
    // 将时间设置在工作时间之外
    const date = new Date(2000, 1, 1, 19)
    vi.setSystemTime(date)

    // 访问 Date.now() 将生成上面设置的日期
    expect(purchase()).toEqual({ message: 'Error' })
  })
})
```

## 函数

函数的模拟可以分为两个不同的类别：_对象监听(spying) & 对象模拟_。

有时你可能只需要验证是否调用了特定函数（以及可能传递了哪些参数）。在这种情况下，我们就需要使用一个对象监听，可以直接使用 `vi.spyOn()` ([在此处阅读更多信息](/api/vi#vi-spyon))。

然而，对象监听只能帮助你 **监听** 函数，他们无法改变这些函数的实现。如果我们需要创建一个函数的假（或模拟）版本，可以使用它 `vi.fn()` ([在此处阅读更多信息](/api/vi#vi-fn))。

我们使用 [Tinyspy](https://github.com/tinylibs/tinyspy) 作为模拟函数的基础，同时也有一套自己的封装来使其与 `Jest` 兼容。`vi.fn()` 和 `vi.spyOn()` 共享相同的方法，但是只有 `vi.fn()` 的返回结果是可调用的。

### 示例

```js
import { afterEach, describe, expect, it, vi } from 'vitest'

const messages = {
  items: [
    { message: 'Simple test message', from: 'Testman' },
    // ...
  ],
  getLatest, // 也可以是一个 `getter 或 setter 如果支持`
}

function getLatest(index = messages.items.length - 1) {
  return messages.items[index]
}

describe('reading messages', () => {
  afterEach(() => {
    vi.restoreAllMocks()
  })

  it('should get the latest message with a spy', () => {
    const spy = vi.spyOn(messages, 'getLatest')
    expect(spy.getMockName()).toEqual('getLatest')

    expect(messages.getLatest()).toEqual(
      messages.items[messages.items.length - 1]
    )

    expect(spy).toHaveBeenCalledTimes(1)

    spy.mockImplementationOnce(() => 'access-restricted')
    expect(messages.getLatest()).toEqual('access-restricted')

    expect(spy).toHaveBeenCalledTimes(2)
  })

  it('should get with a mock', () => {
    const mock = vi.fn().mockImplementation(getLatest)

    expect(mock()).toEqual(messages.items[messages.items.length - 1])
    expect(mock).toHaveBeenCalledTimes(1)

    mock.mockImplementationOnce(() => 'access-restricted')
    expect(mock()).toEqual('access-restricted')

    expect(mock).toHaveBeenCalledTimes(2)

    expect(mock()).toEqual(messages.items[messages.items.length - 1])
    expect(mock).toHaveBeenCalledTimes(3)
  })
})
```

### 了解更多

- [Jest's Mock Functions](https://jestjs.io/docs/mock-function-api)

## 全局(Globals)

你可以通过使用 [`vi.stubGlobal`](/api/vi#stubglobal) 来模拟 `jsdom` 或 `node` 中不存在的全局变量。它将把全局变量的值放入 `globalThis` 对象。

```ts
import { vi } from 'vitest'

const IntersectionObserverMock = vi.fn(() => ({
  disconnect: vi.fn(),
  observe: vi.fn(),
  takeRecords: vi.fn(),
  unobserve: vi.fn(),
}))

vi.stubGlobal('IntersectionObserver', IntersectionObserverMock)

// 现在你可以通过 `IntersectionObserver` 或 `window.IntersectionObserver` 访问
```

## 模块

模拟模块监听在其他代码中调用的第三方库，允许你测试参数、输出甚至重新声明其实现。

### 自动模拟算法(Automocking algorithm)

参见 [`vi.mock()` API 部分](/api/vi#vi-mock) 以获得更深入详细 API 描述。

如果你的代码导入了模拟模块，并且没有任何与此模块相关联的 `__mocks__` 文件或 `factory`，Vitest 将通过调用模块并模拟每个导出来的模拟模块本身。

以下原则适用

- 所有的数组将被清空
- 所有的基础类型和集合将保持不变
- 所有的对象都将被深度克隆
- 类的所有实例及其原型都将被深度克隆

### Virtual Modules

Vitest 支持模拟 Vite [虚拟模块](https://cn.vitejs.dev/guide/api-plugin#virtual-modules-convention)。它的工作方式与 Jest 中处理虚拟模块的方式不同。我们不需要将 `virtual: true` 传递给 `vi.mock` 函数，而是需要告诉 Vite 模块存在，否则它将在解析过程中失败。有几种方法可以做到这一点：

1. 提供别名

```ts
// vitest.config.js
export default {
  test: {
    alias: {
      '$app/forms': resolve('./mocks/forms.js'),
    },
  },
}
```

2. 提供解析虚拟模块的插件

```ts
// vitest.config.js
export default {
  plugins: [
    {
      name: 'virtual-modules',
      resolveId(id) {
        if (id === '$app/forms') {
          return 'virtual:$app/forms'
      },
    },
  ],
}
```

第二种方法的好处是可以动态创建不同的虚拟入口点。如果将多个虚拟模块重定向到一个文件中，那么所有这些模块都将受到 `vi.mock` 的影响，因此请确保使用唯一的标识符。

### Mocking Pitfalls

请注意，对在同一文件的其他方法中调用的方法的模拟调用是不可能的。例如，在此代码中：

```ts
export function foo() {
  return 'foo'
}

export function foobar() {
  return `${foo()}bar`
}
```

不可能从外部模拟 `foo` 方法，因为它是直接引用的。因此，此代码对 `foobar` 内部的 `foo` 调用没有影响（但会影响其他模块中的 `foo` 调用）：

```ts
import { vi } from 'vitest'
import * as mod from './foobar.js'

// 这只会影响在原始模块之外的 "foo"
vi.spyOn(mod, 'foo')
vi.mock('./foobar.js', async (importOriginal) => {
  return {
    ...(await importOriginal<typeof import('./foobar.js')>()),
    // 这只会影响在原始模块之外的 "foo"
    foo: () => 'mocked',
  }
})
```

你可以通过直接向 `foobar` 方法提供实现来确认这种行为：

```ts
// foobar.test.js
import * as mod from './foobar.js'

vi.spyOn(mod, 'foo')

// 导出的 foo  引用模拟的方法
mod.foobar(mod.foo)
```

```ts
// foobar.js
export function foo() {
  return 'foo'
}

export function foobar(injectedFoo) {
  return injectedFoo === foo // false
}
```

这就是预期行为。当以这种方式包含 mock 时，这通常是不良代码的标志。考虑将代码重构为多个文件，或者使用[依赖项注入](https://en.wikipedia.org/wiki/dependency_injection)等技术来改进应用体系结构。

### 示例

```js
import { Client } from 'pg'
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest'
import { failure, success } from './handlers.js'

// get todos
export async function getTodos(event, context) {
  const client = new Client({
    // ...clientOptions
  })

  await client.connect()

  try {
    const result = await client.query('SELECT * FROM todos;')

    client.end()

    return success({
      message: `${result.rowCount} item(s) returned`,
      data: result.rows,
      status: true,
    })
  }
  catch (e) {
    console.error(e.stack)

    client.end()

    return failure({ message: e, status: false })
  }
}

vi.mock('pg', () => {
  const Client = vi.fn()
  Client.prototype.connect = vi.fn()
  Client.prototype.query = vi.fn()
  Client.prototype.end = vi.fn()

  return { Client }
})

vi.mock('./handlers.js', () => {
  return {
    success: vi.fn(),
    failure: vi.fn(),
  }
})

describe('get a list of todo items', () => {
  let client

  beforeEach(() => {
    client = new Client()
  })

  afterEach(() => {
    vi.clearAllMocks()
  })

  it('should return items successfully', async () => {
    client.query.mockResolvedValueOnce({ rows: [], rowCount: 0 })

    await getTodos()

    expect(client.connect).toBeCalledTimes(1)
    expect(client.query).toBeCalledWith('SELECT * FROM todos;')
    expect(client.end).toBeCalledTimes(1)

    expect(success).toBeCalledWith({
      message: '0 item(s) returned',
      data: [],
      status: true,
    })
  })

  it('should throw an error', async () => {
    const mError = new Error('Unable to retrieve rows')
    client.query.mockRejectedValueOnce(mError)

    await getTodos()

    expect(client.connect).toBeCalledTimes(1)
    expect(client.query).toBeCalledWith('SELECT * FROM todos;')
    expect(client.end).toBeCalledTimes(1)
    expect(failure).toBeCalledWith({ message: mError, status: false })
  })
})
```

## 文件系统

文件系统模拟文件系统可确保测试不依赖于实际文件系统，从而使测试更可靠、更可预测。这种隔离有助于避免先前测试的副作用。它允许测试可能难以或无法用实际文件系统复制的错误条件和边缘情况，如权限问题、磁盘满的情况或读/写错误。

Vitest 并不提供任何文件系统模拟 API。您可以使用 `vi.mock` 手动模拟 `fs` 模块，但这很难维护。相反，我们建议使用 [`memfs`](https://www.npmjs.com/package/memfs) 来为你做这件事。`memfs` 创建了一个内存文件系统，可以在不接触实际磁盘的情况下模拟文件系统操作。这种方法既快速又安全，可以避免对真实文件系统产生任何潜在的副作用。

### 例子

要自动将每个 `fs` 调用重定向到 `memfs`，可以在项目根目录下创建 `__mocks__/fs.cjs` 和 `__mocks__/fs/promises.cjs` 文件：

::: code-group
```ts [__mocks__/fs.cjs]
// we can also use `import`, but then
// every export should be explicitly defined

const { fs } = require('memfs')
module.exports = fs
```

```ts [__mocks__/fs/promises.cjs]
// we can also use `import`, but then
// every export should be explicitly defined

const { fs } = require('memfs')
module.exports = fs.promises
```
:::

```ts
// read-hello-world.js
import { readFileSync } from 'node:fs'

export function readHelloWorld(path) {
  return readFileSync(path)
}
```

```ts
import { fs, vol } from 'memfs'
// hello-world.test.js
import { beforeEach, expect, it, vi } from 'vitest'
import { readHelloWorld } from './read-hello-world.js'

// tell vitest to use fs mock from __mocks__ folder
// this can be done in a setup file if fs should always be mocked
vi.mock('node:fs')
vi.mock('node:fs/promises')

beforeEach(() => {
  // reset the state of in-memory fs
  vol.reset()
})

it('should return correct text', () => {
  const path = '/hello-world.txt'
  fs.writeFileSync(path, 'hello world')

  const text = readHelloWorld(path)
  expect(text).toBe('hello world')
})

it('can return a value multiple times', () => {
  // you can use vol.fromJSON to define several files
  vol.fromJSON(
    {
      './dir1/hw.txt': 'hello dir1',
      './dir2/hw.txt': 'hello dir2',
    },
    // default cwd
    '/tmp',
  )

  expect(readHelloWorld('/tmp/dir1/hw.txt')).toBe('hello dir1')
  expect(readHelloWorld('/tmp/dir2/hw.txt')).toBe('hello dir2')
})
```

## 请求

因为 Vitest 运行在 Node 环境中，所以模拟网络请求是一件非常棘手的事情；由于没有办法使用 Web API，因此我们需要一些可以为我们模拟网络行为的包。推荐使用 [Mock Service Worker](https://mswjs.io/) 来进行这个操作。它可以模拟 `REST` 和 `GraphQL` 网络请求，并且与框架无关。

Mock Service Worker (MSW) 的工作原理是拦截测试请求，让我们可以在不更改任何应用代码的情况下使用它。在浏览器中，它使用 [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) 。在 Node.js 和 Vitest 中，它使用 [`@mswjs/interceptors`](https://github.com/mswjs/interceptors) 库。要了解有关 MSW 的更多信息，请阅读他们的 [introduction](https://mswjs.io/docs/) 。

### 配置

您可以像下面一样在您的 [setup file](/config/#setupfiles)

```js
import { graphql, http, HttpResponse } from 'msw'
import { setupServer } from 'msw/node'
import { afterAll, afterEach, beforeAll } from 'vitest'

const posts = [
  {
    userId: 1,
    id: 1,
    title: 'first post title',
    body: 'first post body',
  },
  // ...
]

export const restHandlers = [
  http.get('https://rest-endpoint.example/path/to/posts', () => {
    return HttpResponse.json(posts)
  }),
]

const graphqlHandlers = [
  graphql.query('ListPosts', () => {
    return HttpResponse.json({
      data: { posts },
    })
  }),
]

const server = setupServer(...restHandlers, ...graphqlHandlers)

// 在所有测试之前启动服务器
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))

// 所有测试后关闭服务器
afterAll(() => server.close())

// 每次测试后重置处理程序 `对测试隔离很重要`
afterEach(() => server.resetHandlers())
```

> 使用 `onUnhandleRequest: 'error'` 配置服务器可以确保即使某个请求没有相应的请求处理程序，也会抛出错误。

### 了解更多

MSW 能做的还有很多。你可以访问 cookie 和查询参数、定义模拟错误响应等等！要查看你可以使用 MSW 做什么，请阅读 [their documentation](https://mswjs.io/docs).

## 计时器

每当测试代码涉及到 timeout 或者 interval 时，并不是让我们的测试程序进行等待或者超时。我们也可以通过模拟对 `setTimeout` 和 `setInterval` 的调用来使用 "fake" 计时器来加速测试。

有关更深入的详细 API 描述，参阅 [`vi.usefaketimers` api 部分](/api/vi#vi-usefaketimers)。

### 示例

```js
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest'

function executeAfterTwoHours(func) {
  setTimeout(func, 1000 * 60 * 60 * 2) // 2小时
}

function executeEveryMinute(func) {
  setInterval(func, 1000 * 60) // 1分钟
}

const mock = vi.fn(() => console.log('executed'))

describe('delayed execution', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })
  afterEach(() => {
    vi.restoreAllMocks()
  })
  it('should execute the function', () => {
    executeAfterTwoHours(mock)
    vi.runAllTimers()
    expect(mock).toHaveBeenCalledTimes(1)
  })
  it('should not execute the function', () => {
    executeAfterTwoHours(mock)
    // 前进2毫秒并不会触发方法
    vi.advanceTimersByTime(2)
    expect(mock).not.toHaveBeenCalled()
  })
  it('should execute every minute', () => {
    executeEveryMinute(mock)
    vi.advanceTimersToNextTimer()
    expect(mock).toHaveBeenCalledTimes(1)
    vi.advanceTimersToNextTimer()
    expect(mock).toHaveBeenCalledTimes(2)
  })
})
```

## Classes

您只需调用一次 `vi.fn` 就能模拟整个类，因为所有的类也都是函数，所以这种方法开箱即用。请注意，目前 Vitest 并不尊重 `new` 关键字，因此在函数的主体中，`new.target` 总是 `undefined`。

```ts
class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  static getType(): string {
    return 'animal'
  }

  speak(): string {
    return 'bark!'
  }

  isHungry() {}
  feed() {}
}
```

我们可以使用 ES5 函数重新创建这个类：

```ts
const Dog = vi.fn(function (name) {
  this.name = name
})

// notice that static methods are mocked directly on the function,
// not on the instance of the class
Dog.getType = vi.fn(() => 'mocked animal')

// mock the "speak" and "feed" methods on every instance of a class
// all `new Dog()` instances will inherit these spies
Dog.prototype.speak = vi.fn(() => 'loud bark!')
Dog.prototype.feed = vi.fn()
```

::: tip WHEN TO USE?
一般来说，如果类是从另一个模块重新导出的，你会在模块工厂内重新创建这样的类：

```ts
import { Dog } from './dog.js'

vi.mock(import('./dog.js'), () => {
  const Dog = vi.fn()
  Dog.prototype.feed = vi.fn()
  // ... other mocks
  return { Dog }
})
```

该方法也可用于将一个类的实例传递给接受相同接口的函数：

```ts
// ./src/feed.ts
// ./tests/dog.test.ts
import { expect, test, vi } from 'vitest'
import { feed } from '../src/feed.js'

function feed(dog: Dog) {
  // ...
}

const Dog = vi.fn()
Dog.prototype.feed = vi.fn()

test('can feed dogs', () => {
  const dogMax = new Dog('Max')

  feed(dogMax)

  expect(dogMax.feed).toHaveBeenCalled()
  expect(dogMax.isHungry()).toBe(false)
})
```
:::

现在，当我们创建一个新的 `Dog` 类实例时，它的 `speak` 方法（与 `feed` 并列）已经被模拟：

```ts
const dog = new Dog('Cooper')
dog.speak() // loud bark!

// you can use built-in assertions to check the validity of the call
expect(dog.speak).toHaveBeenCalled()
```

我们可以为特定实例重新分配返回值：

```ts
const dog = new Dog('Cooper')

// "vi.mocked" is a type helper, since
// TypeScript doesn't know that Dog is a mocked class,
// it wraps any function in a MockInstance<T> type
// without validating if the function is a mock
vi.mocked(dog.speak).mockReturnValue('woof woof')

dog.speak() // woof woof
```

要模拟属性，我们可以使用 `vi.spyOn(dog, 'name', 'get')` 方法。这样就可以在被模拟的属性上使用 spy 断言：

```ts
const dog = new Dog('Cooper')

const nameSpy = vi.spyOn(dog, 'name', 'get').mockReturnValue('Max')

expect(dog.name).toBe('Max')
expect(nameSpy).toHaveBeenCalledTimes(1)
```

::: tip
您还可以使用相同的方法监视获取器和设置器。
:::

## 备忘单

::: info 提示
下列示例中的 `vi` 是直接从 `vitest` 导入的。如果在你的 [config](/config/) 中将 `globals` 设置为 `true`，则可以全局使用它。
:::

我想…

### 模拟导出变量
```js
// some-path.js
export const getter = 'variable'
```

```ts
// some-path.test.ts
import * as exports from './some-path.js'

vi.spyOn(exports, 'getter', 'get').mockReturnValue('mocked')
```

### 监听模块导出 setter/getter

```ts
import * as exports from 'some-path'
vi.spyOn(exports, 'getter', 'get')
vi.spyOn(exports, 'setter', 'set')
```

### 模拟模块导出函数

1. `vi.mock` 的示例：

::: warning
不要忘记将 `vi.mock` 调用提升到文件顶部。它将始终在所有导入之前执行。
:::

```ts
// ./some-path.js
export function method() {}
```

```ts
import { method } from './some-path.js'
vi.mock('./some-path.js', () => ({
  method: vi.fn(),
}))
```

2. `vi.spyOn` 的示例：

```ts
import * as exports from './some-path.js'
vi.spyOn(exports, 'method').mockImplementation(() => {})
```

### 模拟模块导出类实现

1. `vi.mock` 和 `.prototype` 的示例:

```ts
// ./some-path.ts
export class SomeClass {}
```

```ts
import { SomeClass } from './some-path.js'

vi.mock(import('./some-path.js'), () => {
  const SomeClass = vi.fn()
  SomeClass.prototype.someMethod = vi.fn()
  return { SomeClass }
})
// SomeClass.mock.instances 上将会有 someMethod 方法
```

2. `vi.spyOn` 的示例:

```ts
import * as mod from './some-path.js'

const SomeClass = vi.fn()
SomeClass.prototype.someMethod = vi.fn()

vi.spyOn(mod, 'SomeClass').mockImplementation(SomeClass)
```

### 监听一个函数是否返回了一个对象

1. 使用 cache 的示例:

```ts
// some-path.ts
export function useObject() {
  return { method: () => true }
}
```

```ts
// useObject.js
import { useObject } from './some-path.js'

const obj = useObject()
obj.method()
```

```ts
// useObject.test.js
import { useObject } from './some-path.js'

vi.mock(import('./some-path.js'), () => {
  let _cache
  const useObject = () => {
    if (!_cache) {
      _cache = {
        method: vi.fn(),
      }
    }
    // 现在每次调用 useObject() 后，都会
    // 返回相同的对象引用
    return _cache
  }
  return { useObject }
})

const obj = useObject()
// obj.method 在 some-path 内调用
expect(obj.method).toHaveBeenCalled()
```

### 模拟部分 module

```ts
import { mocked, original } from './some-path.js'

vi.mock(import('./some-path.js'), async (importOriginal) => {
  const mod = await importOriginal()
  return {
    ...mod,
    mocked: vi.fn(),
  }
})
original() // 有原始的行为
mocked() // 是一个 spy 函数
```

::: warning
别忘了，这只是 [mocks _external_ access](#mocking-pitfalls)。在本例中，如果 `original` 在内部调用 `mocked`，它将始终调用模块中定义的函数，而不是 mock 工厂中的函数。
:::

### 模拟当前日期

要模拟 `Date` 的时间，你可以使用 `vi.setSystemTime` 辅助函数。 该值将**不会**在不同的测试之间自动重置。

请注意，使用 `vi.useFakeTimers` 也会更改 `Date` 的时间。

```ts
const mockDate = new Date(2022, 0, 1)
vi.setSystemTime(mockDate)
const now = new Date()
expect(now.valueOf()).toBe(mockDate.valueOf())
// 重置模拟的时间
vi.useRealTimers()
```

### 模拟全局变量

你可以通过为 `globalThis` 赋值或使用 [`vi.stubGlobal`](/api/vi#vi-stubglobal) 助手来设置全局变量。 使用 `vi.stubGlobal` 时，**不会**在不同的测试之间自动重置，除非你启用 [`unstubGlobals`](/config/#unstubglobals) 配置选项或调用 [`vi.unstubAllGlobals`](/api/vi#vi-unstuballglobals)。

```ts
vi.stubGlobal('__VERSION__', '1.0.0')
expect(__VERSION__).toBe('1.0.0')
```

### 模拟 `import.meta.env`

1. 要更改环境变量，你只需为其分配一个新值即可。 该值将**不会**在不同的测试之间自动重置。

::: warning
环境变量值将在不同的测试之间**不会**自动重置。
:::

```ts
import { beforeEach, expect, it } from 'vitest'

// 你可以在 beforeEach 钩子里手动重置
const originalViteEnv = import.meta.env.VITE_ENV

beforeEach(() => {
  import.meta.env.VITE_ENV = originalViteEnv
})

it('changes value', () => {
  import.meta.env.VITE_ENV = 'staging'
  expect(import.meta.env.VITE_ENV).toBe('staging')
})
```

2. 如果你想自动重置值，可以使用启用了 [`unstubEnvs`](/config/#unstubEnvs) 配置选项的 `vi.stubEnv` 助手（或调用 [`vi.unstubAllEnvs`](/api/vi#vi-unstuballenvs) 在 `beforeEach` 钩子中手动执行）：

```ts
import { expect, it, vi } from 'vitest'

// 在运行测试之前， "VITE_ENV" 的值是 "test"
import.meta.env.VITE_ENV === 'test'

it('changes value', () => {
  vi.stubEnv('VITE_ENV', 'staging')
  expect(import.meta.env.VITE_ENV).toBe('staging')
})

it('the value is restored before running an other test', () => {
  expect(import.meta.env.VITE_ENV).toBe('test')
})
```

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    unstubEnvs: true,
  },
})
```
