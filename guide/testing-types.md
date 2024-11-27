---
title: 类型测试 | 指南
---

# 类型测试

::: tip Sample Project

[GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/typecheck) - [Play Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/typecheck?initialPath=__vitest__/)

:::

Vitest 允许你使用 `expectTypeOf` 或 `assertType` 语法为你的类型编写测试。默认情况下，`*.test-d.ts` 文件中的所有测试都被视为类型测试，但你可以使用 [`typecheck.include`](/config/#typecheck-include) 配置选项更改它。

在这里，Vitest 调用 `tsc` 或 `vue-tsc`，具体取决于你的配置，并解析结果。如果发现任何类型错误，Vitest 还会在你的源代码中打印出类型错误。你可以使用 [`typecheck.ignoreSourceErrors`](/config/#typecheck-ignoresourceerrors) 配置选项禁用它。

请记住，Vitest 不会运行这些文件，编译器只会对它们进行静态分析。也就是说，如果您使用动态名称或 `test.each` 或 `test.for`，测试名称将不会被评估 - 它将原样显示。

::: warning
在 Vitest 2.1 之前，您的 `typecheck.include` 覆盖了 `include` 模式，因此您的运行时测试并没有实际运行；它们只是被类型检查。

自 Vitest 2.1 起，如果您的 `include` 和 `typecheck.include` 重叠，Vitest 将分别报告类型测试和运行时测试。
:::

使用 CLI 标志，如 `--allowOnly` 和 `-t` 也支持类型检查。

```ts
import { assertType, expectTypeOf } from 'vitest'
import { mount } from './mount.js'

test('my types work properly', () => {
  expectTypeOf(mount).toBeFunction()
  expectTypeOf(mount).parameter(0).toMatchTypeOf<{ name: string }>()

  // @ts-expect-error name is a string
  assertType(mount({ name: 42 }))
})
```

在测试文件中触发的任何类型错误都将被视为测试错误，因此你可以使用任何类型技巧来测试项目中的类型。

你可以在 [API 部分](/api/#expecttypeof) 中查看可能的匹配器列表。

## 读取错误

如果使用的是 `expectTypeOf` API，请参阅 [expect-type 关于其错误信息的文档](https://github.com/mmkal/expect-type#error-messages)。

当类型不匹配时，`.toEqualTypeOf` 和 `.toMatchTypeOf`会使用一种特殊的辅助类型来生成尽可能可操作的错误信息。但要理解它们还有一些细微差别。由于断言是 "流畅地 "编写的，所以失败应该发生在 "预期 "类型上，而不是 "实际 "类型上（`expect<Actual>().toEqualTypeOf<Expected>()`）。这意味着类型错误可能有点令人困惑，因此该库生成了一个 `MismatchInfo` 类型，试图明确说明期望是什么。例如

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: string }>()
```

是一个将失败的断言，因为 `{a: 1}` 的类型是 `{a: number}` 而不是 `{a: string}`。 这种情况下的错误信息如下

```
test/test.ts:999:999 - error TS2344: Type '{ a: string; }' does not satisfy the constraint '{ a: \\"Expected: string, Actual: number\\"; }'.
  Types of property 'a' are incompatible.
    Type 'string' is not assignable to type '\\"Expected: string, Actual: number\\"'.

999 expectTypeOf({a: 1}).toEqualTypeOf<{a: string}>()
```

请注意，报告的类型约束是一个可读性强的消息，指定了"期望"和"实际"类型。不要字面上解读句子 `Types of property 'a' are incompatible // Type 'string' is not assignable to type "Expected: string, Actual: number"` ，而是看属性名（`'a'`）和消息内容：`Expected: string, Actual: number`。这将告诉你出了什么问题，在大多数情况下。当然，对于非常复杂的类型，调试可能需要更多的努力，并且可能需要一些试验。如果错误消息实际上是误导性的，请[提出问题](https://github.com/mmkal/expect-type)。

对于像 `expectTypeOf(1).toBeString()` 这样的断言，`toBe...` 方法（如 `toBeString`、`toBeNumber`、`toBeVoid` 等）在被测试的 `Actual` 类型不匹配时会解析为一个不可调用的类型。例如，失败的断言可能会显示如下内容：

```
test/test.ts:999:999 - error TS2349: This expression is not callable.
  Type 'ExpectString<number>' has no call signatures.

999 expectTypeOf(1).toBeString()
                    ~~~~~~~~~~
```

这部分的`This expression is not callable`并没有太大的帮助 - 有意义的错误在下一行，`Type 'ExpectString<number> has no call signatures`。这基本上意味着你传递了一个数字，但断言它应该是一个字符串。

如果 TypeScript 添加了对 ["throw" 类型](https://github.com/microsoft/TypeScript/pull/40468) 的支持，这些错误消息将会显著改进。在那之前，它们需要一定程度的仔细观察。

#### 具体的 "expected " 对象与类型参数

像这样的断言的错误消息：

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: '' })
```

对于像这样的断言，错误消息将不够有帮助：

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: string }>()
```

这是因为 TypeScript 编译器需要推断 `.toEqualTypeOf({a: ''})` 样式的类型参数，并且该库只能通过将其与通用的 `Mismatch` 类型进行比较来标记它为失败。因此，在可能的情况下，使用类型参数而不是具体类型来使用 `.toEqualTypeOf` 和 `toMatchTypeOf`。如果使用两个具体类型进行比较更加方便，可以使用 `typeof`：

```ts
const one = valueFromFunctionOne({ some: { complex: inputs } })
const two = valueFromFunctionTwo({ some: { other: inputs } })

expectTypeOf(one).toEqualTypeof<typeof two>()
```

如果你发现很难使用 `expectTypeOf` API 并找出错误，你始终可以使用更简单的 `assertType` API：

```ts
const answer = 42

assertType<number>(answer)
// @ts-expect-error answer is not a string
assertType<string>(answer)
```

::: tip
使用 `@ts-expect-error` 语法时，你可能想确保没有输入错误。你可以通过在 [`test.include`](/config/#include) 配置选项中包含你的类型文件来做到这一点，因此 Vitest 实际上也会*运行*这些测试并因 `ReferenceError` 而失败。

这将通过，因为它预计会出现错误，但 “answer” 这个词有错别字，所以这是一个误报错误：

```ts
// @ts-expect-error answer is not a string
assertType<string>(answr)
```

:::

## 运行类型检查

要启用类型检查，只需在 `package.json` 文件中的 Vitest 命令中添加 [`--typecheck`](/config/#typecheck) 标志：

```json [package.json]
{
  "scripts": {
    "test": "vitest --typecheck"
  }
}
```

现在你可以运行 typecheck:

::: code-group

```bash [npm]
npm run test
```

```bash [yarn]
yarn test
```

```bash [pnpm]
pnpm run test
```

```bash [bun]
bun test
```

:::

Vitest 使用 `tsc --noEmit` 或 `vue-tsc --noEmit`，具体取决于你的配置，因此可以从管道中删除这些脚本。
