---
title: Browser Mode | Guide
outline: deep
---

# 浏览器模式 <Badge type="warning">实验性</Badge> {#browser-mode}

此页面提供有关 Vitest API 中实验性浏览器模式功能的信息，该功能允许你在浏览器中本地运行测试，提供对窗口和文档等浏览器全局变量的访问。此功能目前正在开发中，API 未来可能会更改。

<img alt="Vitest UI" img-light src="/ui-browser-1-light.png">
<img alt="Vitest UI" img-dark src="/ui-browser-1-dark.png">

## 安装

为方便设置，可使用 `vitest init browser` 命令安装所需的依赖项并创建浏览器配置。

::: code-group

```bash [npm]
npx vitest init browser
```

```bash [yarn]
yarn exec vitest init browser
```

```bash [pnpm]
pnpx vitest init browser
```

```bash [bun]
bunx vitest init browser
```

:::

### 手动安装

您也可以手动安装软件包。默认情况下，浏览器模式不需要任何额外的 E2E provider 就能在本地运行测试，因为它会复用你现有的浏览器。

::: code-group

```bash [npm]
npm install -D vitest @vitest/browser
```

```bash [yarn]
yarn add -D vitest @vitest/browser
```

```bash [pnpm]
pnpm add -D vitest @vitest/browser
```

```bash [bun]
bun add -D vitest @vitest/browser
```

:::

