---
title: Context API | Browser Mode
---

# 上下文

Vitest 通过 `@vitest/browser/context` 入口点公开上下文模块。从 2.0 开始，它公开了一小部分实用程序，这些实用程序可能在测试中对你有用。

## `userEvent`

::: tip
`userEvent` API 的详细说明见[Interactivity API](/guide/browser/interactivity-api).
:::

```ts
/**
 * Handler for user interactions. The support is implemented by the browser provider (`playwright` or `webdriverio`).
 * If used with `preview` provider, fallbacks to simulated events via `@testing-library/user-event`.
 * @experimental
 */
export const userEvent: {
  setup: () => UserEvent
  cleanup: () => Promise<void>
  click: (element: Element, options?: UserEventClickOptions) => Promise<void>
  dblClick: (element: Element, options?: UserEventDoubleClickOptions) => Promise<void>
  tripleClick: (element: Element, options?: UserEventTripleClickOptions) => Promise<void>
  selectOptions: (
    element: Element,
    values: HTMLElement | HTMLElement[] | string | string[],
    options?: UserEventSelectOptions,
  ) => Promise<void>
  keyboard: (text: string) => Promise<void>
  type: (element: Element, text: string, options?: UserEventTypeOptions) => Promise<void>
  clear: (element: Element) => Promise<void>
  tab: (options?: UserEventTabOptions) => Promise<void>
  hover: (element: Element, options?: UserEventHoverOptions) => Promise<void>
  unhover: (element: Element, options?: UserEventHoverOptions) => Promise<void>
  fill: (element: Element, text: string, options?: UserEventFillOptions) => Promise<void>
  dragAndDrop: (source: Element, target: Element, options?: UserEventDragAndDropOptions) => Promise<void>
}
```

## `commands`

::: tip
Commands API 的详细说明见[Commands API](/guide/browser/commands).
:::

```ts
/**
 * Available commands for the browser.
 * A shortcut to `server.commands`.
 */
export const commands: BrowserCommands
```

## `page`

页面导出提供了与当前页面交互的实用程序。

::: warning
虽然它从 Playwright 的 `page` 中获取了一些实用程序，但它与 Playwright 的 `page` 并不是同一个对象。由于浏览器上下文是在浏览器中评估的，您的测试无法访问 Playwright 的 `page`，因为它是在服务器上运行的。
:::

使用 [Commands API](/guide/browser/commands) 如果您需要访问 Playwright 的 `page` 对象。
```ts
export const page: {
  /**
   * Change the size of iframe's viewport.
   */
  viewport: (width: number, height: number) => Promise<void>
  /**
   * Make a screenshot of the test iframe or a specific element.
   * @returns Path to the screenshot file or path and base64.
   */
  screenshot: ((options: Omit<ScreenshotOptions, 'base64'> & { base64: true }) => Promise<{
    path: string
    base64: string
  }>) & ((options?: ScreenshotOptions) => Promise<string>)
  /**
   * Extend default `page` object with custom methods.
   */
  extend: (methods: Partial<BrowserPage>) => BrowserPage
  /**
   * Wrap an HTML element in a `Locator`. When querying for elements, the search will always return this element.
   */
  elementLocator: (element: Element) => Locator

  /**
   * Locator APIs. See its documentation for more details.
   */
  getByRole: (role: ARIARole | string, options?: LocatorByRoleOptions) => Locator
  getByLabelText: (text: string | RegExp, options?: LocatorOptions) => Locator
  getByTestId: (text: string | RegExp) => Locator
  getByAltText: (text: string | RegExp, options?: LocatorOptions) => Locator
  getByPlaceholder: (text: string | RegExp, options?: LocatorOptions) => Locator
  getByText: (text: string | RegExp, options?: LocatorOptions) => Locator
  getByTitle: (text: string | RegExp, options?: LocatorOptions) => Locator
}
```

::: tip
The `getBy*` API is explained at [Locators API](/guide/browser/locators).
:::

## `cdp`

`cdp` 导出返回当前的 Chrome DevTools 协议会话。它主要用于库作者在其基础上构建工具。

::: warning
CDP 会话仅适用于 `playwright` provider，并且仅在使用 `chromium` 浏览器时有效。有关详细信息，请参阅 playwright 的 [`CDPSession`](https://playwright.dev/docs/api/class-cdpsession)文档。
:::

```ts
export const cdp: () => CDPSession
```

## `server`

`server` 导出表示运行 Vitest 服务器的 Node.js 环境。它主要用于调试或根据环境限制测试。

```ts
export const server: {
  /**
   * Vitest 服务运行的平台。
   * 与在服务上调用 `process.platform` 相同。
   */
  platform: Platform
  /**
   * Vitest 服务的运行版本。
   * 与在服务上调用 `process.version` 相同。
   */
  version: string
  /**
   *  browser provider 的名字.
   */
  provider: string
  /**
   * 当前浏览器的名字。
   */
  browser: string
  /**
   * 浏览器的可用命令。
   */
  commands: BrowserCommands
  /**
   * 序列化测试配置。
   */
  config: SerializedConfig
}
```
