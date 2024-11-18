---
title: 报告器 | 指南
outline: deep
---

# 报告器

Vitest 提供了几种内置报告器，以不同格式显示测试输出，以及使用自定义报告器的能力。你可以使用 `--reporter` 命令行选项，或者在你的 `outputFile`[配置选项](https://vitest.dev/config/#reporters) 中加入 `reporters` 属性来选择不同的报告器。如果没有指定报告器，Vitest 将使用下文所述的默认报告器。

通过命令行使用报告器:

```bash
npx vitest --reporter=verbose
```

通过配置文件 [`vitest.config.ts`](/config/) 使用报告器:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    reporters: ['verbose'],
  },
})
```

某些报告器可以通过传递附加选项进行自定义。具体选项将在下面的章节中介绍。


```ts
export default defineConfig({
  test: {
    reporters: ['default', ['junit', { suiteName: 'UI tests' }]],
  },
})
```

## 报告器输出

默认情况下，Vitest 的报告器会将输出打印到终端。当使用 `json` 、`html` 或 `junit` 报告器时，你可以在 Vite 配置文件中或通过 CLI 加入 `outputFile` [配置选项](https://vitest.dev/config/#outputfile)，将测试输出写入文件。

:::code-group

```bash [CLI]
npx vitest --reporter=json --outputFile=./test-output.json
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['json'],
    outputFile: './test-output.json',
  },
})
```

:::

## 组合报告器

你可以同时使用多个报告器，并以不同格式打印测试结果。

例如:

```bash
npx vitest --reporter=json --reporter=default
```

```ts
export default defineConfig({
  test: {
    reporters: ['json', 'default'],
    outputFile: './test-output.json',
  },
})
```

上面的示例将同时把测试结果以默认样式打印到终端，和以 JSON 格式写入指定的输出文件。

使用多个报告器时，还可以指定多个输出文件，如下所示:

```ts
export default defineConfig({
  test: {
    reporters: ['junit', 'json', 'verbose'],
    outputFile: {
      junit: './junit-report.xml',
      json: './json-report.json',
    },
  },
})
```

这个示例将编写单独的 JSON 和 XML 报告，并将详细报告打印到终端。

## 内置报告器

### 默认报告器

<<<<<<< HEAD
默认情况下（即未指定报告程序），Vitest 会在每个测试套件运行时分级显示结果，并在套件通过后折叠显示。所有测试运行结束后，最终终端输出将显示结果摘要和失败测试的详细信息。
=======
By default (i.e. if no reporter is specified), Vitest will display summary of running tests and their status at the bottom. Once a suite passes, its status will be reported on top of the summary.

You can disable the summary by configuring the reporter:

:::code-group
```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: [
      ['default', { summary: false }]
    ]
  },
})
```
:::
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

项目中的测试输出示例:

```bash
 ✓ test/example-1.test.ts (5 tests | 1 skipped) 306ms
 ✓ test/example-2.test.ts (5 tests | 1 skipped) 307ms

 ❯ test/example-3.test.ts 3/5
 ❯ test/example-4.test.ts 1/5

 Test Files 2 passed (4)
      Tests 10 passed | 3 skipped (65)
   Start at 11:01:36
   Duration 2.00s
```

测试完成后的最终输出:

```bash
 ✓ test/example-1.test.ts (5 tests | 1 skipped) 306ms
 ✓ test/example-2.test.ts (5 tests | 1 skipped) 307ms
 ✓ test/example-3.test.ts (5 tests | 1 skipped) 307ms
 ✓ test/example-4.test.ts (5 tests | 1 skipped) 307ms

 Test Files  4 passed (4)
      Tests  16 passed | 4 skipped (20)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### 基础报告器

<<<<<<< HEAD
基础报告器会显示已运行的测试文件，以及整个套件运行结束后的结果摘要。单独的测试除非不合格，否则不列入报告。
=======
The `basic` reporter is equivalent to `default` reporter without `summary`.
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

