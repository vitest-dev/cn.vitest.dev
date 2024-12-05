---
title: Interactivity API | Browser Mode
---

# Interactivity API

Vitest 使用 [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) 或 [webdriver](https://www.w3.org/TR/webdriver/) API 实现了 [`@testing-library/user-event`](https://testing-library.com/docs/user-event)应用程序接口的子集，而不是伪造事件，这使得浏览器行为更加可靠和一致。

```ts
import { userEvent } from '@vitest/browser/context'

await userEvent.click(document.querySelector('.button'))
```

<<<<<<< HEAD
几乎所有的 `userEvent` 方法都继承了其 provider 的选项。要在 IDE 中查看所有可用的选项，请根据我们的 provider ，将 `webdriver` 或 `playwright` 类型添加到 [ setup 文件](/config/#setupfile)或 [config 文件](/config/file)中（取决于 `tsconfig.json` 中 `included` 的内容）：
=======
Almost every `userEvent` method inherits its provider options. To see all available options in your IDE, add `webdriver` or `playwright` types (depending on your provider) to your [setup file](/config/#setupfile) or a [config file](/config/) (depending on what is in `included` in your `tsconfig.json`):
>>>>>>> e35f3fa69f314a1d985a5c9a104038fe3fb3446f

::: code-group
```ts [playwright]
/// <reference types="@vitest/browser/providers/playwright" />
```
```ts [webdriverio]
/// <reference types="@vitest/browser/providers/webdriverio" />
```
:::

## userEvent.setup

```ts
function setup(): UserEvent
```

创建一个新的用户事件实例。如果需要保持键盘状态，以便正确按下和释放按钮，这将非常有用。

::: warning
与 `@testing-library/user-event` 不同，来自 `@vitest/browser/context` 的默认 `userEvent` 实例只创建一次，而不是每次调用其方法时都创建一次！您可以从本代码段中看到其工作方式的不同之处：

```ts
import { userEvent as vitestUserEvent } from '@vitest/browser/context'
import { userEvent as originalUserEvent } from '@testing-library/user-event'

await vitestUserEvent.keyboard('{Shift}') // press shift without releasing
await vitestUserEvent.keyboard('{/Shift}') // releases shift

await originalUserEvent.keyboard('{Shift}') // press shift without releasing
await originalUserEvent.keyboard('{/Shift}') // DID NOT release shift because the state is different
```

这种行为更有用，因为我们并没有模拟键盘，而是实际按下了 Shift 键，所以保留原来的行为会在字段中键入时造成意想不到的问题。
:::

## userEvent.click

```ts
function click(
  element: Element | Locator,
  options?: UserEventClickOptions,
): Promise<void>
```

点击元素。继承 provider 的选项。有关此方法如何工作的详细说明，请参阅 provider 的文档。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('clicks on an element', async () => {
  const logo = page.getByRole('img', { name: /logo/ })

  await userEvent.click(logo)
  // or you can access it directly on the locator
  await logo.click()
})
```

References:

- [Playwright `locator.click` API](https://playwright.dev/docs/api/class-locator#locator-click)
- [WebdriverIO `element.click` API](https://webdriver.io/docs/api/element/click/)
- [testing-library `click` API](https://testing-library.com/docs/user-event/convenience/#click)

## userEvent.dblClick

```ts
function dblClick(
  element: Element | Locator,
  options?: UserEventDoubleClickOptions,
): Promise<void>
```

触发元素的双击事件

请参阅你的 provider 的文档以获取有关此方法如何工作的详细说明。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('triggers a double click on an element', async () => {
  const logo = page.getByRole('img', { name: /logo/ })

  await userEvent.dblClick(logo)
  // or you can access it directly on the locator
  await logo.dblClick()
})
```

References:

- [Playwright `locator.dblclick` API](https://playwright.dev/docs/api/class-locator#locator-dblclick)
- [WebdriverIO `element.doubleClick` API](https://webdriver.io/docs/api/element/doubleClick/)
- [testing-library `dblClick` API](https://testing-library.com/docs/user-event/convenience/#dblClick)

## userEvent.tripleClick

```ts
function tripleClick(
  element: Element | Locator,
  options?: UserEventTripleClickOptions,
): Promise<void>
```

Triggers a triple click event on an element. Since there is no `tripleclick` in browser api, this method will fire three click events in a row, and so you must check [click event detail](https://developer.mozilla.org/en-US/docs/Web/API/Element/click_event#usage_notes) to filter the event: `evt.detail === 3`.

Please refer to your provider's documentation for detailed explanation about how this method works.

```ts
import { page, userEvent } from '@vitest/browser/context'

test('triggers a triple click on an element', async () => {
  const logo = page.getByRole('img', { name: /logo/ })
  let tripleClickFired = false
  logo.addEventListener('click', (evt) => {
    if (evt.detail === 3) {
      tripleClickFired = true
    }
  })

  await userEvent.tripleClick(logo)
  // or you can access it directly on the locator
  await logo.tripleClick()

  expect(tripleClickFired).toBe(true)
})
```

References:

- [Playwright `locator.click` API](https://playwright.dev/docs/api/class-locator#locator-click): implemented via `click` with `clickCount: 3` .
- [WebdriverIO `browser.action` API](https://webdriver.io/docs/api/browser/action/): implemented via actions api with `move` plus three `down + up + pause` events in a row
- [testing-library `tripleClick` API](https://testing-library.com/docs/user-event/convenience/#tripleClick)

## userEvent.fill

```ts
function fill(
  element: Element | Locator,
  text: string,
): Promise<void>
```

为 `input/textarea/conteneditable` 字段设置值。这将在设置新值前移除输入中的任何现有文本。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('update input', async () => {
  const input = page.getByRole('input')

  await userEvent.fill(input, 'foo') // input.value == foo
  await userEvent.fill(input, '{{a[[') // input.value == {{a[[
  await userEvent.fill(input, '{Shift}') // input.value == {Shift}

  // or you can access it directly on the locator
  await input.fill('foo') // input.value == foo
})
```

该方法聚焦元素、填充元素并在填充后触发一个 `input` 事件。您可以使用空字符串来清除字段。

::: tip
该 API 比使用 [`userEvent.type`](#userevent-type) 或 [`userEvent.keyboard`](#userevent-keyboard) 更快，但**不支持** [user-event `keyboard` syntax](https://testing-library.com/docs/user-event/keyboard) （例如，`{Shift}{selectall}`）。

在不需要输入特殊字符或对按键事件进行细粒度控制的情况下，我们建议使用此 API 而不是 [`userEvent.type`](#userevent-type)。
:::

References:

- [Playwright `locator.fill` API](https://playwright.dev/docs/api/class-locator#locator-fill)
- [WebdriverIO `element.setValue` API](https://webdriver.io/docs/api/element/setValue)
- [testing-library `type` API](https://testing-library.com/docs/user-event/utility/#type)

## userEvent.keyboard

```ts
function keyboard(text: string): Promise<void>
```

通过 `userEvent.keyboard` 可以触发键盘输入。如果任何输入有焦点，它就会在该输入中键入字符。否则，它将触发当前焦点元素（如果没有焦点元素，则为 `document.body`）上的键盘事件。

This API supports [user-event `keyboard` syntax](https://testing-library.com/docs/user-event/keyboard).

```ts
import { userEvent } from '@vitest/browser/context'

test('trigger keystrokes', async () => {
  await userEvent.keyboard('foo') // translates to: f, o, o
  await userEvent.keyboard('{{a[[') // translates to: {, a, [
  await userEvent.keyboard('{Shift}{f}{o}{o}') // translates to: Shift, f, o, o
  await userEvent.keyboard('{a>5}') // press a without releasing it and trigger 5 keydown
  await userEvent.keyboard('{a>5/}') // press a for 5 keydown and then release it
})
```

References:

- [Playwright `Keyboard` API](https://playwright.dev/docs/api/class-keyboard)
- [WebdriverIO `action('key')` API](https://webdriver.io/docs/api/browser/action#key-input-source)
- [testing-library `type` API](https://testing-library.com/docs/user-event/utility/#type)

## userEvent.tab

```ts
function tab(options?: UserEventTabOptions): Promise<void>
```

发送一个 `Tab` 键事件。这是`userEvent.keyboard('{tab}')`的简写。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('tab works', async () => {
  const [input1, input2] = page.getByRole('input').elements()

  expect(input1).toHaveFocus()

  await userEvent.tab()

  expect(input2).toHaveFocus()

  await userEvent.tab({ shift: true })

  expect(input1).toHaveFocus()
})
```

References:

- [Playwright `Keyboard` API](https://playwright.dev/docs/api/class-keyboard)
- [WebdriverIO `action('key')` API](https://webdriver.io/docs/api/browser/action#key-input-source)
- [testing-library `tab` API](https://testing-library.com/docs/user-event/convenience/#tab)

## userEvent.type

```ts
function type(
  element: Element | Locator,
  text: string,
  options?: UserEventTypeOptions,
): Promise<void>
```

::: warning
如果不依赖 [special characters](https://testing-library.com/docs/user-event/keyboard)（例如，`{shift}` 或 `{selectall}`），建议使用 [`userEvent.fill`](#userevent-fill)。
:::

`type` 方法在 [`keyboard`](https://testing-library.com/docs/user-event/keyboard) API 的基础上实现了 `@testing-library/user-event` 的 [`type`](https://testing-library.com/docs/user-event/utility/#type) 工具。

该函数允许您在 input/textarea/conteneditable 中键入字符。它支持 [user-event `keyboard` syntax](https://testing-library.com/docs/user-event/keyboard)。

如果只需按下字符而无需输入，请使用 [`userEvent.keyboard`](#userevent-keyboard) API。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('update input', async () => {
  const input = page.getByRole('input')

  await userEvent.type(input, 'foo') // input.value == foo
  await userEvent.type(input, '{{a[[') // input.value == foo{a[
  await userEvent.type(input, '{Shift}') // input.value == foo{a[
})
```

::: info
Vitest 没有像 `input.type` 那样在定位器上公开 `.type` 方法，因为它的存在只是为了与 `userEvent` 库兼容。请考虑使用 `.fill`，因为它更快。
:::

References:

- [Playwright `locator.press` API](https://playwright.dev/docs/api/class-locator#locator-press)
- [WebdriverIO `action('key')` API](https://webdriver.io/docs/api/browser/action#key-input-source)
- [testing-library `type` API](https://testing-library.com/docs/user-event/utility/#type)

## userEvent.clear

```ts
function clear(element: Element | Locator): Promise<void>
```

此方法会清除输入元素的内容。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('clears input', async () => {
  const input = page.getByRole('input')

  await userEvent.fill(input, 'foo')
  expect(input).toHaveValue('foo')

  await userEvent.clear(input)
  // or you can access it directly on the locator
  await input.clear()

  expect(input).toHaveValue('')
})
```

References:

- [Playwright `locator.clear` API](https://playwright.dev/docs/api/class-locator#locator-clear)
- [WebdriverIO `element.clearValue` API](https://webdriver.io/docs/api/element/clearValue)
- [testing-library `clear` API](https://testing-library.com/docs/user-event/utility/#clear)

## userEvent.selectOptions

```ts
function selectOptions(
  element: Element | Locator,
  values:
    | HTMLElement
    | HTMLElement[]
    | Locator
    | Locator[]
    | string
    | string[],
  options?: UserEventSelectOptions,
): Promise<void>
```

The `userEvent.selectOptions` allows selecting a value in a `<select>` element.

::: warning
如果 select 元素没有 [`multiple`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#attr-multiple) 属性，Vitest 将只选择数组中的第一个元素。

与 `@testing-library` 不同，Vitest 目前不支持 [listbox](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/listbox_role)，但我们计划在将来添加对它的支持。
:::

```ts
import { page, userEvent } from '@vitest/browser/context'

test('clears input', async () => {
  const select = page.getByRole('select')

  await userEvent.selectOptions(select, 'Option 1')
  // or you can access it directly on the locator
  await select.selectOptions('Option 1')

  expect(select).toHaveValue('option-1')

  await userEvent.selectOptions(select, 'option-1')
  expect(select).toHaveValue('option-1')

  await userEvent.selectOptions(select, [
    page.getByRole('option', { name: 'Option 1' }),
    page.getByRole('option', { name: 'Option 2' }),
  ])
  expect(select).toHaveValue(['option-1', 'option-2'])
})
```

::: warning
`webdriverio` provider 不支持选择多个元素，因为它不提供选择多个元素的 API。
:::

References:

- [Playwright `locator.selectOption` API](https://playwright.dev/docs/api/class-locator#locator-select-option)
- [WebdriverIO `element.selectByIndex` API](https://webdriver.io/docs/api/element/selectByIndex)
- [testing-library `selectOptions` API](https://testing-library.com/docs/user-event/utility/#-selectoptions-deselectoptions)

## userEvent.hover

```ts
function hover(
  element: Element | Locator,
  options?: UserEventHoverOptions,
): Promise<void>
```

该方法将光标位置移动到所选元素上。有关此方法如何工作的详细说明，请参阅 provider 的文档。

::: warning
如果使用的是 `webdriverio` provider，光标默认会移动到元素的中心。

如果使用的是 `playwright` provider，光标会移动到元素的某个可见点。
:::

```ts
import { page, userEvent } from '@vitest/browser/context'

test('hovers logo element', async () => {
  const logo = page.getByRole('img', { name: /logo/ })

  await userEvent.hover(logo)
  // or you can access it directly on the locator
  await logo.hover()
})
```

References:

- [Playwright `locator.hover` API](https://playwright.dev/docs/api/class-locator#locator-hover)
- [WebdriverIO `element.moveTo` API](https://webdriver.io/docs/api/element/moveTo/)
- [testing-library `hover` API](https://testing-library.com/docs/user-event/convenience/#hover)

## userEvent.unhover

```ts
function unhover(
  element: Element | Locator,
  options?: UserEventHoverOptions,
): Promise<void>
```

其作用与 [`userEvent.hover`](#userevent-hover) 相同，但会将光标移至 `document.body` 元素。

::: warning
默认情况下，光标位置位于 body 元素的 "某个" 可见位置（在 `playwright` provider中）或中心位置（在 `webdriverio` provider中），因此如果当前悬停的元素已经位于相同位置，本方法将不起作用。
:::

```ts
import { page, userEvent } from '@vitest/browser/context'

test('unhover logo element', async () => {
  const logo = page.getByRole('img', { name: /logo/ })

  await userEvent.unhover(logo)
  // or you can access it directly on the locator
  await logo.unhover()
})
```

References:

- [Playwright `locator.hover` API](https://playwright.dev/docs/api/class-locator#locator-hover)
- [WebdriverIO `element.moveTo` API](https://webdriver.io/docs/api/element/moveTo/)
- [testing-library `hover` API](https://testing-library.com/docs/user-event/convenience/#hover)

## userEvent.upload

```ts
function upload(
  element: Element | Locator,
  files: string[] | string | File[] | File,
): Promise<void>
```

更改文件输入元素，使其包含指定文件。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('can upload a file', async () => {
  const input = page.getByRole('button', { name: /Upload files/ })

  const file = new File(['file'], 'file.png', { type: 'image/png' })

  await userEvent.upload(input, file)
  // or you can access it directly on the locator
  await input.upload(file)

  // you can also use file paths relative to the test file
  await userEvent.upload(input, '../fixtures/file.png')
})
```

::: warning
`webdriverio` provider 仅在 `chrome` 和 `edge` 浏览器中支持该命令。目前也只支持字符串类型。
:::

References:

- [Playwright `locator.setInputFiles` API](https://playwright.dev/docs/api/class-locator#locator-set-input-files)
- [WebdriverIO `browser.uploadFile` API](https://webdriver.io/docs/api/browser/uploadFile)
- [testing-library `upload` API](https://testing-library.com/docs/user-event/utility/#upload)

## userEvent.dragAndDrop

```ts
function dragAndDrop(
  source: Element | Locator,
  target: Element | Locator,
  options?: UserEventDragAndDropOptions,
): Promise<void>
```

将源元素拖到目标元素的顶部。不要忘记，源元素的`draggable`属性必须设置为 `true`。

```ts
import { page, userEvent } from '@vitest/browser/context'

test('drag and drop works', async () => {
  const source = page.getByRole('img', { name: /logo/ })
  const target = page.getByTestId('logo-target')

  await userEvent.dragAndDrop(source, target)
  // or you can access it directly on the locator
  await source.dropTo(target)

  await expect.element(target).toHaveTextContent('Logo is processed')
})
```

::: warning
 `preview` provider不支持此 API。
:::

References:

- [Playwright `frame.dragAndDrop` API](https://playwright.dev/docs/api/class-frame#frame-drag-and-drop)
- [WebdriverIO `element.dragAndDrop` API](https://webdriver.io/docs/api/element/dragAndDrop/)
