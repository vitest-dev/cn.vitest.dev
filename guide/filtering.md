---
title: 测试筛选 | 指南
---

# 测试筛选

用于测试套件(suite)和测试的筛选(filtering)、超时(timeouts)、并发(concurrent)。

## CLI

你可以使用 CLI 按名称筛选测试文件：

```bash
$ vitest basic
```

将只执行包含 `basic` 的测试文件，例如：

```
basic.test.ts
basic-foo.test.ts
basic/foo.test.ts
```

你还可以使用 `-t, --testNamePattern <pattern>` 选项按全名过滤测试。当你想按文件内定义的名称而不是文件名本身进行过滤时，这将非常有用。

<<<<<<< HEAD
## 指定超时阈值
=======
Since Vitest 2.2, you can also specify the test by filename and line number:

```bash
$ vitest basic/foo.test.ts:10
```

::: warning
Note that you have to specify the full filename, and specify the exact line number, i.e. you can't do

```bash
$ vitest foo:10
$ vitest basic/foo.test.ts:10-25
```
:::

## Specifying a Timeout
>>>>>>> b629c6141185f4a49b4c443b5cdcfe1a9ddda849

你可以选择将超时阈值（以毫秒为单位）作为第三个参数传递给测试。默认值为 [5 秒](/config/#testtimeout)。

```ts
import { test } from 'vitest'

test('name', async () => {
  /* ... */
}, 1000)
```

Hooks 也可以接收超时阈值，默认值为 5 秒。

```ts
import { beforeAll } from 'vitest'

beforeAll(async () => {
  /* ... */
}, 1000)
```

## 跳过测试套件和测试

使用 `.skip` 以避免运行某些测试套件或测试

```ts
import { assert, describe, it } from 'vitest'

describe.skip('skipped suite', () => {
  it('test', () => {
    // 已跳过此测试套件，无错误
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('suite', () => {
  it.skip('skipped test', () => {
    // 已跳过此测试，无错误
    assert.equal(Math.sqrt(4), 3)
  })
})
```

## 选择要运行的测试套件和测试

使用 `.only` 仅运行某些测试套件或测试

```ts
import { assert, describe, it } from 'vitest'

// 仅运行此测试套件（以及标记为 Only 的其他测试套件）
describe.only('suite', () => {
  it('test', () => {
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('another suite', () => {
  it('skipped test', () => {
    // 已跳过测试，因为测试在 Only 模式下运行
    assert.equal(Math.sqrt(4), 3)
  })

  it.only('test', () => {
    // 仅运行此测试（以及标记为 Only 的其他测试）
    assert.equal(Math.sqrt(4), 2)
  })
})
```

## 未实现的测试套件和测试

使用 `.todo` 留存将要实施的测试套件和测试的待办事项

```ts
import { describe, it } from 'vitest'

// 此测试套件的报告中将显示一个条目
describe.todo('unimplemented suite')

// 此测试的报告中将显示一个条目
describe('suite', () => {
  it.todo('unimplemented test')
})
```
