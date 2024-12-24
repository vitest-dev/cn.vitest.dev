### root

- **CLI:** `-r, --root <path>`
- **Config:** [root](/config/#root)

根路径

### config

- **CLI:** `-c, --config <path>`

配置文件的路径

### update

- **CLI:** `-u, --update`
- **Config:** [update](/config/#update)

更新快照

### watch

- **CLI:** `-w, --watch`
- **Config:** [watch](/config/#watch)

启用观察模式

### testNamePattern

- **CLI:** `-t, --testNamePattern <pattern>`
- **Config:** [testNamePattern](/config/#testnamepattern)

使用符合指定 regexp 模式的运行测试

### dir

- **CLI:** `--dir <path>`
- **Config:** [dir](/config/#dir)

扫描测试文件的基本目录

### ui

- **CLI:** `--ui`
- **Config:** [ui](/config/#ui)

启用UI

### open

- **CLI:** `--open`
- **Config:** [open](/config/#open)

自动打开用户界面（默认值：`!process.env.CI`）。

### api.port

- **CLI:** `--api.port [port]`

指定服务器端口。注意，如果端口已被使用，Vite 会自动尝试下一个可用端口，因此这可能不是服务器最终监听的实际端口。如果为 `true`，将设置为`51204`

### api.host

- **CLI:** `--api.host [host]`

指定服务器应该监听哪些 IP 地址。设为 `0.0.0.0` 或 `true` 则监听所有地址，包括局域网地址和公共地址

### api.strictPort

- **CLI:** `--api.strictPort`

设置为 true 时，如果端口已被使用，则退出，而不是自动尝试下一个可用端口

### silent

- **CLI:** `--silent`
- **Config:** [silent](/config/#silent)

测试中的静默控制台输出

### hideSkippedTests

- **CLI:** `--hideSkippedTests`

隐藏跳过测试的日志

### reporters

- **CLI:** `--reporter <name>`
- **Config:** [reporters](/config/#reporters)

指定 reporters

### outputFile

- **CLI:** `--outputFile <filename/-s>`
- **Config:** [outputFile](/config/#outputfile)

如果还指定了支持报告程序，则将测试结果写入文件，使用 cac 的点符号表示多个报告程序的单个输出结果 (比如: --outputFile.tap=./tap.txt)

### coverage.all

- **CLI:** `--coverage.all`
- **Config:** [coverage.all](/config/#coverage-all)

是否在报告中包含所有文件，包括未测试的文件

### coverage.provider

- **CLI:** `--coverage.provider <name>`
- **Config:** [coverage.provider](/config/#coverage-provider)

选择覆盖范围采集工具， 可用值为: "v8", "istanbul" and "custom"

### coverage.enabled

- **CLI:** `--coverage.enabled`
- **Config:** [coverage.enabled](/config/#coverage-enabled)

启用覆盖范围收集。可使用 `--coverage` CLI 选项覆盖（默认值：`false`）。

### coverage.include

- **CLI:** `--coverage.include <pattern>`
- **Config:** [coverage.include](/config/#coverage-include)

作为 glob 模式包含在覆盖范围内的文件。使用多个模式时，可指定多次（默认值：`**`）。

### coverage.exclude

- **CLI:** `--coverage.exclude <pattern>`
- **Config:** [coverage.exclude](/config/#coverage-exclude)

覆盖范围中要排除的文件。使用多个扩展名时，可指定多次（默认情况下： 访问 [`coverage.exclude`](https://vitest.dev/config/#coverage-exclude)

### coverage.extension

- **CLI:** `--coverage.extension <extension>`
- **Config:** [coverage.extension](/config/#coverage-extension)

包含在覆盖范围内的扩展名。使用多个扩展名时，可指定多次 (默认: `[".js", ".cjs", ".mjs", ".ts", ".mts", ".tsx", ".jsx", ".vue", ".svelte"]`)

### coverage.clean

- **CLI:** `--coverage.clean`
- **Config:** [coverage.clean](/config/#coverage-clean)

运行测试前清除覆盖结果（默认值：true）

### coverage.cleanOnRerun

- **CLI:** `--coverage.cleanOnRerun`
- **Config:** [coverage.cleanOnRerun](/config/#coverage-cleanonrerun)

重新运行监视时清理覆盖率报告（默认值：true）

### coverage.reportsDirectory

- **CLI:** `--coverage.reportsDirectory <path>`
- **Config:** [coverage.reportsDirectory](/config/#coverage-reportsdirectory)

将覆盖率报告写入的目录（默认值： ./coverage）

### coverage.reporter

- **CLI:** `--coverage.reporter <name>`
- **Config:** [coverage.reporter](/config/#coverage-reporter)

使用的报告。更多信息请访问 [`coverage.reporter`](https://vitest.dev/config/#coverage-reporter)。 (默认值: `["text", "html", "clover", "json"]`)

### coverage.reportOnFailure

- **CLI:** `--coverage.reportOnFailure`
- **Config:** [coverage.reportOnFailure](/config/#coverage-reportonfailure)

即使测试失败也能生成覆盖率报告 (默认值: `false`)

### coverage.allowExternal

- **CLI:** `--coverage.allowExternal`
- **Config:** [coverage.allowExternal](/config/#coverage-allowexternal)

收集项目根目录外文件的覆盖范围（默认值：`false`）

### coverage.skipFull

- **CLI:** `--coverage.skipFull`
- **Config:** [coverage.skipFull](/config/#coverage-skipfull)

 不显示语句、分支和函数覆盖率为 100% 的文件（默认值：`false`）

### coverage.thresholds.100

- **CLI:** `--coverage.thresholds.100`
- **Config:** [coverage.thresholds.100](/config/#coverage-thresholds-100)

将所有覆盖率阈值设置为 100 的快捷方式（默认值：`false`）

### coverage.thresholds.perFile

- **CLI:** `--coverage.thresholds.perFile`
- **Config:** [coverage.thresholds.perFile](/config/#coverage-thresholds-perfile)

查每个文件的阈值。 `--coverage.thresholds.lines`, `--coverage.thresholds.functions`, `--coverage.thresholds.branches`, `--coverage.thresholds.statements` 为实际阈值（默认值：`false`）

### coverage.thresholds.autoUpdate

- **CLI:** `--coverage.thresholds.autoUpdate`
- **Config:** [coverage.thresholds.autoUpdate](/config/#coverage-thresholds-autoupdate)

更新阈值： 当当前覆盖率高于配置的阈值时，将 "lines"、"functions"、"branches"和 "statements"更新到配置文件（默认值：`false`）

### coverage.thresholds.lines

- **CLI:** `--coverage.thresholds.lines <number>`

针对代码行的覆盖度阈值设定，请访问 [istanbuljs](https://github.com/istanbuljs/nyc#coverage-thresholds) 了解更多信息。此选项不适用于自定义 providers

### coverage.thresholds.functions

- **CLI:** `--coverage.thresholds.functions <number>`

针对函数的覆盖度阈值设定，请访问 [istanbuljs](https://github.com/istanbuljs/nyc#coverage-thresholds) 了解更多信息。 此选项不适用于自定义 providers

### coverage.thresholds.branches

- **CLI:** `--coverage.thresholds.branches <number>`

针对 branches 的覆盖度阈值设定，请访问 [istanbuljs](https://github.com/istanbuljs/nyc#coverage-thresholds) 了解更多信息。 此选项不适用于自定义 providers

### coverage.thresholds.statements

- **CLI:** `--coverage.thresholds.statements <number>`

针对 statements 的覆盖度阈值设定，请访问 [istanbuljs](https://github.com/istanbuljs/nyc#coverage-thresholds) 了解更多信息。 此选项不适用于自定义 providers

### coverage.ignoreClassMethods

- **CLI:** `--coverage.ignoreClassMethods <name>`
- **Config:** [coverage.ignoreClassMethods](/config/#coverage-ignoreclassmethods)

覆盖时要忽略的类方法名称数组。更多信息请访问 [istanbuljs](https://github.com/istanbuljs/nyc#ignoring-methods) 。该选项仅适用于 istanbul providers（默认值：`[]`）。

### coverage.processingConcurrency

- **CLI:** `--coverage.processingConcurrency <number>`
- **Config:** [coverage.processingConcurrency](/config/#coverage-processingconcurrency)

处理覆盖率结果时使用的并发限制。 （默认最小值介于 20 和 CPU 数量之间）

### coverage.customProviderModule

- **CLI:** `--coverage.customProviderModule <path>`
- **Config:** [coverage.customProviderModule](/config/#coverage-customprovidermodule)

指定自定义覆盖范围提供程序模块的模块名称或路径。 请访问[自定义 providers 覆盖范围](https://vitest.dev/guide/coverage#custom-coverage-provider) 了解更多信息。 此选项仅适用于自定义 providers

### coverage.watermarks.statements

- **CLI:** `--coverage.watermarks.statements <watermarks>`

High and low watermarks for statements in the format of `<high>,<low>`

### coverage.watermarks.lines

- **CLI:** `--coverage.watermarks.lines <watermarks>`

High and low watermarks for lines in the format of `<high>,<low>`

### coverage.watermarks.branches

- **CLI:** `--coverage.watermarks.branches <watermarks>`

High and low watermarks for branches in the format of `<high>,<low>`

### coverage.watermarks.functions

- **CLI:** `--coverage.watermarks.functions <watermarks>`

High and low watermarks for functions in the format of `<high>,<low>`

### mode

- **CLI:** `--mode <name>`
- **Config:** [mode](/config/#mode)

覆盖 Vite 模式 (默认值: `test` 或 `benchmark`)

### workspace

- **CLI:** `--workspace <path>`
- **Config:** [workspace](/config/#workspace)

工作区配置文件的路径

### isolate

- **CLI:** `--isolate`
- **Config:** [isolate](/config/#isolate)

隔离运行每个测试文件。要禁用隔离, 使用 `--no-isolate` (默认值: `true`)

### globals

- **CLI:** `--globals`
- **Config:** [globals](/config/#globals)

全局注入

### dom

- **CLI:** `--dom`

使用 happy-dom 模拟浏览器 API

### browser.enabled

- **CLI:** `--browser.enabled`
- **Config:** [browser.enabled](/guide/browser/config#browser-enabled)

在浏览器中运行测试。 相当于 `--browser.enabled` (默认值: `false`)

### browser.name

- **CLI:** `--browser.name <name>`
- **Config:** [browser.name](/guide/browser/config#browser-name)

<<<<<<< HEAD
在特定浏览器中运行所有测试。某些浏览器只适用于特定的 providers (比如 `--browser.provider`). 通过 [`browser.name`](https://vitest.dev/config/#browser-name) 查看更多信息
=======
Run all tests in a specific browser. Some browsers are only available for specific providers (see `--browser.provider`). Visit [`browser.name`](https://vitest.dev/guide/browser/config/#browser-name) for more information
>>>>>>> a846195377cb7746f3b7572999425757c7dcae53

### browser.headless

- **CLI:** `--browser.headless`
- **Config:** [browser.headless](/guide/browser/config#browser-headless)

在无头模式下运行浏览器（即不打开图形用户界面）。如果在 CI 中运行 Vitest，默认情况下将启用无头模式 (默认值: `process.env.CI`)

### browser.api.port

- **CLI:** `--browser.api.port [port]`
- **Config:** [browser.api.port](/guide/browser/config#browser-api-port)

指定服务器端口。注意，如果端口已被使用，Vite 会自动尝试下一个可用端口，因此这可能不是服务器最终监听的实际端口。如果为 `true`，将设置为 `63315`

### browser.api.host

- **CLI:** `--browser.api.host [host]`
- **Config:** [browser.api.host](/guide/browser/config#browser-api-host)

指定服务器应该监听哪些 IP 地址。设为 `0.0.0.0` 或 `true` 则监听所有地址，包括局域网地址和公共地址

### browser.api.strictPort

- **CLI:** `--browser.api.strictPort`
- **Config:** [browser.api.strictPort](/guide/browser/config#browser-api-strictport)

设置为 true 时，如果端口已被使用，则退出，而不是自动尝试下一个可用端口

### browser.provider

- **CLI:** `--browser.provider <name>`
- **Config:** [browser.provider](/guide/browser/config#browser-provider)

用于运行浏览器测试的 Provider。某些浏览器只适用于特定的提供 Provider，可以是"webdriverio", "playwright", "preview"，或自定义 provider. 通过 [`browser.provider`](https://vitest.dev/config/#browser-provider) 查看更多信息 (默认值: `"preview"`)

### browser.providerOptions

- **CLI:** `--browser.providerOptions <options>`
- **Config:** [browser.providerOptions](/guide/browser/config#browser-provideroptions)

传递给浏览器提供程序的选项。更多信息请访问 [`browser.providerOptions`](https://vitest.dev/config/#browser-provideroptions)。

### browser.isolate

- **CLI:** `--browser.isolate`
- **Config:** [browser.isolate](/guide/browser/config#browser-isolate)

隔离运行每个浏览器测试文件。要禁用隔离请使用 `--browser.isolate=false` (默认值: `true`)

### browser.ui

- **CLI:** `--browser.ui`
- **Config:** [browser.ui](/guide/browser/config#browser-ui)

运行测试时显示 Vitest UI(默认值: `!process.env.CI`)

### browser.fileParallelism

- **CLI:** `--browser.fileParallelism`
- **Config:** [browser.fileParallelism](/guide/browser/config#browser-fileparallelism)

浏览器测试文件是否应并行运行。使用 `--browser.fileParallelism=false` 可禁用 (默认值: `true`)

### pool

- **CLI:** `--pool <pool>`
- **Config:** [pool](/config/#pool)

如果未在浏览器中运行，则指定 pool (默认值: `threads`)

### poolOptions.threads.isolate

- **CLI:** `--poolOptions.threads.isolate`
- **Config:** [poolOptions.threads.isolate](/config/#pooloptions-threads-isolate)

在线程池中隔离测试 (默认值: `true`)

### poolOptions.threads.singleThread

- **CLI:** `--poolOptions.threads.singleThread`
- **Config:** [poolOptions.threads.singleThread](/config/#pooloptions-threads-singlethread)

在单线程内运行测试 (默认值: `false`)

### poolOptions.threads.maxThreads

- **CLI:** `--poolOptions.threads.maxThreads <workers>`
- **Config:** [poolOptions.threads.maxThreads](/config/#pooloptions-threads-maxthreads)

运行测试的最大线程数或百分比

### poolOptions.threads.minThreads

- **CLI:** `--poolOptions.threads.minThreads <workers>`
- **Config:** [poolOptions.threads.minThreads](/config/#pooloptions-threads-minthreads)

运行测试的最小线程数或百分比

### poolOptions.threads.useAtomics

- **CLI:** `--poolOptions.threads.useAtomics`
- **Config:** [poolOptions.threads.useAtomics](/config/#pooloptions-threads-useatomics)

使用 Atomics 同步线程。这在某些情况下可以提高性能，但在较旧的 Node 版本中可能会导致 segfault。 (默认值: `false`)

### poolOptions.vmThreads.isolate

- **CLI:** `--poolOptions.vmThreads.isolate`
- **Config:** [poolOptions.vmThreads.isolate](/config/#pooloptions-vmthreads-isolate)

在线程池中隔离测试 (默认值: `true`)

### poolOptions.vmThreads.singleThread

- **CLI:** `--poolOptions.vmThreads.singleThread`
- **Config:** [poolOptions.vmThreads.singleThread](/config/#pooloptions-vmthreads-singlethread)

在单线程内运行测试（默认值：`false`）

### poolOptions.vmThreads.maxThreads

- **CLI:** `--poolOptions.vmThreads.maxThreads <workers>`
- **Config:** [poolOptions.vmThreads.maxThreads](/config/#pooloptions-vmthreads-maxthreads)

 运行测试的最大线程数或百分比

### poolOptions.vmThreads.minThreads

- **CLI:** `--poolOptions.vmThreads.minThreads <workers>`
- **Config:** [poolOptions.vmThreads.minThreads](/config/#pooloptions-vmthreads-minthreads)

运行测试的最小线程数或百分比

### poolOptions.vmThreads.useAtomics

- **CLI:** `--poolOptions.vmThreads.useAtomics`
- **Config:** [poolOptions.vmThreads.useAtomics](/config/#pooloptions-vmthreads-useatomics)

使用 Atomics 同步线程。这在某些情况下可以提高性能，但在较旧的 Node 版本中可能会导致 segfault。 (默认值: `false`)

### poolOptions.vmThreads.memoryLimit

- **CLI:** `--poolOptions.vmThreads.memoryLimit <limit>`
- **Config:** [poolOptions.vmThreads.memoryLimit](/config/#pooloptions-vmthreads-memorylimit)

虚拟机线程池的内存限制。如果发现内存泄漏，请尝试调整该值。

### poolOptions.forks.isolate

- **CLI:** `--poolOptions.forks.isolate`
- **Config:** [poolOptions.forks.isolate](/config/#pooloptions-forks-isolate)

在 forks pool 中隔离测试 (默认值: `true`)

### poolOptions.forks.singleFork

- **CLI:** `--poolOptions.forks.singleFork`
- **Config:** [poolOptions.forks.singleFork](/config/#pooloptions-forks-singlefork)

单个子进程内运行测试 (default: `false`)

### poolOptions.forks.maxForks

- **CLI:** `--poolOptions.forks.maxForks <workers>`
- **Config:** [poolOptions.forks.maxForks](/config/#pooloptions-forks-maxforks)

运行测试的最大进程数

### poolOptions.forks.minForks

- **CLI:** `--poolOptions.forks.minForks <workers>`
- **Config:** [poolOptions.forks.minForks](/config/#pooloptions-forks-minforks)

运行测试的最小进程数

### poolOptions.vmForks.isolate

- **CLI:** `--poolOptions.vmForks.isolate`
- **Config:** [poolOptions.vmForks.isolate](/config/#pooloptions-vmforks-isolate)

在 forks pool 中隔离测试 (default: `true`)

### poolOptions.vmForks.singleFork

- **CLI:** `--poolOptions.vmForks.singleFork`
- **Config:** [poolOptions.vmForks.singleFork](/config/#pooloptions-vmforks-singlefork)

在单个子进程内运行测试 (default: `false`)

### poolOptions.vmForks.maxForks

- **CLI:** `--poolOptions.vmForks.maxForks <workers>`
- **Config:** [poolOptions.vmForks.maxForks](/config/#pooloptions-vmforks-maxforks)

运行测试的最大进程数

### poolOptions.vmForks.minForks

- **CLI:** `--poolOptions.vmForks.minForks <workers>`
- **Config:** [poolOptions.vmForks.minForks](/config/#pooloptions-vmforks-minforks)

运行测试的最小进程数

### poolOptions.vmForks.memoryLimit

- **CLI:** `--poolOptions.vmForks.memoryLimit <limit>`
- **Config:** [poolOptions.vmForks.memoryLimit](/config/#pooloptions-vmforks-memorylimit)

VM forks pool 的内存限制。如果你观察到内存泄漏问题，可以尝试调整这个值。

### fileParallelism

- **CLI:** `--fileParallelism`
- **Config:** [fileParallelism](/config/#fileparallelism)

是否所有测试文件都应并行运行. 使用 `--no-file-parallelism` 去禁用 (默认值: `true`)

### maxWorkers

- **CLI:** `--maxWorkers <workers>`
- **Config:** [maxWorkers](/config/#maxworkers)

同时并发执行测试任务的最大线程数或百分比

### minWorkers

- **CLI:** `--minWorkers <workers>`
- **Config:** [minWorkers](/config/#minworkers)

同时并发执行测试任务的最小线程数或百分比

### environment

- **CLI:** `--environment <name>`
- **Config:** [environment](/config/#environment)

如果不在浏览器中运行，则指定运行环境 (默认值: `node`)

### passWithNoTests

- **CLI:** `--passWithNoTests`
- **Config:** [passWithNoTests](/config/#passwithnotests)

未发现测试时通过

### logHeapUsage

- **CLI:** `--logHeapUsage`
- **Config:** [logHeapUsage](/config/#logheapusage)

在节点中运行时，显示每个测试的堆大小

### allowOnly

- **CLI:** `--allowOnly`
- **Config:** [allowOnly](/config/#allowonly)

允许执行那些被标记为"only"的测试用例或测试套件 (默认值: `!process.env.CI`)

### dangerouslyIgnoreUnhandledErrors

- **CLI:** `--dangerouslyIgnoreUnhandledErrors`
- **Config:** [dangerouslyIgnoreUnhandledErrors](/config/#dangerouslyignoreunhandlederrors)

忽略任何未处理的错误

### sequence.shuffle.files

- **CLI:** `--sequence.shuffle.files`
- **Config:** [sequence.shuffle.files](/config/#sequence-shuffle-files)

以随机顺序运行文件。如果启用此选项，长时间运行的测试将不会提前开始。 (默认值: `false`)

### sequence.shuffle.tests

- **CLI:** `--sequence.shuffle.tests`
- **Config:** [sequence.shuffle.tests](/config/#sequence-shuffle-tests)

以随机方式运行测试（默认值：`false`）

### sequence.concurrent

- **CLI:** `--sequence.concurrent`
- **Config:** [sequence.concurrent](/config/#sequence-concurrent)

使测试并行运行（默认值：`false`）

### sequence.seed

- **CLI:** `--sequence.seed <seed>`
- **Config:** [sequence.seed](/config/#sequence-seed)

设置随机化种子。如果 --sequence.shuffle（随机序列）是`false`，则此选项无效。 t 通过 ["Random Seed" page](https://en.wikipedia.org/wiki/Random_seed) 查看更多信息

### sequence.hooks

- **CLI:** `--sequence.hooks <order>`
- **Config:** [sequence.hooks](/config/#sequence-hooks)

更改钩子的执行顺序。 可接受的值有: "stack", "list" and "parallel". 通过 [`sequence.hooks`](https://vitest.dev/config/#sequence-hooks) 查看更多信息 (默认值: `"parallel"`)

### sequence.setupFiles

- **CLI:** `--sequence.setupFiles <order>`
- **Config:** [sequence.setupFiles](/config/#sequence-setupfiles)

 更改设置文件的执行顺序。可接受的值有 "list" 和 "parallel"。如果设置为"list"，将按照定义的顺序运行设置文件。如果设置为 "parallel"，将并行运行设置文件（默认值：`"parallel"`）。

### inspect

- **CLI:** `--inspect [[host:]port]`
- **Config:** [inspect](/config/#inspect)

启用 Node.js 检查器（默认值：`127.0.0.1:9229`）

### inspectBrk

- **CLI:** `--inspectBrk [[host:]port]`
- **Config:** [inspectBrk](/config/#inspectbrk)

启用 Node.js 检查器并在测试开始前中断

### testTimeout

- **CLI:** `--testTimeout <timeout>`
- **Config:** [testTimeout](/config/#testtimeout)

测试的默认超时（毫秒）（默认值：`5000`）。

### hookTimeout

- **CLI:** `--hookTimeout <timeout>`
- **Config:** [hookTimeout](/config/#hooktimeout)

默认钩子超时（以毫秒为单位）（默认值：`10000`）

### bail

- **CLI:** `--bail <number>`
- **Config:** [bail](/config/#bail)

当指定数量的测试失败时停止测试执行（默认值：`0`）

### retry

- **CLI:** `--retry <times>`
- **Config:** [retry](/config/#retry)

如果测试失败，重试特定次数（默认值： `0`）。

### diff.aAnnotation

- **CLI:** `--diff.aAnnotation <annotation>`
- **Config:** [diff.aAnnotation](/config/#diff-aannotation)

Annotation for expected lines (default: `Expected`)

### diff.aIndicator

- **CLI:** `--diff.aIndicator <indicator>`
- **Config:** [diff.aIndicator](/config/#diff-aindicator)

Indicator for expected lines (default: `-`)

### diff.bAnnotation

- **CLI:** `--diff.bAnnotation <annotation>`
- **Config:** [diff.bAnnotation](/config/#diff-bannotation)

Annotation for received lines (default: `Received`)

### diff.bIndicator

- **CLI:** `--diff.bIndicator <indicator>`
- **Config:** [diff.bIndicator](/config/#diff-bindicator)

Indicator for received lines (default: `+`)

### diff.commonIndicator

- **CLI:** `--diff.commonIndicator <indicator>`
- **Config:** [diff.commonIndicator](/config/#diff-commonindicator)

Indicator for common lines (default: ` `)

### diff.contextLines

- **CLI:** `--diff.contextLines <lines>`
- **Config:** [diff.contextLines](/config/#diff-contextlines)

Number of lines of context to show around each change (default: `5`)

### diff.emptyFirstOrLastLinePlaceholder

- **CLI:** `--diff.emptyFirstOrLastLinePlaceholder <placeholder>`
- **Config:** [diff.emptyFirstOrLastLinePlaceholder](/config/#diff-emptyfirstorlastlineplaceholder)

Placeholder for an empty first or last line (default: `""`)

### diff.expand

- **CLI:** `--diff.expand`
- **Config:** [diff.expand](/config/#diff-expand)

Expand all common lines (default: `true`)

### diff.includeChangeCounts

- **CLI:** `--diff.includeChangeCounts`
- **Config:** [diff.includeChangeCounts](/config/#diff-includechangecounts)

Include comparison counts in diff output (default: `false`)

### diff.omitAnnotationLines

- **CLI:** `--diff.omitAnnotationLines`
- **Config:** [diff.omitAnnotationLines](/config/#diff-omitannotationlines)

Omit annotation lines from the output (default: `false`)

### diff.printBasicPrototype

- **CLI:** `--diff.printBasicPrototype`
- **Config:** [diff.printBasicPrototype](/config/#diff-printbasicprototype)

Print basic prototype Object and Array (default: `true`)

### diff.truncateThreshold

- **CLI:** `--diff.truncateThreshold <threshold>`
- **Config:** [diff.truncateThreshold](/config/#diff-truncatethreshold)

Number of lines to show before and after each change (default: `0`)

### diff.truncateAnnotation

- **CLI:** `--diff.truncateAnnotation <annotation>`
- **Config:** [diff.truncateAnnotation](/config/#diff-truncateannotation)

Annotation for truncated lines (default: `... Diff result is truncated`)

### exclude

- **CLI:** `--exclude <glob>`
- **Config:** [exclude](/config/#exclude)

测试中排除的其他文件路径匹配模式

### expandSnapshotDiff

- **CLI:** `--expandSnapshotDiff`
- **Config:** [expandSnapshotDiff](/config/#expandsnapshotdiff)

快照失败时显示完整差异

### disableConsoleIntercept

- **CLI:** `--disableConsoleIntercept`
- **Config:** [disableConsoleIntercept](/config/#disableconsoleintercept)

禁用自动拦截控制台日志（默认值：`false`）

### typecheck.enabled

- **CLI:** `--typecheck.enabled`
- **Config:** [typecheck.enabled](/config/#typecheck-enabled)

在测试的同时启用类型检查（默认值：`false`）

### typecheck.only

- **CLI:** `--typecheck.only`
- **Config:** [typecheck.only](/config/#typecheck-only)

仅运行类型检查测试。这将自动启用类型检查（默认值：`false`）

### typecheck.checker

- **CLI:** `--typecheck.checker <name>`
- **Config:** [typecheck.checker](/config/#typecheck-checker)

指定要使用的类型检查器。可用值为 "tsc"和 "vue-tsc "以及一个可执行文件的路径（默认值：`tsc`）

### typecheck.allowJs

- **CLI:** `--typecheck.allowJs`
- **Config:** [typecheck.allowJs](/config/#typecheck-allowjs)

允许对 JavaScript 文件进行类型检查。默认值取自 tsconfig.json

### typecheck.ignoreSourceErrors

- **CLI:** `--typecheck.ignoreSourceErrors`
- **Config:** [typecheck.ignoreSourceErrors](/config/#typecheck-ignoresourceerrors)

忽略源文件中的类型错误

### typecheck.tsconfig

- **CLI:** `--typecheck.tsconfig <path>`
- **Config:** [typecheck.tsconfig](/config/#typecheck-tsconfig)

自定义 tsconfig 文件的路径

### project

- **CLI:** `--project <name>`
- **Config:** [project](/config/#project)

如果我们正在使用 Vitest 的工作区功能，这是要运行的项目名称。这个参数可以重复以指定多个项目：`--project=1 --project=2`。我们还可以使用通配符来过滤项目，例如 `--project=packages*`，以及使用 `--project=!pattern` 来排除项目。

### slowTestThreshold

- **CLI:** `--slowTestThreshold <threshold>`
- **Config:** [slowTestThreshold](/config/#slowtestthreshold)

测试速度慢的阈值（以毫秒为单位）（默认值：`300`）

### teardownTimeout

- **CLI:** `--teardownTimeout <timeout>`
- **Config:** [teardownTimeout](/config/#teardowntimeout)

拆卸函数的默认超时（以毫秒为单位）（默认值：`10000`）

### maxConcurrency

- **CLI:** `--maxConcurrency <number>`
- **Config:** [maxConcurrency](/config/#maxconcurrency)

套件中并发测试的最大次数（默认值：`5`）

### expect.requireAssertions

- **CLI:** `--expect.requireAssertions`
- **Config:** [expect.requireAssertions](/config/#expect-requireassertions)

要求所有测试至少有一个断言

### expect.poll.interval

- **CLI:** `--expect.poll.interval <interval>`
- **Config:** [expect.poll.interval](/config/#expect-poll-interval)

断言的轮询间隔 `expect.poll()` (默认值: `50`)

### expect.poll.timeout

- **CLI:** `--expect.poll.timeout <timeout>`
- **Config:** [expect.poll.timeout](/config/#expect-poll-timeout)

断言的轮询超时（以毫秒为单位） `expect.poll()` (默认值: `1000`)

### printConsoleTrace

- **CLI:** `--printConsoleTrace`
- **Config:** [printConsoleTrace](/config/#printconsoletrace)

 始终打印控制台堆栈跟踪

### includeTaskLocation

- **CLI:** `--includeTaskLocation`
- **Config:** [includeTaskLocation](/config/#includetasklocation)

Collect test and suite locations in the `location` property

### run

- **CLI:** `--run`

禁用 watch 模式

### color

- **CLI:** `--no-color`

删除控制台输出中的颜色

### clearScreen

- **CLI:** `--clearScreen`

 watch 模式下重新运行测试时清除终端屏幕（默认值：`true`）。

### standalone

- **CLI:** `--standalone`

启动 Vitest 而不运行测试。文件过滤器将被忽略，只有在发生变化时才会运行测试。(默认值:`false`)
