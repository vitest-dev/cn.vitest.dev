---
title: 测试快照 | 指南
---

# 测试快照

当你希望确保函数的输出不会意外更改时，快照测试是一个非常有用的工具。

<CourseLink href="https://vueschool.io/lessons/snapshots-in-vitest?friend=vueuse">通过 Vue School 的视频学习快照</CourseLink>

使用快照时，Vitest 将获取给定值的快照，将其比较时将参考存储在测试旁边的快照文件。如果两个快照不匹配，则测试将失败：要么更改是意外的，要么参考快照需要更新到测试结果的新版本。

## 使用快照

要将一个值快照，你可以使用 `expect()` 的 [`toMatchSnapshot()`](/api/#tomatchsnapshot) API:

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchSnapshot()
})
```

此测试在第一次运行时，Vitest 会创建一个快照文件，如下所示：

```js
// Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html

exports['toUpperCase 1'] = '"FOOBAR"'
```

快照文件应该与代码更改一起提交，并作为代码审查过程的一部分进行审查。在随后的测试运行中，Vitest 会将执行的输出与之前的快照进行比较。如果他们匹配，测试就会通过。如果它们不匹配，要么测试运行时在你的代码中发现了应该修复的错误，要么实现已经更改，需要更新快照。

## 内联快照

::: warning
在异步并发测试中使用快照时，由于 JavaScript 的限制，你需要使用 [测试环境](/guide/test-context) 中的 `expect` 来确保检测到正确的测试。
:::

如同前文，你可以使用 [`toMatchInlineSnapshot()`](/api/#tomatchinlinesnapshot) 将内联快照存储在测试文件中。

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchInlineSnapshot()
})
```

Vitest 不会创建快照文件，而是直接修改测试文件，将快照作为字符串更新到文件中：

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchInlineSnapshot('"FOOBAR"')
})
```

这允许你直接查看期望输出，而无需跨不同的文件跳转。

## 更新快照

::: warning
在异步并发测试中使用快照时，由于 JavaScript 的限制，你需要使用 [测试环境](/guide/test-context) 中的 `expect` 来确保检测到正确的测试。
:::

当接收到的值与快照不匹配时，测试将失败，并显示它们之间的差异。当需要更改快照时，你可能希望从当前状态更新快照。

在监听(watch)模式下, 你可以在终端中键入 `u` 键直接更新失败的快照。

或者，你可以在 CLI 中使用 `--update` 或 `-u` 标记使 Vitest 进入快照更新模式。

```bash
vitest -u
```

## 文件快照

调用 `toMatchSnapshot()` 时，我们将所有快照存储在格式化的快照文件中。这意味着我们需要转义快照字符串中的一些字符（即双引号 `"` 和反引号 `\``）。同时，你可能会丢失快照内容的语法突出显示（如果它们是某种语言）。

