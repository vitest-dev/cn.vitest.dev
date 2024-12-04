---
title: 命令行界面 | 指南
outline: deep
---

# 命令行界面

## 命令

### `vitest`

在当前目录中启动 Vitest。在开发环境会自动进入监听(`watch`)模式，在 CI 环境会自动进入运行(`run`)模式。

你可以通过添加参数作为过滤器来运行测试文件，比如：

```bash
vitest foobar
```

将仅运行路径中包含 `foobar` 的测试文件。 此过滤器仅检查包含，不支持正则表达式或 glob 模式（除非你的终端在 Vitest 接收过滤器之前对其进行处理）。

### `vitest run`

在没有监听模式的情况下执行单次运行。

### `vitest watch`

运行所有测试套件，监听变化并在变化时重新运行测试。与没有参数的情况下调用 `vitest` 一样。在 CI 环境中，此命令将回退到 `vitest run`。

### `vitest dev`

`vitest watch` 的别名。

### `vitest related`

仅运行涵盖源文件列表的测试。 适用于静态惰性导入(例如, `import('./index.ts')` 或者 `import index from './index.ts`)，但不适用于动态导入(例如, `import(filepath)`)。 所有文件都应该相对于根文件夹。

与 [`lint-staged`](https://github.com/okonet/lint-staged) 或你的 CI 设置一起运行很有用。

```bash
vitest related /src/index.ts /src/hello-world.js
```

::: tip
不要忘记 Vitest 默认情况下以启用的监视模式运行。如果你使用的是 `lint-staged` 之类的工具，你还应该传递 `--run` 选项，以便该命令可以正常退出。

```js [.lintstagedrc.js]
export default {
  '*.{js,ts}': 'vitest related --run',
}
```

:::

### `vitest bench`

仅运行 [基准](https://vitest.dev/guide/features.html#benchmarking-experimental) 测试，比较性能结果。

### `vitest init`

`vitest-init<name>` 可以用于设置项目配置。目前，它只支持 [`browser`](/guide/browser/) 值：

```bash
vitest init browser
```

### `vitest list`

`vitest list` 命令继承所有的 `vitest` 选项以打印所有匹配测试的列表。此命令忽略 `reporters` 选项。默认情况下，它将打印与文件过滤器和名称模式匹配的所有测试的名称：

```shell
vitest list filename.spec.ts -t="some-test"
```

```txt
describe > some-test
describe > some-test > test 1
describe > some-test > test 2
```

你可以传递 `--json` 标志以 JSON 格式打印测试，也可以将其保存在单独的文件中：

```bash
vitest list filename.spec.ts -t="some-test" --json=./file.json
```

如果 `--json` 标志没有接收到值，它将把 JSON 输出到 stdout 中。

你还可以传递 `--filesOnly` 标志来仅打印测试文件：

```bash
vitest list --filesOnly
```

```txt
tests/test1.test.ts
tests/test2.test.ts
```

## 选项

::: tip
Vitest 支持 CLI 参数的 both camel case 和 kebab case 。例如，`--passWithNoTests` 和 `--pass-with-no-tests` 都有效（`--no-color` 和 `--inspect-brk` 是例外）。

Vitest 还支持不同的指定值的方式：`--reporter dot` 和 `--reporter=dot` 都是有效的。

如果选项支持值数组，则需要多次传递选项：

```bash
vitest --reporter=dot --reporter=default
```

布尔值选项可以用 `no-` 前缀来否定。将值指定为 `false` 也有效：

```bash
vitest --no-api
vitest --api=false
```

:::

<!--@include: ./cli-generated.md-->

### changed

- **类型**: `boolean | string`
- **默认值**: false

  设置为 true 时，仅对已更改的文件运行测试。默认情况下，将考虑所有未提交的更改（包括已暂存和未暂存的文件）。

  要对最近一次提交中的更改运行测试，可以使用 `--changed HEAD~1`。还可以使用提交哈希（`commit hash`）或分支名称。

  如果与 `forceRerunTriggers` 配置选项配合使用，并找到与更改的文件匹配的内容，将运行整个测试套件。

  与代码覆盖一起使用时，报告将只包含与更改相关的文件。

  如果与 [`forceRerunTriggers`](/config/#forcereruntriggers)配置选项搭配使用，则在 `forceRerunTriggers` 列表中列出的文件至少有一个发生变化时，将运行整个测试套件。默认情况下，Vitest 配置文件和 `package.json` 的更改将始终重新运行整个套件。

### shard

- **类型**: `string`
- **默认值**: disabled

  测试套件分片，格式为 `<index>/<count>`，其中

  - `count` 是正整数，表示分割的部分数
  - `index` 是正整数，表示当前分片的索引

  该命令将将所有测试分成 `count` 个相等的部分，并只运行位于 `index` 部分的测试。例如，要将测试套件分成三个部分，请使用以下命令：

```sh
vitest run --shard=1/3
vitest run --shard=2/3
vitest run --shard=3/3
```

:::warning 警告
无法在启用 `--watch`（默认情况下在开发中启用）时使用此选项。
:::

::: tip
如果在没有输出文件的情况下使用 `--reporter=blob`，则默认路径将包括当前碎片配置，以避免与其他 Vitest 进程发生冲突。
:::

### merge-reports

- **类型:** `boolean | string`

合并位于指定文件夹中的每个 blob 报告（默认情况下为`.vitest-reports`）。你可以将任何报告程序与此命令一起使用（[`blob`](/guide/reporters#blob-reporter) 除外）：

```sh
vitest --merge-reports --reporter=junit
```

[cac's dot notation]: https://github.com/cacjs/cac#dot-nested-options
