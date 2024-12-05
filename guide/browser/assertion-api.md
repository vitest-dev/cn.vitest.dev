---
title: Assertion API | Browser Mode
---

# Assertion API

Vitest 捆绑了 [`@testing-library/jest-dom`](https://github.com/testing-library/jest-dom)库，以提供各种开箱即用的 DOM 断言。有关详细文档，请阅读 `jest-dom` readme：

- [`toBeDisabled`](https://github.com/testing-library/jest-dom#toBeDisabled)
- [`toBeEnabled`](https://github.com/testing-library/jest-dom#toBeEnabled)
- [`toBeEmptyDOMElement`](https://github.com/testing-library/jest-dom#toBeEmptyDOMElement)
- [`toBeInTheDocument`](https://github.com/testing-library/jest-dom#toBeInTheDocument)
- [`toBeInvalid`](https://github.com/testing-library/jest-dom#toBeInvalid)
- [`toBeRequired`](https://github.com/testing-library/jest-dom#toBeRequired)
- [`toBeValid`](https://github.com/testing-library/jest-dom#toBeValid)
- [`toBeVisible`](https://github.com/testing-library/jest-dom#toBeVisible)
- [`toContainElement`](https://github.com/testing-library/jest-dom#toContainElement)
- [`toContainHTML`](https://github.com/testing-library/jest-dom#toContainHTML)
- [`toHaveAccessibleDescription`](https://github.com/testing-library/jest-dom#toHaveAccessibleDescription)
- [`toHaveAccessibleErrorMessage`](https://github.com/testing-library/jest-dom#toHaveAccessibleErrorMessage)
- [`toHaveAccessibleName`](https://github.com/testing-library/jest-dom#toHaveAccessibleName)
- [`toHaveAttribute`](https://github.com/testing-library/jest-dom#toHaveAttribute)
- [`toHaveClass`](https://github.com/testing-library/jest-dom#toHaveClass)
- [`toHaveFocus`](https://github.com/testing-library/jest-dom#toHaveFocus)
- [`toHaveFormValues`](https://github.com/testing-library/jest-dom#toHaveFormValues)
- [`toHaveStyle`](https://github.com/testing-library/jest-dom#toHaveStyle)
- [`toHaveTextContent`](https://github.com/testing-library/jest-dom#toHaveTextContent)
- [`toHaveValue`](https://github.com/testing-library/jest-dom#toHaveValue)
- [`toHaveDisplayValue`](https://github.com/testing-library/jest-dom#toHaveDisplayValue)
- [`toBeChecked`](https://github.com/testing-library/jest-dom#toBeChecked)
- [`toBePartiallyChecked`](https://github.com/testing-library/jest-dom#toBePartiallyChecked)
- [`toHaveRole`](https://github.com/testing-library/jest-dom#toHaveRole)
- [`toHaveErrorMessage`](https://github.com/testing-library/jest-dom#toHaveErrorMessage)

<<<<<<< HEAD
如果我们正在使用 [TypeScript](/guide/browser/#typescript) 或者希望在 `expect` 中有正确的类型提示，请确保我们的 [setup 文件](/config/#setupfile)或 [config 文件](/config/file)中引用了 `@vitest/browser/providers/playwright` 或 `@vitest/browser/providers/webdriverio` ，具体取决于我们使用的提供者。如果使用的是默认的 `preview` 提供，可以改为 `@vitest/browser/matchers` 。
=======
If you are using [TypeScript](/guide/browser/#typescript) or want to have correct type hints in `expect`, make sure you have either `@vitest/browser/providers/playwright` or `@vitest/browser/providers/webdriverio` referenced in your [setup file](/config/#setupfile) or a [config file](/config/) depending on the provider you use. If you use the default `preview` provider, you can specify `@vitest/browser/matchers` instead.
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f

::: code-group
```ts [preview]
/// <reference types="@vitest/browser/matchers" />
```
```ts [playwright]
/// <reference types="@vitest/browser/providers/playwright" />
```
```ts [webdriverio]
/// <reference types="@vitest/browser/providers/webdriverio" />
```
:::

浏览器中的测试由于其异步特性，可能会不一致地失败。因此，即使条件延迟（如超时、网络请求或动画），也必须有办法保证断言成功。为此，Vitest 通过 [`expect.poll`](/api/expect#poll)和 `expect.element` API 提供了可重试的断言：

```ts
import { expect, test } from 'vitest'
import { page } from '@vitest/browser/context'

test('error banner is rendered', async () => {
  triggerError()

  // @testing-library 提供内置重试功能的查询
  // 它会尝试找到 banner，直到它渲染出来
  const banner = await page.getByRole('alert', {
    name: /error/i,
  })

  // Vitest 提供内置重试功能的 `expect.element`
  // 它会检查 `element.textContent` 直到等于 “Error!”。
  await expect.element(banner).toHaveTextContent('Error!')
})
```

::: tip
`expect.element` 是 `expect.poll(() => element)`的简写，工作方式完全相同。

`toHaveTextContent` 和所有其他 [`@testing-library/jest-dom`](https://github.com/testing-library/jest-dom)断言在没有内置重试机制的常规`expect`中仍然可用：

```ts
// 如果 .textContent 不是 `'Error!'`，则会立即失败。
expect(banner).toHaveTextContent('Error!')
```
:::