为了改善这种情况，我们引入 [`toMatchFileSnapshot()`](/api/expect#tomatchfilesnapshot) 以在文件中显式快照。这允许你为快照文件分配任何文件扩展名，并使它们更具可读性。

```ts
import { expect, it } from 'vitest'

it('render basic', async () => {
  const result = renderHTML(h('div', { class: 'foo' }))
  await expect(result).toMatchFileSnapshot('./test/basic.output.html')
})
```

它将与 `./test/basic.output.html` 的内容进行比较。并且可以用 `--update` 标志写回。

## 图像快照

快照图像也可以使用 [`jest-image-snapshot`](https://github.com/americanexpress/jest-image-snapshot)。

```bash
npm i -D jest-image-snapshot
```

```ts
test('image snapshot', () => {
  expect(readFileSync('./test/stubs/input-image.png')).toMatchImageSnapshot()
})
```

## 自定义序列化程序

你可以添加自己的逻辑来修改快照的序列化方式。像 Jest 一样，Vitest 为内置的 JavaScript 类型、HTML 元素、ImmutableJS 和 React 元素提供了默认的序列化程序。

可以使用 [`expect.addSnapshotSerializer`](/api/expect#expect-addsnapshotserializer) 添加自定义序列器。

```ts
expect.addSnapshotSerializer({
  serialize(val, config, indentation, depth, refs, printer) {
    // `printer` is a function that serializes a value using existing plugins.
    return `Pretty foo: ${printer(val.foo, config, indentation, depth, refs)}`
  },
  test(val) {
    return val && Object.prototype.hasOwnProperty.call(val, 'foo')
  },
})
```

我们还支持 [snapshotSerializers](/config/#snapshotserializers) 选项来隐式添加自定义序列化器。

```ts [path/to/custom-serializer.ts]
import { SnapshotSerializer } from 'vitest'

export default {
  serialize(val, config, indentation, depth, refs, printer) {
    // `printer` 是一个使用现有插件序列化数值的函数。
    return `Pretty foo: ${printer(val.foo, config, indentation, depth, refs)}`
  },
  test(val) {
    return val && Object.prototype.hasOwnProperty.call(val, 'foo')
  },
} satisfies SnapshotSerializer
```

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    snapshotSerializers: ['path/to/custom-serializer.ts'],
  },
})
```

如下所示的测试添加后：

```ts
test('foo snapshot test', () => {
  const bar = {
    foo: {
      x: 1,
      y: 2,
    },
  }

  expect(bar).toMatchSnapshot()
})
```

你将获得以下快照：

```
Pretty foo: Object {
  "x": 1,
  "y": 2,
}
```

我们使用的是 Jest 的 `pretty-format` 来序列化快照。你可以在这里阅读更多相关内容：[pretty-format](https://github.com/facebook/jest/blob/main/packages/pretty-format/README.md#serialize).

## 与 Jest 的区别

Vitest 提供了与 [Jest](https://jestjs.io/docs/snapshot-testing) 几乎兼容的快照功能，除少数例外:

#### 1. 快照文件中的注释标头不同

```diff
- // Jest Snapshot v1, https://goo.gl/fbAQLP
+ // Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html
```

这实际上不会影响功能，但在从 Jest 迁移时可能会影响提交差异。

#### 2. `printBasicPrototype` 默认为 `false`

Jest 和 Vitest 的快照都是由 [`pretty-format`](https://github.com/facebook/jest/blob/main/packages/pretty-format) 支持的。在 Vitest 中，我们将 `printBasicPrototype` 的默认值设置为 `false` 以提供更清晰的快照输出，在 Jest 版本 < 29.0.0 中默认为 `true`。

```ts
import { expect, test } from 'vitest'

test('snapshot', () => {
  const bar = [
    {
      foo: 'bar',
    },
  ]

  // in Jest
  expect(bar).toMatchInlineSnapshot(`
    Array [
      Object {
        "foo": "bar",
      },
    ]
  `)

  // in Vitest
  expect(bar).toMatchInlineSnapshot(`
    [
      {
        "foo": "bar",
      },
    ]
  `)
})
```

我们相信这种预设有更好的可读性和开发体验。如果你仍然喜欢 Jest 的行为，可以通过以下方式更改配置：

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    snapshotFormat: {
      printBasicPrototype: true,
    },
  },
})
```

#### 3. 使用 V 形 `>` 而非冒号 `:` 作为自定义消息的分隔符

当创建快照文件期间传递自定义消息时，Vitest 使用 V 形 `>` 作为分隔符而不是冒号 `:` 以提高自定义消息可读性。

对于以下示例测试代码：

```js
test('toThrowErrorMatchingSnapshot', () => {
  expect(() => {
    throw new Error('error')
  }).toThrowErrorMatchingSnapshot('hint')
})
```

在 Jest 中，快照将是：

```console
exports[`toThrowErrorMatchingSnapshot: hint 1`] = `"error"`;
```

在 Vitest 中，等效的快照将是：

```console
exports[`toThrowErrorMatchingSnapshot > hint 1`] = `[Error: error]`;
```

#### 4. `toThrowErrorMatchingSnapshot` 和 `toThrowErrorMatchingInlineSnapshot` 的默认 `Error` 快照不同

```js
import { expect, test } from 'vitest'

test('snapshot', () => {
  // in Jest and Vitest
  expect(new Error('error')).toMatchInlineSnapshot(`[Error: error]`)

  // Jest snapshots `Error.message` for `Error` instance
  // Vitest prints the same value as toMatchInlineSnapshot
  expect(() => {
    throw new Error('error')
  }).toThrowErrorMatchingInlineSnapshot(`"error"`) // [!code --]
  }).toThrowErrorMatchingInlineSnapshot(`[Error: error]`) // [!code ++]
})
```
