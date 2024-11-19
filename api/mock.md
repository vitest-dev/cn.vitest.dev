# Mock Functions

我们可以使用 `vi.fn` 方法创建一个 mock 函数来跟踪其执行情况。如果要跟踪已创建对象上的方法，可以使用 `vi.spyOn` 方法：

```js
import { vi } from 'vitest'

const fn = vi.fn()
fn('hello world')
fn.mock.calls[0] === ['hello world']

const market = {
  getApples: () => 100,
}

const getApplesSpy = vi.spyOn(market, 'getApples')
market.getApples()
getApplesSpy.mock.calls.length === 1
```

我们应该在 [`expect`](/api/expect) 上使用 mock 断言（例如 [`toHaveBeenCalled`](/api/expect#tohavebeencalled) ）来断言 mock 结果。在这里我们介绍了用于操作 mock 行为的可用属性和方法。

::: tip
The custom function implementation in the types below is marked with a generic `<T>`.
:::

## getMockImplementation

<<<<<<< HEAD
- **类型:** `(...args: any) => any`
=======
```ts
function getMockImplementation(): T | undefined
```
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

返回当前的模拟实现（如果有）。

<<<<<<< HEAD
如果使用 [`vi.fn`](/api/vi#vi-fn) 创建了 mock，它将把向下传递的方法视为 mock 实现。

如果使用 [`vi.spyOn`](/api/vi#vi-spyon) 创建了 mock，除非提供了自定义实现，否则将返回 `undefined` 。

## getMockName

- **类型:** `() => string`

用它来返回使用方法 `.mockName(name)` 给 mock 的名称。

## mockClear

- **类型:** `() => MockInstance`

清除每次调用的所有信息。调用该方法后，`.mock` 上的所有属性都将返回空状态。此方法不会重置实现。如果需要在不同断言之间清理 mock，该方法非常有用。

如果我们希望在每次测试前自动调用该方法，可以在配置中启用 [`clearMocks``](/config/#clearmocks)设置。

## mockName

- **类型:** `(name: string) => MockInstance`

设置内部 mock 名称。用于在断言失败时查看 mock 名称。

## mockImplementation

- **类型:** `(fn: Function) => MockInstance`

接受一个函数，该函数将用作 mock 的实现。
=======
If the mock was created with [`vi.fn`](/api/vi#vi-fn), it will use the provided method as the mock implementation.

If the mock was created with [`vi.spyOn`](/api/vi#vi-spyon), it will return `undefined` unless a custom implementation is provided.

## getMockName

```ts
function getMockName(): string
```

Use it to return the name assigned to the mock with the `.mockName(name)` method. By default, it will return `vi.fn()`.

## mockClear

```ts
function mockClear(): MockInstance<T>
```

Clears all information about every call. After calling it, all properties on `.mock` will return to their initial state. This method does not reset implementations. It is useful for cleaning up mocks between different assertions.

To automatically call this method before each test, enable the [`clearMocks`](/config/#clearmocks) setting in the configuration.

## mockName

```ts
function mockName(name: string): MockInstance<T>
```

Sets the internal mock name. This is useful for identifying the mock when an assertion fails.

## mockImplementation

```ts
function mockImplementation(fn: T): MockInstance<T>
```

Accepts a function to be used as the mock implementation. TypeScript expects the arguments and return type to match those of the original function.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const mockFn = vi.fn().mockImplementation((apples: number) => apples + 1)
// or: vi.fn(apples => apples + 1);

const NelliesBucket = mockFn(0)
const BobsBucket = mockFn(1)

NelliesBucket === 1 // true
BobsBucket === 2 // true

mockFn.mock.calls[0][0] === 0 // true
mockFn.mock.calls[1][0] === 1 // true
```

## mockImplementationOnce

<<<<<<< HEAD
- **类型:** `(fn: Function) => MockInstance`

接受一个函数，该函数将在下一次调用时用作 mock 的实现。可以链式调用多个函数，从而产生不同的结果。
=======
```ts
function mockImplementationOnce(fn: T): MockInstance<T>
```

Accepts a function to be used as the mock implementation. TypeScript expects the arguments and return type to match those of the original function. This method can be chained to produce different results for multiple function calls.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const myMockFn = vi
  .fn()
  .mockImplementationOnce(() => true) // 1st call
  .mockImplementationOnce(() => false) // 2nd call

myMockFn() // 1st call: true
myMockFn() // 2nd call: false
```

<<<<<<< HEAD
当模拟函数的实现耗尽时，它会调用 `vi.fn(() => defaultValue)` 或 `.mockImplementation(() => defaultValue)` 所设置的默认实现（如果它们被调用）：
=======
When the mocked function runs out of implementations, it will invoke the default implementation set with `vi.fn(() => defaultValue)` or `.mockImplementation(() => defaultValue)` if they were called:
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const myMockFn = vi
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call')

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn())
```

## withImplementation

<<<<<<< HEAD
- **类型:** `(fn: Function, callback: () => void) => MockInstance`
- **类型:** `(fn: Function, callback: () => Promise<unknown>) => Promise<MockInstance>`
=======
```ts
function withImplementation(
  fn: T,
  cb: () => void
): MockInstance<T>
function withImplementation(
  fn: T,
  cb: () => Promise<void>
): Promise<MockInstance<T>>
```
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

在执行回调时，临时覆盖原始模拟实现。

```js
const myMockFn = vi.fn(() => 'original')

myMockFn.withImplementation(
  () => 'temp',
  () => {
    myMockFn() // 'temp'
  }
)

myMockFn() // 'original'
```

可与异步回调一起使用。该方法必须等待，之后才能使用原始实现。

```ts
test('async callback', () => {
  const myMockFn = vi.fn(() => 'original')

  // We await this call since the callback is async
  await myMockFn.withImplementation(
    () => 'temp',
    async () => {
      myMockFn() // 'temp'
    }
  )

  myMockFn() // 'original'
})
```

请注意，该方法优先于 [`mockImplementationOnce`](#mockimplementationonce)。

## mockRejectedValue

<<<<<<< HEAD
- **类型:** `(value: any) => MockInstance`
=======
```ts
function mockRejectedValue(value: unknown): MockInstance<T>
```
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

接受在调用 async 函数时将被拒绝的错误。

```ts
const asyncMock = vi.fn().mockRejectedValue(new Error('Async error'))

await asyncMock() // throws Error<'Async error'>
```

## mockRejectedValueOnce

<<<<<<< HEAD
- **类型:** `(value: any) => MockInstance`

接受一个将在下一次函数调用中被剔除的值。如果是链式调用，则每次连续调用都会剔除指定值。
=======
```ts
function mockRejectedValueOnce(value: unknown): MockInstance<T>
```

Accepts a value that will be rejected during the next function call. If chained, each consecutive call will reject the specified value.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const asyncMock = vi
  .fn()
  .mockResolvedValueOnce('first call')
  .mockRejectedValueOnce(new Error('Async error'))

await asyncMock() // 'first call'
await asyncMock() // throws Error<'Async error'>
```

## mockReset

<<<<<<< HEAD
- **类型:** `() => MockInstance`

执行 `mockClear` 的功能，并使内部实现成为空函数（调用时返回 `undefined` ）。这也会重置所有 "once" 实现。当我们想将 mock 完全重置为默认状态时，这很有用。

如果我们希望在每次测试前自动调用该方法，可以启用配置中的 [`mockReset`](/config/#mockreset) 设置。

## mockRestore

- **类型:** `() => MockInstance`

执行与 `mockReset` 相同的功能，并将内部实现还原为原始函数。

请注意，从 `vi.fn()` 恢复 mock 会将实现设置为返回 `undefined` 的空函数。还原 `vi.fn(impl)` 会将实现还原为 `impl`。

如果希望在每次测试前自动调用此方法，可以启用配置中的 [`restoreMocks`](/config/#restoreMocks) 设置。

## mockResolvedValue

- **类型:** `(value: any) => MockInstance`

接受将在调用 async 函数时解析的值。
=======
```ts
function mockReset(): MockInstance<T>
```

Performs the same actions as `mockClear` and sets the inner implementation to an empty function (returning `undefined` when invoked). This also resets all "once" implementations. It is useful for completely resetting a mock to its default state.

To automatically call this method before each test, enable the [`mockReset`](/config/#mockreset) setting in the configuration.

## mockRestore

```ts
function mockRestore(): MockInstance<T>
```

Performs the same actions as `mockReset` and restores the inner implementation to the original function.

Note that restoring a mock created with `vi.fn()` will set the implementation to an empty function that returns `undefined`. Restoring a mock created with `vi.fn(impl)` will restore the implementation to `impl`.

To automatically call this method before each test, enable the [`restoreMocks`](/config/#restoremocks) setting in the configuration.

## mockResolvedValue

```ts
function mockResolvedValue(value: Awaited<ReturnType<T>>): MockInstance<T>
```

Accepts a value that will be resolved when the async function is called. TypeScript will only accept values that match the return type of the original function.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const asyncMock = vi.fn().mockResolvedValue(42)

await asyncMock() // 42
```

## mockResolvedValueOnce

<<<<<<< HEAD
- **类型:** `(value: any) => MockInstance`

接受一个将在下一次函数调用时解析的值。如果是链式调用，则每次连续调用都会解析指定的值。
=======
```ts
function mockResolvedValueOnce(value: Awaited<ReturnType<T>>): MockInstance<T>
```

Accepts a value that will be resolved during the next function call. TypeScript will only accept values that match the return type of the original function. If chained, each consecutive call will resolve the specified value.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const asyncMock = vi
  .fn()
  .mockResolvedValue('default')
  .mockResolvedValueOnce('first call')
  .mockResolvedValueOnce('second call')

await asyncMock() // first call
await asyncMock() // second call
await asyncMock() // default
await asyncMock() // default
```

## mockReturnThis

<<<<<<< HEAD
- **类型:** `() => MockInstance`

如果需要在不调用实际实现的情况下从方法中返回 `this` 上下文，请使用此参数。这是一个简写：
=======
```ts
function mockReturnThis(): MockInstance<T>
```

Use this if you need to return the `this` context from the method without invoking the actual implementation. This is a shorthand for:
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
spy.mockImplementation(function () {
  return this
})
```

## mockReturnValue

<<<<<<< HEAD
- **类型:** `(value: any) => MockInstance`

接受一个值，该值将在调用 mock 函数时返回。
=======
```ts
function mockReturnValue(value: ReturnType<T>): MockInstance<T>
```

Accepts a value that will be returned whenever the mock function is called. TypeScript will only accept values that match the return type of the original function.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const mock = vi.fn()
mock.mockReturnValue(42)
mock() // 42
mock.mockReturnValue(43)
mock() // 43
```

## mockReturnValueOnce

<<<<<<< HEAD
- **类型:** `(value: any) => MockInstance`

接受将在下一次函数调用中返回的值。如果是链式调用，则每次连续调用都会返回指定值。

如果没有更多的 `mockReturnValueOnce` 值可使用，mock 将返回到预先定义的实现（如果有的话）。
=======
```ts
function mockReturnValueOnce(value: ReturnType<T>): MockInstance<T>
```

Accepts a value that will be returned whenever the mock function is called. TypeScript will only accept values that match the return type of the original function.

When the mocked function runs out of implementations, it will invoke the default implementation set with `vi.fn(() => defaultValue)` or `.mockImplementation(() => defaultValue)` if they were called:
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```ts
const myMockFn = vi
  .fn()
  .mockReturnValue('default')
  .mockReturnValueOnce('first call')
  .mockReturnValueOnce('second call')

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn())
```

## mock.calls

<<<<<<< HEAD
这是一个数组，包含每次调用的所有参数。数组中的一项就是该调用的参数。
=======
```ts
const calls: Parameters<T>[]
```

This is an array containing all arguments for each call. One item of the array is the arguments of that call.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```js
const fn = vi.fn()

fn('arg1', 'arg2')
fn('arg3')

fn.mock.calls
=== [
  ['arg1', 'arg2'], // first call
  ['arg3'], // second call
]
```

## mock.lastCall

<<<<<<< HEAD
其中包含最后一次调用的参数。如果 mock 未被调用，将返回 `undefined`。

## mock.results

这是一个数组，包含函数 `returned` 的所有值。数组中的一项是一个具有 `type` 和 `value` 属性的对象。可用的类型有:
=======
```ts
const lastCall: Parameters<T> | undefined
```

This contains the arguments of the last call. If mock wasn't called, it will return `undefined`.

## mock.results

```ts
interface MockResultReturn<T> {
  type: 'return'
  /**
   * The value that was returned from the function.
   * If function returned a Promise, then this will be a resolved value.
   */
  value: T
}

interface MockResultIncomplete {
  type: 'incomplete'
  value: undefined
}

interface MockResultThrow {
  type: 'throw'
  /**
   * An error that was thrown during function execution.
   */
  value: any
}

type MockResult<T> =
  | MockResultReturn<T>
  | MockResultThrow
  | MockResultIncomplete

const results: MockResult<ReturnType<T>>[]
```

This is an array containing all values that were `returned` from the function. One item of the array is an object with properties `type` and `value`. Available types are:
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

- `'return'` - 函数返回时没有抛出。
- `'throw'` - 函数抛出了一个值。


`value` 属性包含返回值或抛出的错误。如果函数返回一个 `Promise`，那么即使Promise rejected，`result` 也将始终为 `'return'`。

```js
const fn = vi
  .fn()
  .mockReturnValueOnce('result')
  .mockImplementationOnce(() => {
    throw new Error('thrown error')
  })

const result = fn() // returned 'result'

try {
  fn() // threw Error
}
catch {}

fn.mock.results
=== [
  // first result
  {
    type: 'return',
    value: 'result',
  },
  // last result
  {
    type: 'throw',
    value: Error,
  },
]
```

## mock.settledResults

<<<<<<< HEAD
包含函数中`resolved` 或 `rejected` 的所有值的数组。
=======
```ts
interface MockSettledResultFulfilled<T> {
  type: 'fulfilled'
  value: T
}

interface MockSettledResultRejected {
  type: 'rejected'
  value: any
}

export type MockSettledResult<T> =
  | MockSettledResultFulfilled<T>
  | MockSettledResultRejected

const settledResults: MockSettledResult<Awaited<ReturnType<T>>>[]
```

An array containing all values that were `resolved` or `rejected` from the function.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

如果函数从未`resolved` 或 `rejected` ，则此数组将为空。

```js
const fn = vi.fn().mockResolvedValueOnce('result')

const result = fn()

fn.mock.settledResults === []

await result

fn.mock.settledResults === [
  {
    type: 'fulfilled',
    value: 'result',
  },
]
```

## mock.invocationCallOrder

<<<<<<< HEAD
此属性返回 mock 函数的执行顺序。它是一个数字数组，由所有定义的 mock 共享。
=======
```ts
const invocationCallOrder: number[]
```

This property returns the order of the mock function's execution. It is an array of numbers that are shared between all defined mocks.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```js
const fn1 = vi.fn()
const fn2 = vi.fn()

fn1()
fn2()
fn1()

fn1.mock.invocationCallOrder === [1, 3]
fn2.mock.invocationCallOrder === [2]
```

## mock.contexts

<<<<<<< HEAD
此属性是每次调用模拟函数时使用的 `this` 值的数组。
=======
```ts
const contexts: ThisParameterType<T>[]
```

This property is an array of `this` values used during each call to the mock function.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

```js
const fn = vi.fn()
const context = {}

fn.apply(context)
fn.call(context)

fn.mock.contexts[0] === context
fn.mock.contexts[1] === context
```

## mock.instances

<<<<<<< HEAD
此属性是一个数组，其中包含使用 `new` 关键字调用模拟时创建的所有实例。请注意，这是函数的实际上下文（`this`），而不是返回值。
=======
```ts
const instances: ReturnType<T>[]
```

This property is an array containing all instances that were created when the mock was called with the `new` keyword. Note that this is an actual context (`this`) of the function, not a return value.
>>>>>>> a3d5ec1a9c68302fc48da99589891ca4491e4d1f

::: warning
如果使用 `new MyClass()` 对 mock 进行实例化，那么 `mock.instances` 将是一个只有一个值的数组：

```js
const MyClass = vi.fn()
const a = new MyClass()

MyClass.mock.instances[0] === a
```

如果从构造函数返回一个值，该值不会出现在 `instances` 数组中，而是会出现在 `results` 中：

```js
const Spy = vi.fn(() => ({ method: vi.fn() }))
const a = new Spy()

Spy.mock.instances[0] !== a
Spy.mock.results[0] === a
```

:::
