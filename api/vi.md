---
outline: deep
---

# Vi

Vitest 通过其 `vi` 辅助工具提供实用功能来帮助你。可以全局访问它（当启用 [globals 配置](/config/#globals) 时），也可以直接从 `vitest` 中导入：

```js
import { vi } from 'vitest'
```

## Mock Modules

本节介绍在 [模拟模块](/guide/mocking#modules) 时可以使用的 API。请注意，Vitest 不支持模拟使用 `require()` 导入的模块。

### vi.mock

- **类型**: `(path: string, factory?: MockOptions | ((importOriginal: () => unknown) => unknown)) => void`
- **类型**: `<T>(path: Promise<T>, factory?: MockOptions | ((importOriginal: () => T) => T | Promise<T>)) => void`

用另一个模块替换提供的 `path` 中的所有导入模块。我们可以在路径内使用配置的 Vite 别名。对 `vi.mock` 的调用是悬挂式的，因此在何处调用并不重要。它总是在所有导入之前执行。如果需要在其作用域之外引用某些变量，可以在 [`vi.hoisted`](/api/vi#vi-hoisted)中定义它们，并在 `vi.mock` 中引用它们。

::: warning
`vi.mock` 仅对使用 `import` 关键字导入的模块有效。它对 `require` 无效。

为了提升 `vi.mock` ，Vitest 会静态分析文件。它会指出不能使用未直接从 `vitest` 软件包导入的 `vi` （例如，从某个实用程序文件导入）。使用 `vi.mock` 与从 `vitest` 导入的 `vi` 一起使用，或者启用 [`globals`](/config/#globals) 配置选项。

Vitest 不会模拟 [setup file](/config/#setupfiles) 中导入的模块，因为这些模块在运行测试文件时已被缓存。我们可以在 [`vi.hoisted`](#vi-hoisted) 中调用 [`vi.resetModules()`](#vi-resetmodules) ，在运行测试文件前清除所有模块缓存。
:::


如果定义了 `factory` 函数，所有导入都将返回其结果。Vitest 只调用一次 factory，并缓存所有后续导入的结果，直到 [`vi.unmock`](#vi-unmock) 或 [`vi.doUnmock`](#vi-dounmock) 被调用。

与 `jest` 不同，工厂可以是异步的。你可以使用 [`vi.importActual`](#vi-importactual)，或将工厂作为第一个参数传递的助手，并在其中获取原始模块。

自 Vitest 2.1 起，您也可以用 `spy` 属性代替工厂函数来提供对象。如果 `spy` 属性为 `true`，Vitest 会像往常一样自动锁定模块，但不会覆盖导出的实现。如果您只想断言导出的方法被另一个方法正确调用，这将非常有用。

```ts
import { calculator } from './src/calculator.ts'

vi.mock('./src/calculator.ts', { spy: true })

// calls the original implementation,
// but allows asserting the behaviour later
const result = calculator(1, 2)

expect(result).toBe(3)
expect(calculator).toHaveBeenCalledWith(1, 2)
expect(calculator).toHaveReturned(3)
```
Vitest 还在 `vi.mock` 和 `vi.doMock` 方法中支持 module promise 而非字符串，以获得更好的集成开发环境支持。当文件被移动时，路径会被更新，`importOriginal` 也会自动继承类型。使用此签名还将强制工厂返回类型与原始模块兼容（但每次导出都是可选的）。

```ts twoslash
// @filename: ./path/to/module.js
export declare function total(...numbers: number[]): number
// @filename: test.js
import { vi } from 'vitest'
// ---cut---
vi.mock(import('./path/to/module.js'), async (importOriginal) => {
  const mod = await importOriginal() // type is inferred
  //    ^?
  return {
    ...mod,
    // replace some exports
    total: vi.fn(),
  }
})
```

在此钩子下，Vitest 仍然对字符串而不是模块对象进行操作。

如果你使用的 TypeScript 在 `tsconfig.json` 中配置了 `paths` 别名，编译器将无法正确解析导入类型。
为了使其正常工作，请确保将所有别名导入替换为相应的相对路径。
例如，使用 `import('./path/to/module.js')`，而不是 `import('@/module')`。

::: warning
`vi.mock` 被提升（换句话说，_移动_）到**文件的顶部**。这意味着无论何时写入它（无论是在 `beforeEach` 还是 `test`），它都会在此之前被调用。

这也意味着不能在 factory 内部使用任何在 factory 外部定义的变量。

如果需要在 factory 内部使用变量，请尝试 [`vi.doMock`](#vi-domock) 。它以同样的方式工作，但不会被吊起。请注意，它只能模拟后续的导入。

如果在 `vi.mock` 之前声明了 `vi.hoisted` 方法，也可以引用该方法定义的变量：

```ts
import { namedExport } from './path/to/module.js'

const mocks = vi.hoisted(() => {
  return {
    namedExport: vi.fn(),
  }
})

vi.mock('./path/to/module.js', () => {
  return {
    namedExport: mocks.namedExport,
  }
})

vi.mocked(namedExport).mockReturnValue(100)

expect(namedExport()).toBe(100)
expect(namedExport).toBe(mocks.namedExport)
```

:::

::: warning
如果我们模拟的模块有默认导出，则需要在返回的工厂函数对象中提供一个 `default` 键。这是 ES 模块特有的注意事项；因此，由于 `jest` 使用 CommonJS 模块，`jest` 文档可能会有所不同。例如：

```ts
vi.mock('./path/to/module.js', () => {
  return {
    default: { myDefaultKey: vi.fn() },
    namedExport: vi.fn(),
    // etc...
  }
})
```

:::
如果要模拟的文件旁边有一个 `__mocks__` 文件夹，且没有提供工厂，Vitest 将尝试在 `__mocks__` 子文件夹中找到一个同名文件，并将其作为实际模块使用。如果模拟的是依赖关系，Vitest 会尝试在项目的 [root](/config/#root)（默认为 `process.cwd()` ）中找到 `__mocks__` 文件夹。我们可以通过 [`deps.moduleDirectories`](/config/#deps-moduledirectories) 配置选项告诉 Vitest 依赖项的位置。

例如，我们有这样的文件结构：

```
- __mocks__
  - axios.js
- src
  __mocks__
    - increment.js
  - increment.js
- tests
  - increment.test.js
```

如果在没有提供工厂或选项的测试文件中调用 `vi.mock` ，它会在 `__mocks__` 文件夹中找到一个文件作为模块使用：

```ts
// increment.test.js
import { vi } from 'vitest'

// axios is a default export from `__mocks__/axios.js`
import axios from 'axios'

// increment is a named export from `src/__mocks__/increment.js`
import { increment } from '../increment.js'

vi.mock('axios')
vi.mock('../increment.js')

axios.get(`/apples/${increment(1)}`)
```

::: warning

请注意，如果不调用 `vi.mock` ，模块**不会**被自动模拟。要复制 Jest 的自动锁定行为，可以在 [`setupFiles`](/config/#setupfiles) 中为每个所需的模块调用 `vi.mock` 。
:::

如果没有提供 `__mocks__` 文件夹或工厂，Vitest 将导入原始模块并自动模拟其所有输出。有关应用的规则，请参阅[模块](/guide/mocking#%E6%A8%A1%E5%9D%97)。

### vi.doMock

- **类型**: `(path: string, factory?: MockOptions | ((importOriginal: () => unknown) => unknown)) => void`
- **类型**: `<T>(path: Promise<T>, factory?: MockOptions | ((importOriginal: () => T) => T | Promise<T>)) => void`

与 [`vi.mock`](#vi-mock) 相同，但它不会被移动到文件顶部，因此我们可以引用全局文件作用域中的变量。模块的下一个 [dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) 将被模拟。

::: warning
这将不会模拟在调用此调用之前导入的模块。不要忘记，ESM 中的所有静态导入都是 [hoaded](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#hoisting)，因此在静态导入前调用此调用不会强制在导入前调用：

```ts
// this will be called _after_ the import statement

import { increment } from './increment.js'
vi.doMock('./increment.js')
```

:::

```ts
// ./increment.js
export function increment(number) {
  return number + 1
}
```

```ts
import { beforeEach, test } from 'vitest'
import { increment } from './increment.js'

// the module is not mocked, because vi.doMock is not called yet
increment(1) === 2

let mockedIncrement = 100

beforeEach(() => {
  // you can access variables inside a factory
  vi.doMock('./increment.js', () => ({ increment: () => ++mockedIncrement }))
})

test('importing the next module imports mocked one', async () => {
  // original import WAS NOT MOCKED, because vi.doMock is evaluated AFTER imports
  expect(increment(1)).toBe(2)
  const { increment: mockedIncrement } = await import('./increment.js')
  // new dynamic import returns mocked module
  expect(mockedIncrement(1)).toBe(101)
  expect(mockedIncrement(1)).toBe(102)
  expect(mockedIncrement(1)).toBe(103)
})
```

### vi.mocked

- **类型**: `<T>(obj: T, deep?: boolean) => MaybeMockedDeep<T>`
- **类型**: `<T>(obj: T, options?: { partial?: boolean; deep?: boolean }) => MaybePartiallyMockedDeep<T>`

TypeScript 的类型助手。只返回传入的对象。

当 `partial` 为 `true` 时，它将期望一个 `Partial<T>` 作为返回值。默认情况下，这只会让 TypeScript 认为第一层的值是模拟的。我们可以将 `{ deep: true }` 作为第二个参数传递给 TypeScript，告诉它整个对象都是模拟的（如果实际上是的话）。

```ts
// example.ts
export function add(x: number, y: number): number {
  return x + y
}

export function fetchSomething(): Promise<Response> {
  return fetch('https://vitest.dev/')
}
```

```ts
// example.test.ts
import * as example from './example'

vi.mock('./example')

test('1 + 1 equals 10', async () => {
  vi.mocked(example.add).mockReturnValue(10)
  expect(example.add(1, 1)).toBe(10)
})

test('mock return value with only partially correct typing', async () => {
  vi.mocked(example.fetchSomething).mockResolvedValue(new Response('hello'))
  vi.mocked(example.fetchSomething, { partial: true }).mockResolvedValue({ ok: false })
  // vi.mocked(example.someFn).mockResolvedValue({ ok: false }) // this is a type error
})
```

### vi.importActual

- **类型**: `<T>(path: string) => Promise<T>`

导入模块，绕过模块是否应被模拟的所有检查。如果我们想部分模拟模块，这一点很有用。

```ts
vi.mock('./example.js', async () => {
  const originalModule = await vi.importActual('./example.js')

  return { ...originalModule, get: vi.fn() }
})
```

### vi.importMock

- **类型**: `<T>(path: string) => Promise<MaybeMockedDeep<T>>`

导入模块并模拟其所有属性（包括嵌套属性）。遵循与 [`vi.mock`](#vi-mock) 相同的规则。有关应用的规则，请参阅[模块](/guide/mocking#%E6%A8%A1%E5%9D%97)。

### vi.unmock

- **类型**: `(path: string | Promise<Module>) => void`

从模拟注册表中删除模块。所有导入调用都将返回原始模块，即使该模块之前已被模拟。该调用会被移动到文件顶端，因此只会解除在 `setupFiles` 中定义的模块。

### vi.doUnmock

- **类型**: `(path: string | Promise<Module>) => void`

与 [`vi.unmock`](#vi-unmock) 相同，但不会移动到文件顶端。下一次导入模块时，将导入原始模块而非 mock。这不会解除先前导入的模块。

```ts
// ./increment.js
export function increment(number) {
  return number + 1
}
```

```ts
import { increment } from './increment.js'

// increment is already mocked, because vi.mock is hoisted
increment(1) === 100

// this is hoisted, and factory is called before the import on line 1
vi.mock('./increment.js', () => ({ increment: () => 100 }))

// all calls are mocked, and `increment` always returns 100
increment(1) === 100
increment(30) === 100

// this is not hoisted, so other import will return unmocked module
vi.doUnmock('./increment.js')

// this STILL returns 100, because `vi.doUnmock` doesn't reevaluate a module
increment(1) === 100
increment(30) === 100

// the next import is unmocked, now `increment` is the original function that returns count + 1
const { increment: unmockedIncrement } = await import('./increment.js')

unmockedIncrement(1) === 2
unmockedIncrement(30) === 31
```

### vi.resetModules

- **类型**: `() => Vitest`

通过清除所有模块的缓存来重置模块注册表。这样就可以在重新导入模块时对模块进行重新评估。顶层导入无法重新评估。这可能有助于隔离测试之间存在本地状态冲突的模块。

```ts
import { vi } from 'vitest'

import { data } from './data.js' // Will not get reevaluated beforeEach test

beforeEach(() => {
  vi.resetModules()
})

test('change state', async () => {
  const mod = await import('./some/path.js') // Will get reevaluated
  mod.changeLocalState('new value')
  expect(mod.getLocalState()).toBe('new value')
})

test('module has old state', async () => {
  const mod = await import('./some/path.js') // Will get reevaluated
  expect(mod.getLocalState()).toBe('old value')
})
```

::: warning
不会重置 mock 注册表。要清除 mock 注册表，请使用 [`vi.unmock`](#vi-unmock) 或 [`vi.doUnmock`](#vi-dounmock) 。
:::

### vi.dynamicImportSettled

等待加载所有导入模块。如果有同步调用开始导入一个模块，而如果不这样做就无法等待，那么它就很有用。

```ts
import { expect, test } from 'vitest'

// cannot track import because Promise is not returned
function renderComponent() {
  import('./component.js').then(({ render }) => {
    render()
  })
}

test('operations are resolved', async () => {
  renderComponent()
  await vi.dynamicImportSettled()
  expect(document.querySelector('.component')).not.toBeNull()
})
```

::: tip
如果在动态导入过程中又启动了另一个动态导入，则该方法将等待直到所有动态导入都解决为止。

该方法还将在导入解析后等待下一个 `setTimeout` 跟他挂钩，因此所有同步操作都应在解析时完成。
:::

## 模拟函数和对象

本节介绍如何使用 [method mock](/api/mock) 替换环境变量和全局变量。

### vi.fn

- **类型:** `(fn?: Function) => Mock`

创建函数的监视程序，但也可以不创建监视程序。每次调用函数时，它都会存储调用参数、返回值和实例。此外，我们还可以使用 [methods](/api/mock) 操纵它的行为。
如果没有给出函数，调用 mock 时将返回 `undefined`。

```ts
const getApples = vi.fn(() => 0)

getApples()

expect(getApples).toHaveBeenCalled()
expect(getApples).toHaveReturnedWith(0)

getApples.mockReturnValueOnce(5)

const res = getApples()
expect(res).toBe(5)
expect(getApples).toHaveNthReturnedWith(2, 5)
```

### vi.isMockFunction

- **类型:** `(fn: Function) => boolean`

检查给定参数是否为 mock 函数。如果使用的是 TypeScript ，它还会缩小参数类型的范围。

### vi.clearAllMocks

将对所有 监听(spies) 调用 [`.mockClear()`](/api/mock#mockclear)。这将清除 mock 历史记录，但不会将其重置为默认实现。

### vi.resetAllMocks

将对所有 监听(spies) 调用 [`.mockReset()`](/api/mock#mockreset)。这将清除 mock 历史记录，并将其重置为空函数（将返回 `undefined` ）。

### vi.restoreAllMocks

将对所有 监听(spies) 调用 [`.mockRestore()`](/api/mock#mockrestore)。这将清除 mock 的历史记录，并将其重置为原来的实现。

### vi.spyOn

- **类型:** `<T, K extends keyof T>(object: T, method: K, accessType?: 'get' | 'set') => MockInstance`

创建与 [`vi.fn()`](#vi-fn) 类似的对象的方法或 getter/setter 的监听(spy) 。它会返回一个 [mock 函数](/api/mock) 。

```ts
let apples = 0
const cart = {
  getApples: () => 42,
}

const spy = vi.spyOn(cart, 'getApples').mockImplementation(() => apples)
apples = 1

expect(cart.getApples()).toBe(1)

expect(spy).toHaveBeenCalled()
expect(spy).toHaveReturnedWith(1)
```

::: tip
你可以在 [`afterEach`](/api/#aftereach)（或启用 [`test.restoreMocks`](/config/#restoreMocks) ）中调用 [`vi.restoreAllMocks`](#vi-restoreallmocks) ，将所有方法还原为原始实现。这将还原原始的 [object descriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) ，因此无法更改方法的实现：

```ts
const cart = {
  getApples: () => 42,
}

const spy = vi.spyOn(cart, 'getApples').mockReturnValue(10)

console.log(cart.getApples()) // 10
vi.restoreAllMocks()
console.log(cart.getApples()) // 42
spy.mockReturnValue(10)
console.log(cart.getApples()) // still 42!
```

:::

::: tip
在[浏览器模式](/guide/browser/)下，无法监视导出的方法。相反，你可以通过调用 `vi.mock("./file-path.js", { spy: true })` 来监视每个导出方法。这将模拟每个导出方法，但保留其完整的实现，从而可以断言该方法是否被正确调用。

```ts
import { calculator } from './src/calculator.ts'

vi.mock('./src/calculator.ts', { spy: true })

calculator(1, 2)

expect(calculator).toHaveBeenCalledWith(1, 2)
expect(calculator).toHaveReturned(3)
```

虽然有可能在 `jsdom` 或其他 Node.js 环境中监视导出，但未来可能会发生变化。
:::

### vi.stubEnv {#vi-stubenv}

- **类型:** `<T extends string>(name: T, value: T extends "PROD" | "DEV" | "SSR" ? boolean : string | undefined) => Vitest`

更改 `process.env` 和 `import.meta.env` 中环境变量的值。我们可以调用 `vi.unstubAllEnvs` 恢复其值。

```ts
import { vi } from 'vitest'

// `process.env.NODE_ENV` and `import.meta.env.NODE_ENV`
// are "development" before calling "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', undefined)

process.env.NODE_ENV === undefined
import.meta.env.NODE_ENV === undefined

// doesn't change other envs
import.meta.env.MODE === 'development'
```

:::tip
我们也可以通过简单赋值来更改值，但无法使用 `vi.unstubAllEnvs` 恢复以前的值：

```ts
import.meta.env.MODE = 'test'
```

:::

### vi.unstubAllEnvs {#vi-unstuballenvs}

- **类型:** `() => Vitest`

恢复通过 `vi.stubEnv` 更改的所有 `import.meta.env` 和 `process.env` 值。首次调用时，Vitest 会记住并保存原始值，直到再次调用 `unstubAllEnvs`。

```ts
import { vi } from 'vitest'

// `process.env.NODE_ENV` and `import.meta.env.NODE_ENV`
// are "development" before calling stubEnv

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', 'staging')

process.env.NODE_ENV === 'staging'
import.meta.env.NODE_ENV === 'staging'

vi.unstubAllEnvs()

// restores to the value that were stored before the first "stubEnv" call
process.env.NODE_ENV === 'development'
import.meta.env.NODE_ENV === 'development'
```

### vi.stubGlobal

- **类型:** `(name: string | number | symbol, value: unknown) => Vitest`

更改全局变量的值。我们可以调用 `vi.unstubAllGlobals` 恢复其原始值。

```ts
import { vi } from 'vitest'

// `innerWidth` is "0" before calling stubGlobal

vi.stubGlobal('innerWidth', 100)

innerWidth === 100
globalThis.innerWidth === 100
// if you are using jsdom or happy-dom
window.innerWidth === 100
```

:::tip
我们也可以通过简单地将其赋值给 `globalThis` 或 `window`（如果我们使用的是 `jsdom` 或 `happy-dom` 环境）来更改该值，但无法使用 `vi.unstubAllGlobals` 恢复原始值：

```ts
globalThis.innerWidth = 100
// if you are using jsdom or happy-dom
window.innerWidth = 100
```

:::

### vi.unstubAllGlobals {#vi-unstuballglobals}

- **类型:** `() => Vitest`

恢复 `globalThis` / `global`（和 `window` / `top` / `self` / `parent `，如果我们使用的是 `jsdom` 或 `happy-dom` 环境）上所有被 `vi.stubGlobal` 更改过的全局值。第一次调用时，Vitest 会记住并保存原始值，直到再次调用 `unstubAllGlobals`。

```ts
import { vi } from 'vitest'

const Mock = vi.fn()

// IntersectionObserver is "undefined" before calling "stubGlobal"

vi.stubGlobal('IntersectionObserver', Mock)

IntersectionObserver === Mock
global.IntersectionObserver === Mock
globalThis.IntersectionObserver === Mock
// if you are using jsdom or happy-dom
window.IntersectionObserver === Mock

vi.unstubAllGlobals()

globalThis.IntersectionObserver === undefined
'IntersectionObserver' in globalThis === false
// throws ReferenceError, because it's not defined
IntersectionObserver === undefined
```

## Fake Timers

本节介绍如何使用 [fake timers](/guide/mocking#%E8%AE%A1%E6%97%B6%E5%99%A8) 。

### vi.advanceTimersByTime

- **类型:** `(ms: number) => Vitest`

该方法将调用每个启动的定时器，直到超过指定的毫秒数或队列为空（以先到者为准）。

```ts
let i = 0
setInterval(() => console.log(++i), 50)

vi.advanceTimersByTime(150)

// log: 1
// log: 2
// log: 3
```

### vi.advanceTimersByTimeAsync

- **类型:** `(ms: number) => Promise<Vitest>`

该方法将调用每个已启动的定时器，直到超过指定的毫秒数或队列为空（以先到者为准）。这将包括异步设置的计时器。

```ts
let i = 0
setInterval(() => Promise.resolve().then(() => console.log(++i)), 50)

await vi.advanceTimersByTimeAsync(150)

// log: 1
// log: 2
// log: 3
```

### vi.advanceTimersToNextTimer

- **类型:** `() => Vitest`

将调用下一个可用的定时器。在每次调用定时器之间进行断言非常有用。我们可以调用它来管理自己的定时器。

```ts
let i = 0
setInterval(() => console.log(++i), 50)

vi.advanceTimersToNextTimer() // log: 1
  .advanceTimersToNextTimer() // log: 2
  .advanceTimersToNextTimer() // log: 3
```

### vi.advanceTimersToNextTimerAsync

- **类型:** `() => Promise<Vitest>`

如果定时器是异步设置的，则会调用下一个可用的定时器并等待解决。在每次调用定时器之间进行断言非常有用。

```ts
let i = 0
setInterval(() => Promise.resolve().then(() => console.log(++i)), 50)

await vi.advanceTimersToNextTimerAsync() // log: 1
expect(console.log).toHaveBeenCalledWith(1)

await vi.advanceTimersToNextTimerAsync() // log: 2
await vi.advanceTimersToNextTimerAsync() // log: 3
```

### vi.advanceTimersToNextFrame <Version>2.1.0</Version> {#vi-advancetimerstonextframe}

- **Type:** `() => Vitest`

Similar to [`vi.advanceTimersByTime`](https://vitest.dev/api/vi#vi-advancetimersbytime), but will advance timers by the milliseconds needed to execute callbacks currently scheduled with `requestAnimationFrame`.

```ts
let frameRendered = false

requestAnimationFrame(() => {
  frameRendered = true
})

vi.advanceTimersToNextFrame()

expect(frameRendered).toBe(true)
```

### vi.getTimerCount

- **类型:** `() => number`

获取等待计时器的数量。

### vi.clearAllTimers

删除所有计划运行的计时器。这些定时器今后将不再运行。

### vi.getMockedSystemTime

- **类型**: `() => Date | null`

返回使用 `setSystemTime` 设置的模拟当前日期。如果没有模拟日期，该方法将返回 `null` 。

### vi.getRealSystemTime

- **类型**: `() => number`

使用 `vi.useFakeTimers` 时，会模拟 `Date.now` 调用。如果需要以毫秒为单位获取实时时间，可以调用此函数。

### vi.runAllTicks

- **类型:** `() => Vitest`

调用由 `process.nextTick` 排在队列中的每个微任务。这也将运行所有自己安排的微任务。

### vi.runAllTimers

- **类型:** `() => Vitest`

该方法将调用每个已经启动的定时器，直到定时器队列为空。这意味着在 `runAllTimers` 期间调用的每个定时器都会被触发。如果时间间隔为无限，则会在尝试 10000 次后触发（可使用 [`fakeTimers.loopLimit`](/config/#faketimers-looplimit) 进行配置）。

```ts
let i = 0
setTimeout(() => console.log(++i))
const interval = setInterval(() => {
  console.log(++i)
  if (i === 3) {
    clearInterval(interval)
  }
}, 50)

vi.runAllTimers()

// log: 1
// log: 2
// log: 3
```

### vi.runAllTimersAsync

- **类型:** `() => Promise<Vitest>`

该方法将异步调用每个已启动的定时器，直到定时器队列为空。这意味着在 `runAllTimersAsync` 期间调用的每个定时器都会被触发，即使是异步定时器。如果我们有一个无限的时间间隔、
会在尝试 10000 次后抛出（可使用 [`fakeTimers.loopLimit`](/config/#faketimers-looplimit) ）。

```ts
setTimeout(async () => {
  console.log(await Promise.resolve('result'))
}, 100)

await vi.runAllTimersAsync()

// log: result
```

### vi.runOnlyPendingTimers

- **类型:** `() => Vitest`

此方法将调用 [`vi.useFakeTimers`](#vii-usefaketimers) 调用后启动的所有计时器。它不会调用在调用期间启动的任何计时器。

```ts
let i = 0
setInterval(() => console.log(++i), 50)

vi.runOnlyPendingTimers()

// log: 1
```

### vi.runOnlyPendingTimersAsync

- **类型:** `() => Promise<Vitest>`

此方法将异步调用 [`vi.useFakeTimers`](#vi-usefaketimers) 调用后启动的每个定时器，即使是异步定时器。它不会触发任何在调用期间启动的定时器。

```ts
setTimeout(() => {
  console.log(1)
}, 100)
setTimeout(() => {
  Promise.resolve().then(() => {
    console.log(2)
    setInterval(() => {
      console.log(3)
    }, 40)
  })
}, 10)

await vi.runOnlyPendingTimersAsync()

// log: 2
// log: 3
// log: 3
// log: 1
```

### vi.setSystemTime

- **类型**: `(date: string | number | Date) => void`

如果启用了伪计时器，此方法将模拟用户更改系统时钟（将影响与日期相关的 API，如 `hrtime` 、`performance.now` 或 `new Date()` ），但不会触发任何计时器。如果未启用假定时器，该方法将仅模拟 `Date.*` 调用。

如果我们需要测试任何依赖于当前日期的内容 -- 例如在代码中调用 [luxon](https://github.com/moment/luxon/) --则非常有用。

接受与 `Date` 相同的字符串和数字参数。

```ts
const date = new Date(1998, 11, 19)

vi.useFakeTimers()
vi.setSystemTime(date)

expect(Date.now()).toBe(date.valueOf())

vi.useRealTimers()
```

### vi.useFakeTimers

- **类型:** `(config?: FakeTimerInstallOpts) => Vitest`

要启用模拟定时器，需要调用此方法。在调用 [`vi.useRealTimers()`](#vi-userealtimers) 之前，它将封装所有对定时器的进一步调用（如 `setTimeout` 、`setInterval` 、`clearTimeout` 、`clearInterval` 、`setImmediate` 、`clearImmediate` 和 `Date`）。

在 `node:child_process` 中使用 `--pool=forks` 运行 Vitest 时，不支持模拟 `nextTick` 。NodeJS 在 `node:child_process` 中内部使用了 `process.nextTick` ，当模拟它时会挂起。使用 `--pool=threads` 运行 Vitest 时支持模拟 `nextTick`。

内部实现基于 [`@sinonjs/fake-timers`](https://github.com/sinonjs/fake-timers) 。

::: tip
`vi.useFakeTimers()` 不再自动模拟 `process.nextTick` 。
仍然可以通过在 `toFake` 参数中指定选项来模拟： `vi.useFakeTimers({ toFake: ['nextTick'] })` 。
:::

### vi.isFakeTimers {#vi-isfaketimers}

- **类型:** `() => boolean`

如果启用了假计时器，则返回 `true` 。

### vi.useRealTimers

- **类型:** `() => Vitest`

定时器用完后，可以调用此方法将模拟的定时器返回到其原始实现。之前调度的所有计时器都将被丢弃。

## Miscellaneous

Vitest 提供的一组有用的辅助函数。

### vi.waitFor {#vi-waitfor}

- **类型:** `<T>(callback: WaitForCallback<T>, options?: number | WaitForOptions) => Promise<T>`

等待回调成功执行。如果回调抛出错误或返回拒绝的承诺，它将继续等待，直到成功或超时。

这在需要等待某些异步操作完成时非常有用，例如，在启动服务器并需要等待其启动时。

```ts
import { expect, test, vi } from 'vitest'
import { createServer } from './server.js'

test('Server started successfully', async () => {
  const server = createServer()

  await vi.waitFor(
    () => {
      if (!server.isReady) {
        throw new Error('Server not started')
      }

      console.log('Server started')
    },
    {
      timeout: 500, // default is 1000
      interval: 20, // default is 50
    }
  )
  expect(server.isReady).toBe(true)
})
```

它也适用于异步回调。

```ts
// @vitest-environment jsdom

import { expect, test, vi } from 'vitest'
import { getDOMElementAsync, populateDOMAsync } from './dom.js'

test('Element exists in a DOM', async () => {
  // start populating DOM
  populateDOMAsync()

  const element = await vi.waitFor(
    async () => {
      // try to get the element until it exists
      const element = (await getDOMElementAsync()) as HTMLElement | null
      expect(element).toBeTruthy()
      expect(element.dataset.initialized).toBeTruthy()
      return element
    },
    {
      timeout: 500, // default is 1000
      interval: 20, // default is 50
    }
  )
  expect(element).toBeInstanceOf(HTMLElement)
})
```

如果使用了 `vi.useFakeTimers` , `vi.waitFor` 会在每次检查回调中自动调用 `vi.advanceTimersByTime(interval)` 。

### vi.waitUntil

- **类型:** `<T>(callback: WaitUntilCallback<T>, options?: number | WaitUntilOptions) => Promise<T>`

这与 `vi.waitFor` 类似，但如果回调抛出任何错误，执行将立即中断并收到一条错误信息。如果回调返回虚假值(falsy) ，下一次检查将继续，直到返回真实值(truthy) 。这在需要等待某项内容存在后再执行下一步时非常有用。

请看下面的示例。我们可以使用 `vi.waitUntil` 等待元素出现在页面上，然后对元素进行操作。

```ts
import { expect, test, vi } from 'vitest'

test('Element render correctly', async () => {
  const element = await vi.waitUntil(() => document.querySelector('.element'), {
    timeout: 500, // default is 1000
    interval: 20, // default is 50
  })

  // do something with the element
  expect(element.querySelector('.element-child')).toBeTruthy()
})
```

### vi.hoisted {#vi-hoisted}

- **类型**: `<T>(factory: () => T) => T`

ES 模块中的所有静态 `import` 语句都被提升到文件顶部，因此在导入之前定义的任何代码都将在导入评估之后执行。

不过，在导入模块之前，调用一些副作用（如模拟日期）可能会很有用。

要绕过这一限制，可以像这样将静态导入重写为动态导入：

```diff
callFunctionWithSideEffect()
- import { value } from './some/module.js'
+ const { value } = await import('./some/module.js')
```

运行 `vitest` 时，可以使用 `vi.hoisted` 方法自动完成此操作。

```diff
- callFunctionWithSideEffect()
import { value } from './some/module.js'
+ vi.hoisted(() => callFunctionWithSideEffect())
```

该方法返回从工厂返回的值。 如果我们需要轻松访问本地定义的变量，可以在我们的 `vi.mock` 工厂中使用该值：

```ts
import { expect, vi } from 'vitest'
import { originalMethod } from './path/to/module.js'

const { mockedMethod } = vi.hoisted(() => {
  return { mockedMethod: vi.fn() }
})

vi.mock('./path/to/module.js', () => {
  return { originalMethod: mockedMethod }
})

mockedMethod.mockReturnValue(100)
expect(originalMethod()).toBe(100)
```

请注意，即使我们的环境不支持顶级等待，也可以异步调用此方法：

```ts
const promised = await vi.hoisted(async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts')
  return response.json()
})
```

### vi.setConfig

- **类型**: `RuntimeConfig`

更新当前测试文件的配置。此方法只会影响当前测试文件的配置选项：

```ts
vi.setConfig({
  allowOnly: true,
  testTimeout: 10_000,
  hookTimeout: 10_000,
  clearMocks: true,
  restoreMocks: true,
  fakeTimers: {
    now: new Date(2021, 11, 19),
    // supports the whole object
  },
  maxConcurrency: 10,
  sequence: {
    hooks: 'stack',
    // supports only "sequence.hooks"
  },
})
```

### vi.resetConfig

- **类型**: `RuntimeConfig`

如果之前调用过 [`vi.setConfig`](#vi-setconfig) ，则会将配置重置为原始状态。
