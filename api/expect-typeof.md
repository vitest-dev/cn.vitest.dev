# expectTypeOf

- **类型:** `<T>(a: unknown) => ExpectTypeOf`

::: warning
在运行时，此函数不执行任何操作。要 [enable typechecking](/guide/testing-types#run-typechecking)，不要忘记传递 `--typecheck` 标志。
:::

## not

- **类型:** `ExpectTypeOf`

  你可以使用 `.not` 属性否定所有断言。

## toEqualTypeOf

- **类型:** `<T>(expected: T) => void`

  该匹配器将检查类型是否完全相同。如果两个对象具有不同的值但类型相同，则此匹配器不会失败。但是，如果对象缺少属性，它将失败。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: number }>()
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 1 })
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 2 })
expectTypeOf({ a: 1, b: 1 }).not.toEqualTypeOf<{ a: number }>()
```

## toMatchTypeOf

- **类型:** `<T>(expected: T) => void`

此匹配器检查期望类型是否扩展了提供的类型。它不同于 `toEqual`，更类似于 [expect's](/api/expect) `toMatchObject()`。使用此匹配器，你可以检查对象是否“匹配”类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf({ a: 1, b: 1 }).toMatchTypeOf({ a: 1 })
expectTypeOf<number>().toMatchTypeOf<string | number>()
expectTypeOf<string | number>().not.toMatchTypeOf<number>()
```

## extract

- **类型:** `ExpectTypeOf<ExtractedUnion>`

你可以使用 `.extract` 来缩小类型以进行进一步测试。

```ts
import { expectTypeOf } from 'vitest'

type ResponsiveProp<T> = T | T[] | { xs?: T, sm?: T, md?: T }

interface CSSProperties {
  margin?: string
  padding?: string
}

function getResponsiveProp<T>(_props: T): ResponsiveProp<T> {
  return {}
}

const cssProperties: CSSProperties = { margin: '1px', padding: '2px' }

expectTypeOf(getResponsiveProp(cssProperties))
  .extract<{ xs?: any }>() // extracts the last type from a union
  .toEqualTypeOf<{
  xs?: CSSProperties
  sm?: CSSProperties
  md?: CSSProperties
}>()

expectTypeOf(getResponsiveProp(cssProperties))
  .extract<unknown[]>() // extracts an array from a union
  .toEqualTypeOf<CSSProperties[]>()
```

::: warning
如果在联合类型中找不到类型，`.extract` 将返回 `never`。
:::

## exclude

- **类型:** `ExpectTypeOf<NonExcludedUnion>`

你可以使用 `.exclude` 从联合中删除类型以进行进一步测试。

```ts
import { expectTypeOf } from 'vitest'

type ResponsiveProp<T> = T | T[] | { xs?: T; sm?: T; md?: T }

interface CSSProperties { margin?: string; padding?: string }

function getResponsiveProp<T>(\_props: T): ResponsiveProp<T> {
return {}
}

const cssProperties: CSSProperties = { margin: '1px', padding: '2px' }

expectTypeOf(getResponsiveProp(cssProperties))
.exclude<unknown[]>()
.exclude<{ xs?: unknown }>() // or just .exclude<unknown[] | { xs?: unknown }>()
.toEqualTypeOf<CSSProperties>()

```

::: warning
如果在联合类型中找不到类型，`.extract` 将返回 `never`。
:::

## returns

- **类型:** `ExpectTypeOf<ReturnValue>`

你可以使用 `.returns` 来提取函数类型的返回值。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(() => {}).returns.toBeVoid()
expectTypeOf((a: number) => [a, a]).returns.toEqualTypeOf([1, 2])
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## parameters

- **类型:** `ExpectTypeOf<Parameters>`

你可以使用 `.parameters` 提取函数参数以对其值执行断言。参数以数组形式返回。

```ts
import { expectTypeOf } from 'vitest'

type NoParam = () => void
type HasParam = (s: string) => void

expectTypeOf<NoParam>().parameters.toEqualTypeOf<[]>()
expectTypeOf<HasParam>().parameters.toEqualTypeOf<[string]>()
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

::: tip
你还可以使用 [`.toBeCallableWith`](#tobecallablewith) 匹配器作为更具表现力的断言。
:::

## parameter

- **类型:** `(nth: number) => ExpectTypeOf`

你可以使用 `.parameter(number)` 调用提取某个函数参数，以对其执行其他断言。

```ts
import { expectTypeOf } from 'vitest'

function foo(a: number, b: string) {
  return [a, b]
}

expectTypeOf(foo).parameter(0).toBeNumber()
expectTypeOf(foo).parameter(1).toBeString()
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## constructorParameters

- **类型:** `ExpectTypeOf<ConstructorParameters>`

你可以将构造函数参数提取为数组元素，并使用此方法对它们执行断言。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(Date).constructorParameters.toEqualTypeOf<
  [] | [string | number | Date]
>()
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