:::code-group

```bash [CLI]
npx vitest --reporter=basic
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['basic'],
  },
})
```

:::

使用基础报告器的输出示例:

```bash
✓ __tests__/file1.test.ts (2) 725ms
✓ __tests__/file2.test.ts (2) 746ms

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### 详细报告器

<<<<<<< HEAD
采用与`默认报告器`相同的层次结构，但不会折叠已通过测试套件的子树。终端最终输出会显示所有已运行的测试，包括已通过的测试。
=======
Verbose reporter is same as `default` reporter, but it also displays each individual test after the suite has finished. It also displays currently running tests that are taking longer than [`slowTestThreshold`](/config/#slowtestthreshold). Similar to `default` reporter, you can disable the summary by configuring the reporter.
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

:::code-group

```bash [CLI]
npx vitest --reporter=verbose
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
<<<<<<< HEAD
    reporters: ['verbose'],
=======
    reporters: [
      ['verbose', { summary: false }]
    ]
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9
  },
})
```

:::

<<<<<<< HEAD
测试套件通过后的终端最终输出示例:
=======
Example output for tests in progress with default `slowTestThreshold: 300`:

```bash
 ✓ __tests__/example-1.test.ts (2) 725ms
   ✓ first test file (2) 725ms
     ✓ 2 + 2 should equal 4
     ✓ 4 - 2 should equal 2

 ❯ test/example-2.test.ts 3/5
   ↳ should run longer than three seconds 1.57s
 ❯ test/example-3.test.ts 1/5

 Test Files 2 passed (4)
      Tests 10 passed | 3 skipped (65)
   Start at 11:01:36
   Duration 2.00s
```

Example of final terminal output for a passing test suite:
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

```bash
✓ __tests__/file1.test.ts (2) 725ms
   ✓ first test file (2) 725ms
     ✓ 2 + 2 should equal 4
     ✓ 4 - 2 should equal 2
✓ __tests__/file2.test.ts (2) 746ms
  ✓ second test file (2) 746ms
    ✓ 1 + 1 should equal 2
    ✓ 2 - 1 should equal 1

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### Dot 报告器

为每个已完成的测试打印一个点，以提供最少的输出，并显示所有已运行的测试。只提供失败测试的详细信息，以及套件的基本报告摘要。

:::code-group

```bash [CLI]
npx vitest --reporter=dot
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['dot'],
  },
})
```

:::

测试套件通过后的终端最终输出示例:

```bash
....

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### JUnit 报告器

以 JUnit XML 格式输出测试结果报告。既可打印到终端，也可使用 [`outputFile`](##报告器输出) 配置选项写入 XML 文件。

:::code-group

```bash [CLI]
npx vitest --reporter=junit
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['junit'],
  },
})
```

:::

JUnit XML 报告示例:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<testsuites name="vitest tests" tests="2" failures="1" errors="0" time="0.503">
    <testsuite name="__tests__/test-file-1.test.ts" timestamp="2023-10-19T17:41:58.580Z" hostname="My-Computer.local" tests="2" failures="1" errors="0" skipped="0" time="0.013">
        <testcase classname="__tests__/test-file-1.test.ts" name="first test file &gt; 2 + 2 should equal 4" time="0.01">
            <failure message="expected 5 to be 4 // Object.is equality" type="AssertionError">
AssertionError: expected 5 to be 4 // Object.is equality
 ❯ __tests__/test-file-1.test.ts:20:28
            </failure>
        </testcase>
        <testcase classname="__tests__/test-file-1.test.ts" name="first test file &gt; 4 - 2 should equal 2" time="0">
        </testcase>
    </testsuite>
</testsuites>
```

<<<<<<< HEAD
输出的 XML 包含嵌套的 `testsuites` 和 `testcase` 标签。我们可以使用 reporter 选项来配置这些属性：
=======
The outputted XML contains nested `testsuites` and `testcase` tags. These can also be customized via reporter options `suiteName` and `classnameTemplate`. `classnameTemplate` can either be a template string or a function.

