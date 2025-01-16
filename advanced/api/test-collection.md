# TestCollection

`TestCollection` 表示套件或模块中顶级 [suite](/advanced/api/test-suite) 和 [test](/advanced/api/test-case) 的集合。它还提供了有用的方法来迭代自身。

::: info
大多数方法返回迭代器而不是数组，以在你不需使用集合中的每个项目时提高性能。如果你更喜欢使用数组，可以展开迭代器：`[...children.allSuites()]`。

另外请注意，集合本身也是一个迭代器：

```ts
for (const child of module.children) {
  console.log(child.type, child.name)
}
```
:::

## size

集合中的测试和套件数量。

::: warning
此数量仅包括顶级的测试和套件，不包括嵌套的套件和测试。
:::

## at

```ts
function at(index: number): TestCase | TestSuite | undefined
```

返回位于特定索引处的测试或套件。此方法接受负数索引。

## array

```ts
function array(): (TestCase | TestSuite)[]
```

相同的集合，但以数组形式返回。如果你想要使用 `map` 和 `filter` 等 `Array` 方法，而这些方法不受 `TaskCollection` 实现的支持时，这将非常有用。

## allSuites

```ts
function allSuites(): Generator<TestSuite, undefined, void>
```

过滤出属于此集合及其子集的所有套件。

```ts
for (const suite of module.children.allSuites()) {
  if (suite.errors().length) {
    console.log('failed to collect', suite.errors())
  }
}
```

## allTests

```ts
function allTests(state?: TestState): Generator<TestCase, undefined, void>
```

过滤出属于此集合及其子集的所有测试。

```ts
for (const test of module.children.allTests()) {
  if (test.result().state === 'pending') {
    console.log('test', test.fullName, 'did not finish')
  }
}
```

你可以传递一个 `state` 值来根据状态过滤测试。

## tests

```ts
function tests(state?: TestState): Generator<TestCase, undefined, void>
```

仅过滤属于此集合的测试。你可以传递一个 `state` 值来根据状态过滤测试。

## suites

```ts
function suites(): Generator<TestSuite, undefined, void>
```

仅过滤属于此集合的套件。
