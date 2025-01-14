# TestSpecification

`TestSpecification` 类描述了要作为测试运行的模块及其参数。

你只能通过在测试项目上调用 [`createSpecification`](/advanced/api/test-project#createspecification) 方法来创建规范：

```ts
const specification = project.createSpecification(
  resolve('./example.test.ts'),
  [20, 40], // optional test lines
)
```

`createSpecification` 期望一个已解析的模块 ID。它不会自动解析文件或检查文件是否存在于文件系统中。

## taskId

[Test module's](/advanced/api/test-suite#id) identifier.

## project

这引用了测试模块所属的 [`TestProject`](/advanced/api/test-project)。

## moduleId

Vite 模块图中的模块 ID。通常，它是一个使用 POSIX 分隔符的绝对文件路径：

```ts
'C:/Users/Documents/project/example.test.ts' // ✅
'/Users/mac/project/example.test.ts' // ✅
'C:\\Users\\Documents\\project\\example.test.ts' // ❌
```

## testModule

Instance of [`TestModule`](/advanced/api/test-module) assosiated with the specification. If test wasn't queued yet, this will be `undefined`.

## pool <Badge type="warning">experimental</Badge> {#pool}

测试模块将运行的 [`pool`](/config/#pool)。

::: danger
通过 [`poolMatchGlob`](/config/#poolmatchglob) 和 [`typecheck.enabled`](/config/#typecheck-enabled)，单个测试项目中可以有多个池。这意味着可以有多个规范具有相同的 `moduleId` 但不同的 `pool`。在 Vitest 4 中，项目将仅支持单个池，此属性将被移除。
:::

## testLines

这是源代码中定义测试文件的行号数组。此字段仅在 `createSpecification` 方法接收数组时定义。

请注意，如果这些行中的至少一行没有测试，整个测试套件将会失败。以下是一个正确的 `testLines` 配置示例：

::: code-group
```ts [script.js]
const specification = project.createSpecification(
  resolve('./example.test.ts'),
  [3, 8, 9],
)
```
```ts:line-numbers{3,8,9} [example.test.js]
import { test, describe } from 'vitest'

test('verification works')

describe('a group of tests', () => { // [!code error]
  // ...

  test('nested test')
  test.skip('skipped test')
})
```
:::

## toJSON

```ts
function toJSON(): SerializedTestSpecification
```

`toJSON` 生成一个 JSON 友好的对象，可以被 [浏览器模式](/guide/browser/) 或 [Vitest UI](/guide/ui) 消费。