The supported placeholders for the `classnameTemplate` option are:
- filename
- filepath
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

```ts
export default defineConfig({
  test: {
    reporters: [
<<<<<<< HEAD
      [
        'junit',
        { suiteName: 'custom suite name', classname: 'custom-classname' },
      ],
    ],
=======
      ['junit', { suiteName: 'custom suite name', classnameTemplate: 'filename:{filename} - filepath:{filepath}' }]
    ]
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9
  },
})
```

输出的 XML 包含嵌套的 `testsuites` 和 `testcase` 标记。你可以使用环境变量 `VITEST_JUNIT_SUITE_NAME` 和 `VITEST_JUNIT_CLASSNAME` 分别配置它们的名称和类名属性。

<<<<<<< HEAD
### JSON 报告器

以 JSON 格式输出测试结果报告。既可打印到终端，也可使用 [`outputFile`](##报告器输出) 配置选项写入文件。
=======
Generates a report of the test results in a JSON format compatible with Jest's `--json` option. Can either be printed to the terminal or written to a file using the [`outputFile`](/config/#outputfile) configuration option.
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

:::code-group

```bash [CLI]
npx vitest --reporter=json
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['json'],
  },
})
```

:::

JSON 报告示例:

```json
{
  "numTotalTestSuites": 4,
  "numPassedTestSuites": 2,
  "numFailedTestSuites": 1,
  "numPendingTestSuites": 1,
  "numTotalTests": 4,
  "numPassedTests": 1,
  "numFailedTests": 1,
  "numPendingTests": 1,
  "numTodoTests": 1,
  "startTime": 1697737019307,
  "success": false,
  "testResults": [
    {
      "assertionResults": [
        {
          "ancestorTitles": ["", "first test file"],
          "fullName": " first test file 2 + 2 should equal 4",
          "status": "failed",
          "title": "2 + 2 should equal 4",
          "duration": 9,
          "failureMessages": ["expected 5 to be 4 // Object.is equality"],
          "location": {
            "line": 20,
            "column": 28
          },
          "meta": {}
        }
      ],
      "startTime": 1697737019787,
      "endTime": 1697737019797,
      "status": "failed",
      "message": "",
      "name": "/root-directory/__tests__/test-file-1.test.ts"
    }
  ],
  "coverageMap": {}
}
```

<<<<<<< HEAD
### HTML 报告器
=======
::: info
Since Vitest 2.2, the JSON reporter includes coverage information in `coverageMap` if coverage is enabled.
:::

### HTML Reporter
>>>>>>> 74d74513bad192b4498234b7a439be8c859fa4d9

生成 HTML 文件，通过交互式 [GUI](/guide/ui) 查看测试结果。文件生成后，Vitest 将保持本地开发服务器运行，并提供一个链接，以便在浏览器中查看报告。

可使用 [`outputFile`](##报告器输出) 配置选项指定输出文件。如果没有提供 `outputFile` 选项，则会创建一个新的 HTML 文件。

:::code-group

```bash [CLI]
npx vitest --reporter=html
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['html'],
  },
})
```

:::

::: tip
该报告器需要安装 [`@vitest/ui`](/guide/ui) 。
:::

### TAP 报告器

按照 [Test Anything Protocol](https://testanything.org/) (TAP)输出报告。

:::code-group

```bash [CLI]
npx vitest --reporter=tap
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['tap'],
  },
})
```

:::

TAP 报告示例:

```bash
TAP version 13
1..1
not ok 1 - __tests__/test-file-1.test.ts # time=14.00ms {
    1..1
    not ok 1 - first test file # time=13.00ms {
        1..2
        not ok 1 - 2 + 2 should equal 4 # time=11.00ms
            ---
            error:
                name: "AssertionError"
                message: "expected 5 to be 4 // Object.is equality"
            at: "/root-directory/__tests__/test-file-1.test.ts:20:28"
            actual: "5"
            expected: "4"
            ...
        ok 2 - 4 - 2 should equal 2 # time=1.00ms
    }
}
```

### TAP 扁平报告器

输出 TAP 扁平报告。与 `TAP Reporter` 一样，测试结果的格式遵循 TAP 标准，但测试套件的格式是扁平列表，而不是嵌套层次结构。

:::code-group

```bash [CLI]
npx vitest --reporter=tap-flat
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['tap-flat'],
  },
})
```

:::

TAP 扁平报告示例:

```bash
TAP version 13
1..2
not ok 1 - __tests__/test-file-1.test.ts > first test file > 2 + 2 should equal 4 # time=11.00ms
    ---
    error:
        name: "AssertionError"
        message: "expected 5 to be 4 // Object.is equality"
    at: "/root-directory/__tests__/test-file-1.test.ts:20:28"
    actual: "5"
    expected: "4"
    ...
ok 2 - __tests__/test-file-1.test.ts > first test file > 4 - 2 should equal 2 # time=0.00ms
```

### Hanging process 报告器

展示任何妨碍 Vitest 安全退出的 hanging processes ，`hanging-process` 报告器本身不显示测试结果，但可与其他报告器结合使用，以便在测试运行时监控进程。使用这个报告器可能会消耗大量资源，因此通常应保留用于在 Vitest 无法正常退出进程的情况下进行调试的目的。

:::code-group

```bash [CLI]
npx vitest --reporter=hanging-process
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['hanging-process'],
  },
})
```

:::

### Github Actions Reporter {#github-actions-reporter}

输出 [工作流命令](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-error-message)
为测试失败提供注释。当 `process.env.GITHUB_ACTIONS === 'true'` 时，会自动启用 [`default`](#default-reporter)报告器。

如果配置了非默认报告器，则需要显式添加 `github-actions`。

```ts
export default defineConfig({
  test: {
    reporters: process.env.GITHUB_ACTIONS ? ['dot', 'github-actions'] : ['dot'],
  },
})
```

<img alt="Github Actions" img-dark src="https://github.com/vitest-dev/vitest/assets/4232207/336cddc2-df6b-4b8a-8e72-4d00010e37f5">
<img alt="Github Actions" img-light src="https://github.com/vitest-dev/vitest/assets/4232207/ce8447c1-0eab-4fe1-abef-d0d322290dca">

### Blob Reporter

将测试结果存储在计算机上，以便以后可以使用 [`--merge-reports`](/guide/cli#merge-reports) 命令进行合并。
默认情况下，将所有结果存储在 `.vitest-reports` 文件夹中，但可以用 `--outputFile` 或 `--outputFile.blob` 标志覆盖。

```bash
npx vitest --reporter=blob --outputFile=reports/blob-1.json
```

如果你在带有 [`--shard`](/guide/cli#shard) 标志的不同机器上运行 Vitest，我们建议你使用此报告程序。
使用 CI 管道末尾的 `--merge-reports` 命令，可以将所有 blob 报告合并到任何报告中：

```bash
npx vitest --merge-reports=reports --reporter=json --reporter=default
```

::: tip
Both `--reporter=blob` and `--merge-reports` do not work in watch mode.
:::

## 自定义报告器

你可以使用从 NPM 安装的第三方自定义报告器，方法是在 `reporter` 选项中指定它们的软件包名称:

:::code-group

```bash [CLI]
npx vitest --reporter=some-published-vitest-reporter
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['some-published-vitest-reporter'],
  },
})
```

:::

此外，你还可以定义自己的[自定义报告器](/advanced/reporters)，并通过指定文件路径来使用它们:

```bash
npx vitest --reporter=./path/to/reporter.ts
```

自定义报告器应实现[报告器接口](https://github.com/vitest-dev/vitest/blob/main/packages/vitest/src/node/types/reporter.ts)。