::: warning
不过，要在 CI 中运行测试，您需要安装 [`playwright`](https://npmjs.com/package/playwright) 或 [`webdriverio`](https://www.npmjs.com/package/webdriverio) 。我们还建议在本地测试时切换到这两个选项中的一个，而不是使用默认的 `preview` 提供程序，因为它依赖于模拟事件而不是使用 Chrome DevTools 协议。

如果你还没有使用这些工具中的一种，我们建议您从 Playwright 开始，因为它支持并行执行，这将使您的测试运行得更快。此外，Playwright 使用的 Chrome DevTools 协议通常比 WebDriver 更快。
:::

::: tabs key:provider
== Playwright
[Playwright](https://npmjs.com/package/playwright) 是一个用于网络测试和自动化的框架。

::: code-group

```bash [npm]
npm install -D vitest @vitest/browser playwright
```

```bash [yarn]
yarn add -D vitest @vitest/browser playwright
```

```bash [pnpm]
pnpm add -D vitest @vitest/browser playwright
```

```bash [bun]
bun add -D vitest @vitest/browser playwright
```

== WebdriverIO

[WebdriverIO](https://www.npmjs.com/package/webdriverio) 允许您使用 WebDriver 协议在本地运行测试。

::: code-group

```bash [npm]
npm install -D vitest @vitest/browser webdriverio
```

```bash [yarn]
yarn add -D vitest @vitest/browser webdriverio
```

```bash [pnpm]
pnpm add -D vitest @vitest/browser webdriverio
```

```bash [bun]
bun add -D vitest @vitest/browser webdriverio
```

:::

## 配置

要在 Vitest 配置中激活浏览器模式，可以使用 `--browser` 标志，或在 Vitest 配置文件中将 `browser.enabled` 字段设为 `true`。下面是一个使用浏览器字段的配置示例：

```ts
export default defineConfig({
  test: {
    browser: {
      provider: 'playwright', // or 'webdriverio'
      enabled: true,
      name: 'chromium', // browser name is required
    },
  },
})
```

::: info
Vitest 分配的端口为 `63315`，以避免与开发服务器冲突，从而允许同时运行两个服务器。你可以使用 [`browser.api`](/config/#browser-api) 选项来更改。

从 Vitest 2.1.5 开始，CLI 不再自动打印 Vite URL。在监视模式下运行时，你可以按 “b” 打印 URL。
:::

如果之前未使用过 Vite，请确保已安装框架插件并在配置中指定。有些框架可能需要额外配置才能运行，请查看其 Vite 相关文档以确定。

::: code-group

```ts [vue]
import vue from '@vitejs/plugin-vue'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  plugins: [vue()],
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },
  },
})
```

```ts [svelte]
import { svelte } from '@sveltejs/vite-plugin-svelte'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  plugins: [svelte()],
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },
  },
})
```

```ts [solid]
import solidPlugin from 'vite-plugin-solid'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  plugins: [solidPlugin()],
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },
  },
})
```

```ts [marko]
import marko from '@marko/vite'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  plugins: [marko()],
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      name: 'chromium',
    },
  },
})
```

:::

::: tip
`react` 不需要插件就能工作，但 `preact` 需要 [extra configuration](https://preactjs.com/guide/v10/getting-started/#create-a-vite-powered-preact-app) 才能使用别名
:::

如果需要使用基于 Node 的运行程序运行某些测试，可以定义一个 [workspace](/guide/workspace) 文件，为不同的测试策略分别配置：

{#workspace-config}

```ts
// vitest.workspace.ts
import { defineWorkspace } from 'vitest/config'

export default defineWorkspace([
  {
    test: {
      // an example of file based convention,
      // you don't have to follow it
      include: [
        'tests/unit/**/*.{test,spec}.ts',
        'tests/**/*.unit.{test,spec}.ts',
      ],
      name: 'unit',
      environment: 'node',
    },
  },
  {
    test: {
      // an example of file based convention,
      // you don't have to follow it
      include: [
        'tests/browser/**/*.{test,spec}.ts',
        'tests/**/*.browser.{test,spec}.ts',
      ],
      name: 'browser',
      browser: {
        enabled: true,
        name: 'chrome',
      },
    },
  },
])
```

### Provider 配置

:::tabs key:provider
== Playwright
你可以通过 [`providerOptions`](/config/#browser-provideroptions)字段配置 Vitest 如何 [启动浏览器](https://playwright.dev/docs/api/class-browsertype#browser-type-launch) 和创建 [页面上下文](https://playwright.dev/docs/api/class-browsercontext)：

```ts
export default defineConfig({
  test: {
    browser: {
      providerOptions: {
        launch: {
          devtools: true,
        },
        context: {
          geolocation: {
            latitude: 45,
            longitude: -30,
          },
          reducedMotion: 'reduce',
        },
      },
    },
  },
})
```

要获得类型提示，请在 `tsconfig.json` 文件的 `compilerOptions.types` 中添加 `@vitest/browser/providers/playwright`。
== WebdriverIO

你可以通过 [`providerOptions`](/config/#browser-provideroptions)字段配置 Vitest 在启动浏览器时应使用哪些 [options](https://webdriver.io/docs/configuration#webdriverio)：

```ts
export default defineConfig({
  test: {
    browser: {
      browser: 'chrome',
      providerOptions: {
        region: 'eu',
        capabilities: {
          browserVersion: '27.0',
          platformName: 'Windows 10',
        },
      },
    },
  },
})
```

要获得类型提示，请在 `tsconfig.json` 文件的 `compilerOptions.types` 中添加 `@vitest/browser/providers/webdriverio`。
:::

## 浏览器选项类型

Vitest 中的浏览器选项取决于 provider。如果在配置文件中传递 `--browser` 且未指定其名称，则 Vitest 将失败。可用选项：

- `webdriverio` 支持这些浏览器:
  - `firefox`
  - `chrome`
  - `edge`
  - `safari`
- `playwright` 支持这些浏览器:
  - `firefox`
  - `webkit`
  - `chromium`

## 浏览器兼容性

Vitest 使用 [Vite dev server](https://cn.vitejs.dev/guide/#browser-support) 来运行您的测试，因此我们只支持 [`esbuild.target`](https://cn.vitejs.dev/config/shared-options#esbuild)选项（默认为 `esnext`）中指定的功能。

默认情况下，Vite 的目标浏览器支持本地 [ES Modules](https://caniuse.com/es6-module)、本地 [ESM dynamic import](https://caniuse.com/es6-module-dynamic-import) 和 [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta)。此外，我们还利用 [`BroadcastChannel`](https://caniuse.com/?search=BroadcastChannel)在 iframe 之间进行通信：

- Chrome >=87
- Firefox >=78
- Safari >=15.4
- Edge >=88

## Running Tests

要使用 CLI 指定浏览器，请使用 `--browser` 标志后跟浏览器名称，如下所示：

```sh
npx vitest --browser=chrome
```

或者你可以使用点符号向 CLI 提供浏览器选项：

```sh
npx vitest --browser.name=chrome --browser.headless
```

默认情况下，Vitest 会自动打开浏览器用户界面进行开发。您的测试将在中间的 iframe 中运行。您可以通过选择首选尺寸、在测试中调用 `page.viewport` 或在 [the config](/config/#browser-viewport) 中设置默认值来配置视口。

## Headless

headless 模式是浏览器模式下可用的另一个选项。在 headless 模式下，浏览器在没有用户界面的情况下在后台运行，这对于运行自动化测试非常有用。Vitest 中的 headless 选项可以设置为布尔值以启用或禁用 headless 模式。

使用 headless 模式时，Vitest 不会自动打开用户界面。如果想继续使用用户界面，但又想 headless 运行测试，可以安装 [`@vitest/ui`](/guide/ui) 包，并在运行 Vitest 时传递 --ui 标志。

这是启用 headless 模式的示例配置：

```ts
export default defineConfig({
  test: {
    browser: {
      provider: 'playwright',
      enabled: true,
      headless: true,
    },
  },
})
```

你还可以在 CLI 中使用 `--browser.headless` 标志设置 headless 模式，如下所示：

```sh
npx vitest --browser.name=chrome --browser.headless
```

在这种情况下，Vitest 将使用 Chrome 浏览器以 headless 模式运行。

::: warning
默认情况下 Headless 模式不可用。您需要使用 [`playwright`](https://npmjs.com/package/playwright) 或 [`webdriverio`](https://www.npmjs.com/package/webdriverio) 提供程序来启用此功能。
:::

## Examples

Vitest 提供的软件包可为多个流行框架呈现开箱即用的组件：

- [`vitest-browser-vue`](https://github.com/vitest-dev/vitest-browser-vue) 渲染 [vue](https://vuejs.org) 组件
- [`vitest-browser-svelte`](https://github.com/vitest-dev/vitest-browser-svelte) 渲染 [svelte](https://svelte.dev) 组件
- [`vitest-browser-react`](https://github.com/vitest-dev/vitest-browser-react) 渲染 [react](https://react.dev) 组件

如果您的框架没有包含此功能，请随意创建自己的软件包--它是框架渲染器和 `page.elementLocator` API 的简单封装。我们将在本页添加指向它的链接。请确保它的名称以 `vitest-browser-` 开头。

除了使用 `@testing-library/your-framework` 渲染组件和查询元素外，你还需要进行断言。Vitest 捆绑了 [`@testing-library/jest-dom`](https://github.com/testing-library/jest-dom)库，可提供各种开箱即用的 DOM 断言。更多信息请参阅 [Assertions API](/guide/browser/assertion-api)。

```ts
import { page } from '@vitest/browser/context'
import { expect } from 'vitest'
// element is rendered correctly
await expect.element(page.getByText('Hello World')).toBeInTheDocument()
```

Vitest 公开了一个[Context API](/guide/browser/context)，其中包含一小套在测试中可能有用的实用程序。例如，如果您需要进行交互，如点击元素或在输入框中输入文本，您可以使用 `@vitest/browser/context` 中的 `userEvent`。更多信息请参阅 [Interactivity API](/guide/browser/interactivity-api)。

```ts
import { page, userEvent } from '@vitest/browser/context'
await userEvent.fill(page.getByLabelText(/username/i), 'Alice')
// or just locator.fill
await page.getByLabelText(/username/i).fill('Alice')
```

::: code-group

```ts [vue]
import { render } from 'vitest-browser-vue'
import Component from './Component.vue'

test('properly handles v-model', async () => {
  const screen = render(Component)

  // Asserts initial state.
  await expect
    .element(screen.getByText('Hi, my name is Alice'))
    .toBeInTheDocument()

  // Get the input DOM node by querying the associated label.
  const usernameInput = screen.getByLabelText(/username/i)

  // Type the name into the input. This already validates that the input
  // is filled correctly, no need to check the value manually.
  await usernameInput.fill('Bob')

  await expect
    .element(screen.getByText('Hi, my name is Bob'))
    .toBeInTheDocument()
})
```

```ts [svelte]
import { expect, test } from 'vitest'
import { render } from 'vitest-browser-svelte'

import Greeter from './greeter.svelte'

test('greeting appears on click', async () => {
  const screen = render(Greeter, { name: 'World' })

  const button = screen.getByRole('button')
  await button.click()
  const greeting = screen.getByText(/hello world/iu)

  await expect.element(greeting).toBeInTheDocument()
})
```

```tsx [react]
import { render } from 'vitest-browser-react'
import Fetch from './fetch'

test('loads and displays greeting', async () => {
  // Render a React element into the DOM
  const screen = render(<Fetch url="/greeting" />)

  await screen.getByText('Load Greeting').click()
  // wait before throwing an error if it cannot find an element
  const heading = screen.getByRole('heading')

  // assert that the alert message is correct
  await expect.element(heading).toHaveTextContent('hello there')
  await expect.element(screen.getByRole('button')).toBeDisabled()
})
```

:::

Vitest 并不支持所有开箱即用的框架，但您可以使用外部工具来运行这些框架的测试。我们还鼓励社区创建他们自己的 `vitest-browser` 封装程序，如果您有这样的封装程序，请随时将其添加到上述示例中。

对于不支持的框架，我们建议使用 `testing-library` 软件包：

- [`@testing-library/preact`](https://testing-library.com/docs/preact-testing-library/intro) 渲染 [preact](https://preactjs.com) 组件
- [`@solidjs/testing-library`](https://testing-library.com/docs/solid-testing-library/intro) 渲染 [solid](https://www.solidjs.com) 组件
- [`@marko/testing-library`](https://testing-library.com/docs/marko-testing-library/intro) 渲染 [marko](https://markojs.com) 组件

::: warning
`testing-library` 提供了一个软件包 `@testing-library/user-event`。我们不建议直接使用它，因为它会模拟事件而非实际触发事件--相反，请使用从 `@vitest/browser/context`导入的 [`userEvent`](/guide/browser/interactivity-api)，它在引擎盖下使用 Chrome DevTools 协议或 Webdriver（取决于 provider）。
:::

::: code-group

```tsx [preact]
// based on @testing-library/preact example
// https://testing-library.com/docs/preact-testing-library/example

import { render } from '@testing-library/preact'
import { page } from '@vitest/browser/context'
import { h } from 'preact'

import HiddenMessage from '../hidden-message'

test('shows the children when the checkbox is checked', async () => {
  const testMessage = 'Test Message'

  const { baseElement } = render(<HiddenMessage>{testMessage}</HiddenMessage>)

  const screen = page.elementLocator(baseElement)

  // .query() will return the element or null if it cannot be found.
  // .element() will return the element or throw an error if it cannot be found.
  expect(screen.getByText(testMessage).query()).not.toBeInTheDocument()

  // The queries can accept a regex to make your selectors more
  // resilient to content tweaks and changes.
  await screen.getByLabelText(/show/i).click()

  await expect.element(screen.getByText(testMessage)).toBeInTheDocument()
})
```

```tsx [solid]
// baed on @testing-library/solid API
// https://testing-library.com/docs/solid-testing-library/api

import { render } from '@testing-library/solid'

it('uses params', async () => {
  const App = () => (
    <>
      <Route
        path="/ids/:id"
        component={() => (
          <p>
            Id:
            {useParams()?.id}
          </p>
        )}
      />
      <Route path="/" component={() => <p>Start</p>} />
    </>
  )
  const { baseElement } = render(() => <App />, { location: 'ids/1234' })
  const screen = page.elementLocator(baseElement)

  await expect.screen(screen.getByText('Id: 1234')).toBeInTheDocument()
})
```

```ts [marko]
// baed on @testing-library/marko API
// https://testing-library.com/docs/marko-testing-library/api

import { render, screen } from '@marko/testing-library'
import Greeting from './greeting.marko'

test('renders a message', async () => {
  const { baseElement } = await render(Greeting, { name: 'Marko' })
  const screen = page.elementLocator(baseElement)
  await expect.element(screen.getByText(/Marko/)).toBeInTheDocument()
  await expect.element(container.firstChild).toMatchInlineSnapshot(`
    <h1>Hello, Marko!</h1>
  `)
})
```

:::

## 限制

### 线程阻塞对话框

使用 Vitest 浏览器时，需要注意的是像 `alert` 或 `confirm` 这样的线程阻塞对话框不能在本地使用。这是因为它们阻塞了网页，这意味着 Vitest 无法继续与该页面通信，导致执行挂起。

在这种情况下，Vitest 为这些 API 提供默认模拟和默认返回值。这确保如果用户不小心使用了同步弹出式 Web API，执行不会挂起。但是，仍然建议用户模拟这些 Web API 以获得更好的体验。在 [Mocking](/guide/mocking) 中阅读更多内容。