::: tip
你还可以使用 [`.toBeConstructibleWith`](#tobeconstructiblewith) 匹配器作为更具表现力的断言。
:::

## instance

- **类型:** `ExpectTypeOf<ConstructableInstance>`

此属性允许访问可以在所提供类的实例上执行匹配器。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(Date).instance.toHaveProperty('toISOString')
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## items

- **类型:** `ExpectTypeOf<T>`

你可以使用 `.items` 获取数组项类型以执行进一步的断言。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf([1, 2, 3]).items.toEqualTypeOf<number>()
expectTypeOf([1, 2, 3]).items.not.toEqualTypeOf<string>()
```

## resolves

- **类型:** `ExpectTypeOf<ResolvedPromise>`

此匹配器提取 `Promise` 的已解析值，因此你可以对其执行其他断言。

```ts
import { expectTypeOf } from 'vitest'

async function asyncFunc() {
  return 123
}

expectTypeOf(asyncFunc).returns.resolves.toBeNumber()
expectTypeOf(Promise.resolve('string')).resolves.toBeString()
```

::: warning
如果用于非承诺类型，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## guards

- **类型:** `ExpectTypeOf<Guard>`

此匹配器提取保护值（例如，`v is number`），因此你可以对其执行断言。

```ts
import { expectTypeOf } from 'vitest'

function isString(v: any): v is string {
  return typeof v === 'string'
}

expectTypeOf(isString).guards.toBeString()
```

::: warning
如果该值不是保护函数，则返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## asserts

- **类型:** `ExpectTypeOf<Assert>`

此匹配器提取断言值（例如，`assert v is number`），因此你可以对其执行断言。

```ts
import { expectTypeOf } from 'vitest'

function assertNumber(v: any): asserts v is number {
  if (typeof v !== 'number') {
    throw new TypeError('Nope !')
  }
}

expectTypeOf(assertNumber).asserts.toBeNumber()
```

::: warning
如果该值不是断言函数，则返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## toBeAny

- **类型:** `() => void`

使用此匹配器，你可以检查提供的类型是否为 `any` 类型。如果类型太具体，测试将失败。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf<any>().toBeAny()
expectTypeOf({} as any).toBeAny()
expectTypeOf('string').not.toBeAny()
```

## toBeUnknown

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `unknown` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf().toBeUnknown()
expectTypeOf({} as unknown).toBeUnknown()
expectTypeOf('string').not.toBeUnknown()
```

## toBeNever

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `never` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf<never>().toBeNever()
expectTypeOf((): never => {}).returns.toBeNever()
```

## toBeFunction

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `function` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(42).not.toBeFunction()
expectTypeOf((): never => {}).toBeFunction()
```

## toBeObject

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `object` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(42).not.toBeObject()
expectTypeOf({}).toBeObject()
```

## toBeArray

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `Array<T>` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(42).not.toBeArray()
expectTypeOf([]).toBeArray()
expectTypeOf([1, 2]).toBeArray()
expectTypeOf([{}, 42]).toBeArray()
```

## toBeString

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `string` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(42).not.toBeString()
expectTypeOf('').toBeString()
expectTypeOf('a').toBeString()
```

## toBeBoolean

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `boolean` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(42).not.toBeBoolean()
expectTypeOf(true).toBeBoolean()
expectTypeOf<boolean>().toBeBoolean()
```

## toBeVoid

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `void` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(() => {}).returns.toBeVoid()
expectTypeOf<void>().toBeVoid()
```

## toBeSymbol

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `symbol` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(Symbol(1)).toBeSymbol()
expectTypeOf<symbol>().toBeSymbol()
```

## toBeNull

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `null` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(null).toBeNull()
expectTypeOf<null>().toBeNull()
expectTypeOf(undefined).not.toBeNull()
```

## toBeUndefined

- **类型:** `() => void`

此匹配器检查提供的类型是否为 `undefined` 类型。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(undefined).toBeUndefined()
expectTypeOf<undefined>().toBeUndefined()
expectTypeOf(null).not.toBeUndefined()
```

## toBeNullable

- **类型:** `() => void`

此匹配器会检查你是否可以对提供的类型使用 `null` 或 `undefined`。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf<undefined | 1>().toBeNullable()
expectTypeOf<null | 1>().toBeNullable()
expectTypeOf<undefined | null | 1>().toBeNullable()
```

## toBeCallableWith

- **类型:** `() => void`

此匹配器确保你可以使用一组参数作为参数来调用函数。

```ts
import { expectTypeOf } from 'vitest'

type NoParam = () => void
type HasParam = (s: string) => void

expectTypeOf<NoParam>().toBeCallableWith()
expectTypeOf<HasParam>().toBeCallableWith('some string')
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## toBeConstructibleWith

- **类型:** `() => void`

此匹配器确保你可以使用一组构造函数作为参数创建一个新实例。

```ts
import { expectTypeOf } from 'vitest'

expectTypeOf(Date).toBeConstructibleWith(new Date())
expectTypeOf(Date).toBeConstructibleWith('01-01-2000')
```

::: warning
如果用在非函数类型上，它将返回 `never`，因此你将无法将它与其他匹配器链接起来。
:::

## toHaveProperty

- **类型:** `<K extends keyof T>(property: K) => ExpectTypeOf<T[K>`

此匹配器检查提供的对象上是否存在属性。如果它存在，它还会为该属性的类型返回同一组匹配器，因此你可以一个接一个地链接断言。

```ts
import { expectTypeOf } from 'vitest'

const obj = { a: 1, b: '' }

expectTypeOf(obj).toHaveProperty('a')
expectTypeOf(obj).not.toHaveProperty('c')

expectTypeOf(obj).toHaveProperty('a').toBeNumber()
expectTypeOf(obj).toHaveProperty('b').toBeString()
expectTypeOf(obj).toHaveProperty('a').not.toBeString()
```
