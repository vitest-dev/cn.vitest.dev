---
title: Examples | Browser Mode
---

# Examples

浏览器模式与框架无关，因此不提供任何渲染组件的方法。不过，你应该可以使用框架的测试工具包。

我们建议根据您的框架使用  `testing-library` packages：

- [`@testing-library/dom`](https://testing-library.com/docs/dom-testing-library/intro) if you don't use a framework
- [`@testing-library/vue`](https://testing-library.com/docs/vue-testing-library/intro) to render [vue](https://vuejs.org) components
- [`@testing-library/svelte`](https://testing-library.com/docs/svelte-testing-library/intro) to render [svelte](https://svelte.dev) components
- [`@testing-library/react`](https://testing-library.com/docs/react-testing-library/intro) to render [react](https://react.dev) components
- [`@testing-library/preact`](https://testing-library.com/docs/preact-testing-library/intro) to render [preact](https://preactjs.com) components
- [`@solidjs/testing-library`](https://testing-library.com/docs/solid-testing-library/intro) to render [solid](https://www.solidjs.com) components
- [`@marko/testing-library`](https://testing-library.com/docs/marko-testing-library/intro) to render [marko](https://markojs.com) components

::: warning
`testing-library` 提供了一个包`@testing-library/user-event`。我们不建议直接使用它，因为它会模拟事件而非实际触发事件--相反，请使用从 `@vitest/browser/context`导入的 [`userEvent`](#interactivity-api)，它使用 Chrome DevTools 协议或 Webdriver（取决于provider）。
:::

::: code-group
```ts [vue]
// based on @testing-library/vue example
// https://testing-library.com/docs/vue-testing-library/examples

import { render, screen } from '@testing-library/vue'
import { userEvent } from '@vitest/browser/context'
import Component from './Component.vue'

test('properly handles v-model', async () => {
  render(Component)

  // Asserts initial state.
  expect(screen.getByText('Hi, my name is Alice')).toBeInTheDocument()

  // Get the input DOM node by querying the associated label.
  const usernameInput = await screen.findByLabelText(/username/i)

  // Type the name into the input. This already validates that the input
  // is filled correctly, no need to check the value manually.
  await userEvent.fill(usernameInput, 'Bob')

  expect(screen.getByText('Hi, my name is Alice')).toBeInTheDocument()
})
```
```ts [svelte]
// based on @testing-library/svelte
// https://testing-library.com/docs/svelte-testing-library/example

import { render, screen } from '@testing-library/svelte'
import { userEvent } from '@vitest/browser/context'
import { expect, test } from 'vitest'

import Greeter from './greeter.svelte'

test('greeting appears on click', async () => {
  const user = userEvent.setup()
  render(Greeter, { name: 'World' })

  const button = screen.getByRole('button')
  await user.click(button)
  const greeting = await screen.findByText(/hello world/iu)

  expect(greeting).toBeInTheDocument()
})
```
```tsx [react]
// based on @testing-library/react example
// https://testing-library.com/docs/react-testing-library/example-intro

import { render, screen } from '@testing-library/react'
import { userEvent } from '@vitest/browser/context'
import Fetch from './fetch'

test('loads and displays greeting', async () => {
  // Render a React element into the DOM
  render(<Fetch url="/greeting" />)

  await userEvent.click(screen.getByText('Load Greeting'))
  // wait before throwing an error if it cannot find an element
  const heading = await screen.findByRole('heading')

  // assert that the alert message is correct
  expect(heading).toHaveTextContent('hello there')
  expect(screen.getByRole('button')).toBeDisabled()
})
```
```tsx [preact]
// based on @testing-library/preact example
// https://testing-library.com/docs/preact-testing-library/example

import { render } from '@testing-library/preact'
import { userEvent } from '@vitest/browser/context'
import { h } from 'preact'

import HiddenMessage from '../hidden-message'

test('shows the children when the checkbox is checked', async () => {
  const testMessage = 'Test Message'

  const { queryByText, getByLabelText, getByText } = render(
    <HiddenMessage>{testMessage}</HiddenMessage>,
  )

  // query* functions will return the element or null if it cannot be found.
  // get* functions will return the element or throw an error if it cannot be found.
  expect(queryByText(testMessage)).not.toBeInTheDocument()

  // The queries can accept a regex to make your selectors more
  // resilient to content tweaks and changes.
  await userEvent.click(getByLabelText(/show/i))

  expect(getByText(testMessage)).toBeInTheDocument()
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
  const { findByText } = render(() => <App />, { location: 'ids/1234' })
  expect(await findByText('Id: 1234')).toBeInTheDocument()
})
```
```ts [marko]
// baed on @testing-library/marko API
// https://testing-library.com/docs/marko-testing-library/api

import { render, screen } from '@marko/testing-library'
import Greeting from './greeting.marko'

test('renders a message', async () => {
  const { container } = await render(Greeting, { name: 'Marko' })
  expect(screen.getByText(/Marko/)).toBeInTheDocument()
  expect(container.firstChild).toMatchInlineSnapshot(`
    <h1>Hello, Marko!</h1>
  `)
})
```
:::
