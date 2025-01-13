---
title: Locators | Browser Mode
outline: [2, 3]
---

# 定位 <Version>2.1.0</Version>

定位器是元素或多个元素的表示。每个定位器由一个称为选择器的字符串定义。Vitest 通过提供方便的方法在后台生成这些选择器，从而抽象了选择器。

<<<<<<< HEAD
定位器 API 使用了 [Playwright 定位器](https://playwright.dev/docs/api/class-locator)的一个分支，称为 [Ivya](https://npmjs.com/ivya)。然而，Vitest 将此 API 提供给每一个 [provider](/guide/browser/#provider-configuration)。
=======
The locator API uses a fork of [Playwright's locators](https://playwright.dev/docs/api/class-locator) called [Ivya](https://npmjs.com/ivya). However, Vitest provides this API to every [provider](/guide/browser/config.html#browser-provider).
>>>>>>> 9ec1afbefb2413ce2ea01ddd41c89a571670d913

## getByRole

```ts
function getByRole(
  role: ARIARole | string,
  options?: LocatorByRoleOptions,
): Locator
```

通过元素的 [ARIA 角色](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)、[ARIA 属性](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes) 和 [可访问名称](https://developer.mozilla.org/en-US/docs/Glossary/Accessible_name) 创建一种定位元素的方式。

::: tip
如果你只查询单个元素（例如使用 `getByText('The name')`），通常更好的做法是使用 `getByRole(expectedRole, { name: 'The name' })`。可访问名称查询并不会替代其他查询，例如 `*ByAltText` 或 `*ByTitle`。虽然可访问名称可以等于这些属性的值，但它并不能替代这些属性的功能。
:::

考虑以下 DOM 结构。

```html
<h3>Sign up</h3>
<label>
  Login
  <input type="text" />
</label>
<label>
  Password
  <input type="password" />
</label>
<br/>
<button>Submit</button>
```

你可以通过每个元素的隐式角色来定位它们：

```ts
await expect.element(
  page.getByRole('heading', { name: 'Sign up' })
).toBeVisible()

await page.getByRole('textbox', { name: 'Login' }).fill('admin')
await page.getByRole('textbox', { name: 'Password' }).fill('admin')

await page.getByRole('button', { name: /submit/i }).click()
```

::: warning
角色通过字符串相等性进行匹配，不会继承自 ARIA 角色层次结构。因此，查询像 `checkbox` 这样的超类角色时，不会包含具有子类角色（如 `switch`）的元素。

默认情况下，许多语义化的 HTML 元素都有一个角色；例如，`<input type="radio">` 具有 "radio" 角色。非语义化的 HTML 元素没有角色；没有添加语义的 `<div>` 和 `<span>` 返回 `null`。`role` 属性可以提供语义。

根据 ARIA 指南，**强烈不建议** 通过 `role` 或 `aria-*` 属性为已经具有隐式角色的内置元素提供角色。
:::

##### Options

- `exact: boolean`

  `name` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `name` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

  ```tsx
  <button>Hello World</button>

  page.getByRole('button', { name: 'hello world' }) // ✅
  page.getByRole('button', { name: 'hello world', exact: true }) // ❌
  page.getByRole('button', { name: 'Hello World', exact: true }) // ✅
  ```

- `checked: boolean`

  是否应包含已选中的元素（由 `aria-checked` 或 `<input type="checkbox"/>` 设置）。默认情况下，不会应用此过滤器。

  更多信息请参阅 [`aria-checked`](https://www.w3.org/TR/wai-aria-1.2/#aria-checked)。

  ```tsx
  <>
    <button role="checkbox" aria-checked="true" />
    <input type="checkbox" checked />
  </>

  page.getByRole('checkbox', { checked: true }) // ✅
  page.getByRole('checkbox', { checked: false }) // ❌
  ```

- `disabled: boolean`

  是否应包含已禁用的元素。默认情况下，不会应用此过滤器。请注意，与其他属性不同，`disable` 状态是可继承的。

  更多信息请参阅 [`aria-disabled`](https://www.w3.org/TR/wai-aria-1.2/#aria-disabled)。

  ```tsx
  <input type="text" disabled />

  page.getByRole('textbox', { disabled: true }) // ✅
  page.getByRole('textbox', { disabled: false }) // ❌
  ```

- `expanded: boolean`

  是否应包含展开的元素。默认情况下，不会应用此过滤器。

  更多信息请参阅 [`aria-expanded`](https://www.w3.org/TR/wai-aria-1.2/#aria-expanded)。

  ```tsx
  <a aria-expanded="true" href="example.com">Link</a>

  page.getByRole('link', { expanded: true }) // ✅
  page.getByRole('link', { expanded: false }) // ❌
  ```

- `includeHidden: boolean`

  是否应查询那些通常从可访问性树中排除的元素。默认情况下，只有非隐藏元素会匹配角色选择器。更多信息请参阅 [可访问性树中的排除规则](https://www.w3.org/TR/wai-aria-1.2/#tree_exclusion)。

  请注意，角色 `none` 和 `presentation` 始终会被包含。

  ```tsx
  <button style="display: none" />

  page.getByRole('button') // ❌
  page.getByRole('button', { includeHidden: false }) // ❌
  page.getByRole('button', { includeHidden: true }) // ✅
  ```

- `level: number`

  一个数字属性，通常存在于 `heading`、`listitem`、`row`、`treeitem` 角色中，并且对于 `<h1>-<h6>` 元素有默认值。默认情况下，不会应用此过滤器。

  更多信息请参阅 [`aria-level`](https://www.w3.org/TR/wai-aria-1.2/#aria-level)。

  ```tsx
  <>
    <h1>Heading Level One</h1>
    <div role="heading" aria-level="1">Second Heading Level One</div>
  </>

  page.getByRole('heading', { level: 1 }) // ✅
  page.getByRole('heading', { level: 2 }) // ❌
  ```

- `name: string | RegExp`

  [可访问名称](https://developer.mozilla.org/en-US/docs/Glossary/Accessible_name)。默认情况下，匹配是不区分大小写的，并且会搜索子字符串。使用 `exact` 选项来控制此行为。

  ```tsx
  <button>Click Me!</button>

  page.getByRole('button', { name: 'Click Me!' }) // ✅
  page.getByRole('button', { name: 'click me!' }) // ✅
  page.getByRole('button', { name: 'Click Me?' }) // ❌
  ```

- `pressed: boolean`

  是否应包含被按下的元素。默认情况下，不会应用此过滤器。

  更多信息请参阅 [`aria-pressed`](https://www.w3.org/TR/wai-aria-1.2/#aria-pressed)。

  ```tsx
  <button aria-pressed="true">👍</button>

  page.getByRole('button', { pressed: true }) // ✅
  page.getByRole('button', { pressed: false }) // ❌
  ```

- `selected: boolean`

  是否应包含被选中的元素。默认情况下，不会应用此过滤器。

  更多信息请参阅 [`aria-selected`](https://www.w3.org/TR/wai-aria-1.2/#aria-selected)。

  ```tsx
  <button role="tab" aria-selected="true">Vue</button>

  page.getByRole('button', { selected: true }) // ✅
  page.getByRole('button', { selected: false }) // ❌
  ```

##### See also

- [MDN 上的 ARIA 角色列表](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)
- [w3.org 上的 ARIA 角色列表](https://www.w3.org/TR/wai-aria-1.2/#role_definitions)
- [testing-library's `ByRole`](https://testing-library.com/docs/queries/byrole/)

## getByAltText

```ts
function getByAltText(
  text: string | RegExp,
  options?: LocatorOptions,
): Locator
```

创建一个能够找到具有匹配 `alt` 属性的元素的定位器。与 testing-library 的实现不同，Vitest 会匹配任何具有匹配 `alt` 属性的元素。

```tsx
<img alt="Incredibles 2 Poster" src="/incredibles-2.png" />

page.getByAltText(/incredibles.*? poster/i) // ✅
page.getByAltText('non existing alt text') // ❌
```

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByAltText`](https://testing-library.com/docs/queries/byalttext/)

## getByLabelText

```ts
function getByLabelText(
  text: string | RegExp,
  options?: LocatorOptions,
): Locator
```

创建一个能够找到具有关联标签的元素的定位器。

`page.getByLabelText('Username')` 定位器将在以下示例中找到所有的输入元素：

```html
// for/htmlFor relationship between label and form element id
<label for="username-input">Username</label>
<input id="username-input" />

// The aria-labelledby attribute with form elements
<label id="username-label">Username</label>
<input aria-labelledby="username-label" />

// Wrapper labels
<label>Username <input /></label>

// Wrapper labels where the label text is in another child element
<label>
  <span>Username</span>
  <input />
</label>

// aria-label attributes
// Take care because this is not a label that users can see on the page,
// so the purpose of your input must be obvious to visual users.
<input aria-label="Username" />
```

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByLabelText`](https://testing-library.com/docs/queries/bylabeltext/)

## getByPlaceholder

```ts
function getByPlaceholder(
  text: string | RegExp,
  options?: LocatorOptions,
): Locator
```

创建一个能够找到具有指定 `placeholder` 属性的元素的定位器。Vitest 会匹配任何具有匹配 `placeholder` 属性的元素，而不仅仅是 `input` 元素。

```tsx
<input placeholder="Username" />

page.getByPlaceholder('Username') // ✅
page.getByPlaceholder('not found') // ❌
```

::: warning
通常情况下，使用 [`getByLabelText`](#getbylabeltext) 依赖标签比依赖占位符更好。
:::

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByPlaceholderText`](https://testing-library.com/docs/queries/byplaceholdertext/)

## getByText

```ts
function getByText(
  text: string | RegExp,
  options?: LocatorOptions,
): Locator
```

创建一个能够找到包含指定文本的元素的定位器。文本将与 TextNode 的 [`nodeValue`](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeValue) 或类型为 `button` 或 `reset` 的输入值进行匹配。通过文本匹配时总是会规范化空白字符，即使在精确匹配的情况下也是如此。例如，它会将多个空格转换为一个空格，将换行符转换为空格，并忽略前导和尾随的空白字符。

```tsx
<a href="/about">About ℹ️</a>

page.getByText(/about/i) // ✅
page.getByText('about', { exact: true }) // ❌
```

::: tip
此定位器适用于定位非交互式元素。如果你需要定位交互式元素，比如按钮或输入框，建议使用 [`getByRole`](#getbyrole)。
:::

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByText`](https://testing-library.com/docs/queries/bytext/)

## getByTitle

```ts
function getByTitle(
  text: string | RegExp,
  options?: LocatorOptions,
): Locator
```

创建一个能够找到具有指定 `title` 属性的元素的定位器。与 testing-library 的 `getByTitle` 不同，Vitest 无法在 SVG 中找到 `title` 元素。

```tsx
<span title="Delete" id="2"></span>

page.getByTitle('Delete') // ✅
page.getByTitle('Create') // ❌
```

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByTitle`](https://testing-library.com/docs/queries/bytitle/)

## getByTestId

```ts
function getByTestId(text: string | RegExp): Locator
```

创建一个能够找到与指定测试 ID 属性匹配的元素的定位器。你可以通过 [`browser.locators.testIdAttribute`](/guide/browser/config#browser-locators-testidattribute) 配置属性名称。

```tsx
<div data-testid="custom-element" />

page.getByTestId('custom-element') // ✅
page.getByTestId('non-existing-element') // ❌
```

::: warning
建议仅在其他定位器不适用于你的使用场景时才使用此方法。使用 `data-testid` 属性并不符合用户实际使用软件的方式，因此如果可能应避免使用。
:::

#### Options

- `exact: boolean`

  `text` 是否精确匹配：区分大小写且完全匹配字符串。默认情况下禁用此选项。如果 `text` 是正则表达式，则忽略此选项。请注意，精确匹配仍然会修剪空白字符。

#### See also

- [testing-library's `ByTestId`](https://testing-library.com/docs/queries/bytestid/)

## nth

```ts
function nth(index: number): Locator
```

此方法返回一个新的定位器，仅匹配多元素查询结果中的特定索引。与 `elements()[n]` 不同，`nth` 定位器会重试，直到元素出现为止。

```html
<div aria-label="one"><input/><input/><input/></div>
<div aria-label="two"><input/></div>
```

```tsx
page.getByRole('textbox').nth(0) // ✅
page.getByRole('textbox').nth(4) // ❌
```

::: tip
在使用 `nth` 之前，你可能会发现使用链式定位器来缩小搜索范围会更有帮助。
有时没有比通过元素位置更好的区分方式；虽然这可能导致测试不稳定，但总比没有强。
:::

```tsx
page.getByLabel('two').getByRole('input') // ✅ better alternative to page.getByRole('textbox').nth(3)
page.getByLabel('one').getByRole('input') // ❌ too ambiguous
page.getByLabel('one').getByRole('input').nth(1) // ✅ pragmatic compromise
```

## first

```ts
function first(): Locator
```

此方法返回一个新的定位器，仅匹配多元素查询结果中的第一个索引。
它是 `nth(0)` 的语法糖。

```html
<input/> <input/> <input/>
```

```tsx
page.getByRole('textbox').first() // ✅
```

## last

```ts
function last(): Locator
```

此方法返回一个新的定位器，仅匹配多元素查询结果中的最后一个索引。
它是 `nth(-1)` 的语法糖。

```html
<input/> <input/> <input/>
```

```tsx
page.getByRole('textbox').last() // ✅
```

## Methods

所有方法都是异步的，必须使用 `await`。自 Vitest 3 起，如果方法没有被 `await`，测试将会失败。

### click

```ts
function click(options?: UserEventClickOptions): Promise<void>
```

点击一个元素。你可以使用选项来设置光标位置。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('img', { name: 'Rose' }).click()
```

- [更多内容请参阅 `userEvent.click`](/guide/browser/interactivity-api#userevent-click)

### dblClick

```ts
function dblClick(options?: UserEventDoubleClickOptions): Promise<void>
```

在元素上触发双击事件。你可以使用选项来设置光标位置。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('img', { name: 'Rose' }).dblClick()
```

- [更多内容请参阅 `userEvent.dblClick`](/guide/browser/interactivity-api#userevent-dblclick)

### tripleClick

```ts
function tripleClick(options?: UserEventTripleClickOptions): Promise<void>
```

在元素上触发三连击事件。由于浏览器 API 中没有 `tripleclick`，此方法会连续触发三次点击事件。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('img', { name: 'Rose' }).tripleClick()
```

- [更多内容请参阅 `userEvent.tripleClick`](/guide/browser/interactivity-api#userevent-tripleclick)

### clear

```ts
function clear(): Promise<void>
```

清除输入元素的内容。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('textbox', { name: 'Full Name' }).clear()
```

- [更多内容请参阅 `userEvent.clear`](/guide/browser/interactivity-api#userevent-clear)

### hover

```ts
function hover(options?: UserEventHoverOptions): Promise<void>
```

将光标位置移动到选中的元素。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('img', { name: 'Rose' }).hover()
```

- [更多内容请参阅 `userEvent.hover`](/guide/browser/interactivity-api#userevent-hover)

### unhover

```ts
function unhover(options?: UserEventHoverOptions): Promise<void>
```

这与 [`locator.hover`](#hover) 的工作方式相同，但将光标移动到 `document.body` 元素。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('img', { name: 'Rose' }).unhover()
```

- [更多内容请参阅 `userEvent.unhover`](/guide/browser/interactivity-api#userevent-unhover)

### fill

```ts
function fill(text: string, options?: UserEventFillOptions): Promise<void>
```

设置当前 `input`、`textarea` 或 `contenteditable` 元素的值。

```ts
import { page } from '@vitest/browser/context'

await page.getByRole('input', { name: 'Full Name' }).fill('Mr. Bean')
```

- [更多内容请参阅 `userEvent.fill`](/guide/browser/interactivity-api#userevent-fill)

### dropTo

```ts
function dropTo(
  target: Locator,
  options?: UserEventDragAndDropOptions,
): Promise<void>
```

将当前元素拖动到目标位置。

```ts
import { page } from '@vitest/browser/context'

const paris = page.getByText('Paris')
const france = page.getByText('France')

await paris.dropTo(france)
```

- [更多内容请参阅 `userEvent.dragAndDrop`](/guide/browser/interactivity-api#userevent-draganddrop)

### selectOptions

```ts
function selectOptions(
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

从 `<select>` 元素中选择一个或多个值。

```ts
import { page } from '@vitest/browser/context'

const languages = page.getByRole('select', { name: 'Languages' })

await languages.selectOptions('EN')
await languages.selectOptions(['ES', 'FR'])
await languages.selectOptions([
  languages.getByRole('option', { name: 'Spanish' }),
  languages.getByRole('option', { name: 'French' }),
])
```

- [更多内容请参阅 `userEvent.selectOptions`](/guide/browser/interactivity-api#userevent-selectoptions)

### screenshot

```ts
function screenshot(options: LocatorScreenshotOptions & { base64: true }): Promise<{
  path: string
  base64: string
}>
function screenshot(options?: LocatorScreenshotOptions & { base64?: false }): Promise<string>
```

创建与定位器选择器匹配的元素的屏幕截图。

你可以使用 `path` 选项指定屏幕截图的保存位置，该选项相对于当前测试文件。如果未设置 `path` 选项，Vitest 将默认使用 [`browser.screenshotDirectory`](/guide/browser/config#browser-screenshotdirectory)（默认为 `__screenshot__`），并结合文件名和测试名来确定屏幕截图的文件路径。

如果你还需要屏幕截图的内容，可以指定 `base64: true` 以返回屏幕截图的 base64 编码内容以及保存路径。

```ts
import { page } from '@vitest/browser/context'

const button = page.getByRole('button', { name: 'Click Me!' })

const path = await button.screenshot()

const { path, base64 } = await button.screenshot({
  path: './button-click-me.png',
  base64: true, // also return base64 string
})
// path - fullpath to the screenshot
// bas64 - base64 encoded string of the screenshot
```

### query

```ts
function query(): Element | null
```

此方法返回与定位器选择器匹配的单个元素，如果没有找到元素则返回 `null`。

如果多个元素匹配该选择器，此方法将抛出错误。如果你需要所有匹配的 DOM 元素，可以使用 [`.elements()`](#elements)；如果你需要匹配选择器的定位器数组，可以使用 [`.all()`](#all)。

考虑以下 DOM 结构：

```html
<div>Hello <span>World</span></div>
<div>Hello</div>
```

这些定位器将不会抛出错误：

```ts
page.getByText('Hello World').query() // ✅ HTMLDivElement
page.getByText('Hello Germany').query() // ✅ null
page.getByText('World').query() // ✅ HTMLSpanElement
page.getByText('Hello', { exact: true }).query() // ✅ HTMLSpanElement
```

这些定位器将抛出错误：

```ts
// returns multiple elements
page.getByText('Hello').query() // ❌
page.getByText(/^Hello/).query() // ❌
```

### element

```ts
function element(): Element
```

此方法返回与定位器选择器匹配的单个元素。

如果 _没有元素_ 匹配该选择器，则会抛出错误。如果你只需要检查元素是否存在，可以考虑使用 [`.query()`](#query)。

如果 _多个元素_ 匹配该选择器，则会抛出错误。如果你需要所有匹配的 DOM 元素，可以使用 [`.elements()`](#elements)；如果你需要匹配选择器的定位器数组，可以使用 [`.all()`](#all)。

::: tip
此方法在需要将其传递给外部库时非常有用。当定位器与 `expect.element` 一起使用时，每次断言 [重试](/guide/browser/assertion-api) 时都会自动调用此方法：

```ts
await expect.element(page.getByRole('button')).toBeDisabled()
```
:::

考虑以下 DOM 结构：

```html
<div>Hello <span>World</span></div>
<div>Hello Germany</div>
<div>Hello</div>
```

这些定位器将不会抛出错误：

```ts
page.getByText('Hello World').element() // ✅
page.getByText('Hello Germany').element() // ✅
page.getByText('World').element() // ✅
page.getByText('Hello', { exact: true }).element() // ✅
```

这些定位器将抛出错误：

```ts
// returns multiple elements
page.getByText('Hello').element() // ❌
page.getByText(/^Hello/).element() // ❌

// returns no elements
page.getByText('Hello USA').element() // ❌
```

### elements

```ts
function elements(): Element[]
```

此方法返回与定位器选择器匹配的元素数组。

此函数不会抛出错误。如果没有元素匹配该选择器，此方法将返回一个空数组。

考虑以下 DOM 结构：

```html
<div>Hello <span>World</span></div>
<div>Hello</div>
```

这些定位器将始终成功：

```ts
page.getByText('Hello World').elements() // ✅ [HTMLElement]
page.getByText('World').elements() // ✅ [HTMLElement]
page.getByText('Hello', { exact: true }).elements() // ✅ [HTMLElement]
page.getByText('Hello').element() // ✅ [HTMLElement, HTMLElement]
page.getByText('Hello USA').elements() // ✅ []
```

### all

```ts
function all(): Locator[]
```

此方法返回一个与选择器匹配的新定位器数组。

在内部，此方法调用 `.elements` 并使用 [`page.elementLocator`](/guide/browser/context#page) 包装每个元素。

- [See `locator.elements()`](#elements)

## Properties

### selector

`selector` 是一个字符串，将由浏览器提供程序用于定位元素。Playwright 将使用 `playwright` 定位器语法，而 `preview` 和 `webdriverio` 将使用 CSS。

::: danger
你不应在测试代码中使用此字符串。`selector` 字符串仅应在使用 Commands API 时使用：

```ts [commands.ts]
import type { BrowserCommand } from 'vitest/node'

const test: BrowserCommand<string> = function test(context, selector) {
  // playwright
  await context.iframe.locator(selector).click()
  // webdriverio
  await context.browser.$(selector).click()
}
```

```ts [example.test.ts]
import { test } from 'vitest'
import { commands, page } from '@vitest/browser/context'

test('works correctly', async () => {
  await commands.test(page.getByText('Hello').selector) // ✅
  // vitest will automatically unwrap it to a string
  await commands.test(page.getByText('Hello')) // ✅
})
```
:::
