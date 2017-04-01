# 05 - Redux, Immutable, and Fetch

**注意**： 本章中的 `state`，`action`，`reducer` 等术语都不会被翻译~

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/05-redux-immutable-fetch).

本章我们会结合使用 React 和 Redux，做一个简单的示例 app。这个示例由一个信息按钮组成，当用户点击的时候，信息会改变。

开始之前，我先简单地介绍一下 ImmutableJS —— 它跟 React 和 Redux 完全没关系，但我们会在本章中用到它。

## ImmutableJS

> 💡 **[ImmutableJS](https://facebook.github.io/immutable-js/)** (简称 Immutable) 是 Facebook 提供的一个操作不可变集合（例如 Map 或 List）的库。更改不可变对象时，不会更改原对象，而是会返回一个新对象。

举例，我们不建议这样：

```js
const obj = { a: 1 }
obj.a = 2 // Mutates `obj`
```

我们建议这样：

```js
const obj = Immutable.Map({ a: 1 })
obj.set('a', 2) // 返回一个新对象，没有更改 `obj`
```

这个库和 **函数式编程** 的思想不谋而合，并让 Redux 的使用如虎添翼。

创建不可变集合时，一个常用的方法是 `Immutable.fromJS()`。这个方法以 JS 对象或数组为参数，并返回一个深拷贝的不可变对象。

```js
const immutablePerson = Immutable.fromJS({
  name: 'Stan',
  friends: ['Kyle', 'Cartman', 'Kenny'],
})

console.log(immutablePerson)

/*
 *  Map {
 *    "name": "Stan",
 *    "friends": List [ "Kyle", "Cartman", "Kenny" ]
 *  }
 */
```

- 运行 `yarn add immutable@4.0.0-rc.2`

## Redux

> 💡 **[Redux](http://redux.js.org/)** 库用来管理应用的生命周期。它创建一个 *store*，作为应用中 state 的唯一源。

从最简单的部分开始，先声明 Redux actions：

- Run `yarn add redux redux-actions`

- 创建 `src/client/action/hello.js`：

```js
// @flow

import { createAction } from 'redux-actions'

export const SAY_HELLO = 'SAY_HELLO'

export const sayHello = createAction(SAY_HELLO)
```

该文件导出了一个 *action* —— `SAY_HELLO`，以及对应的 *action creator* —— `sayHello`，creator 是一个方法。我们用 [`redux-actions`](https://github.com/acdlite/redux-actions) 来处理 Redux actions。 `redux-actions` 实现了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 模型 —— *action creators* 返回的对象包含 `type` 和 `payload` 两个属性。

- 创建 `src/client/reducer/hello.js` ：

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import { SAY_HELLO } from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    default:
      return state
  }
}

export default helloReducer
```

上面的代码用 Immutable Map 初始化了 reducer 的 state，该 state 包含一个属性 `message`，值为 `Initial reducer message`。`helloReducer` 处理 `SAY_HELLO` actions 的方式很简单 —— 只是把 `message` 的值设置为 payload 的值。Flow 注释把 `action` 参数解构为 `type` 和 `payload`；其中，`payload` 的类型为 `any`。为了给 `state` 提供类型注释，我们用 Flow 语法 `import type` 来得到 `fromJS` 它的类型。为了保持代码清晰和可读性，我们把这个类型重命名为 `Immut`，因为要是把注释写成 `state: fromJS`，让人看着头大。`import type` 和其他的 Flow 注释一样，不会影响代码运行。注意看一下 `Immutable.fromJS()` 和 `set()` 是怎么用的；在之前那个简单的例子里，我们已经用过一次了。


## React-Redux

> 💡 **[react-redux](https://github.com/reactjs/react-redux)** 把 Redux store 和 React 组件的使用 *结合* 了起来。有了 `react-redux`，当 Redux store 改变的时候，React 组件就会自动更新。

- 运行 `yarn add react-redux`

下一节我们会创建 *Components* 和 *Containers*。

**Components（组件）** 是有点 *傻乎乎* 的 React 组件，某种程度上来说，它们感知不到 Redux state 的更新。 **Containers** 是相对 *聪明* 的组件，它们能感知状态变化。

- 创建 `src/client/component/button.jsx`：

```js
// @flow

import React from 'react'

type Props = {
  label: string,
  handleClick: Function,
}

const Button = ({ label, handleClick }: Props) =>
  <button onClick={handleClick}>{label}</button>

export default Button
```

**注意**: 我们在这里使用了 Flow 的 *类型别名*。我们自定义了 `Props` 类型，来解构组件的 `props`。

- 创建 `src/client/component/message.jsx`：

```js
// @flow

import React from 'react'

type Props = {
  message: string,
}

const Message = ({ message }: Props) =>
  <p>{message}</p>

export default Message
```

以上的例子属于 *傻乎乎* 的组件。这些组件缺乏逻辑，只会展示通过 *props（属性）* 传进来的值。`button.jsx` 和 `message.jsx` 的区别是， `Button` 组件的属性里包含了一个 action dispatcher，而 `Message` 组件只是用来展示数据。


再强调一下，*components* 不能感知 Redux 的 **actions** 或者 app 的 **state**；所以，我们要创建 **containers**， 从而向这俩组件中传入 action dispatchers 和数据。

Again, *components* don't know anything about Redux **actions** or the **state** of our app, which is why we are going to create smart **containers** that will feed the proper action dispatchers and data to these 2 dumb components.

- 创建 `src/client/container/hello-button.js` ：

```js
// @flow

import { connect } from 'react-redux'

import { sayHello } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHello('Hello!')) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

这个 container 用 `sayHello` action 和 Redux 的 `dispatch` 方法，挂载了 `Button` 组件。

- 创建 `src/client/container/message.js` ：

```js
// @flow

import { connect } from 'react-redux'

import Message from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('message'),
})

export default connect(mapStateToProps)(Message)
```

这个 container 把 Redux 的应用状态和 `Message` 组件相挂载。当装填改变， `Message` 会根据 `message` 属性自动重新渲染。组件和属性之间的联系，是通过 `react-redux` 提供的 `connect` 方法。

- 修改 `src/client/app.jsx`：

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import Message from './container/message'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
  </div>

export default App
```

我们还没有初始化 Redux store，也还没有在 app 中应用以上两个 containers：

- 修改 `src/client/index.jsx`：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers } from 'redux'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

const store = createStore(combineReducers({ hello: helloReducer }),
  // eslint-disable-next-line no-underscore-dangle
  isProd ? undefined : window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__())

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

花点时间 review 一下我们的代码。首先，用 `createStore` 方法, 以 reducers 为参数，创建了一个 *store*。我们现在只有一个 reducer，但为了未来代码的扩展性，我们用 `combineReducers` 方法把 reducers 组成了一个集合。该最后一个参数，是用来把 Redux 绑定到浏览器的 [开发工具](https://github.com/zalmoxisus/redux-devtools-extension) —— debug 的时候很有用。因为 `__REDUX_DEVTOOLS_EXTENSION__` 的下划线，ESLint 会报错，所以在这一行我们禁用了下划线规则。利用我们之前写的 `wrapApp` 方法，可以非常容易的把 app 包裹在 `Provider` 组件中，并向其中传入 store。

🏁 现在可以用运行 `yarn start` 和 `yarn dev:wds`，然后打开 `http://localhost:8000`。页面内容是 "Initial reducer message" 和一个按钮。点击按钮，信息会变成 "Hello!"。如果你的浏览器安装了 Redux 开发者插件，就能更清楚的看到 app 的状态变化了。

恭喜！我们的 app 现在看起来终于有点像那么回事了。虽然表面上看这个 app 没什么厉害的地方，但我们知道，在底层，它是由一个相当牛逼的技术栈作支撑的。

## 用异步请求来拓展 app

接下来，我们要添加一个新按钮；点击这个按钮，会发出一个 AJAX 请求。仅作示例，这个请求会发送一个数据，然后服务器会返回硬编码的 `1234`。

### The server endpoint

- Create a `src/shared/routes.js` file containing:

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

这个方法是个帮助类，这样用：

```js
helloEndpointRoute()     // -> '/ajax/hello/:num' (for Express)
helloEndpointRoute(1234) // -> '/ajax/hello/1234' (for the actual call)
```

赶紧先测试下

- 创建 `src/shared/routes.test.js`：

```js
import { helloEndpointRoute } from './routes'

test('helloEndpointRoute', () => {
  expect(helloEndpointRoute()).toBe('/ajax/hello/:num')
  expect(helloEndpointRoute(123)).toBe('/ajax/hello/123')
})
```

- 运行 `yarn test`

- 在 `src/server/index.js`，添加：

```js
import { helloEndpointRoute } from '../shared/routes'

// [under app.get('/')...]

app.get(helloEndpointRoute(), (req, res) => {
  res.json({ serverMessage: `Hello from the server! (received ${req.params.num})` })
})
```

### 创建新的 containers

- 创建 `src/client/container/hello-async-button.js` ：

```js
// @flow

import { connect } from 'react-redux'

import { sayHelloAsync } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello asynchronously and send 1234',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHelloAsync(1234)) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

这个例子只是为了说明怎样向异步请求传参数，为了简单，我传的值是硬编码的 `1234`；一般来说，这个值应该是来自用户输入。

- 创建 `src/client/container/message-async.js`：

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

You can see that in this container, we are referring to a `messageAsync` property, which we're going to add to our reducer soon.

What we need now is to create the `sayHelloAsync` action.

### Fetch

> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** is a standardized JavaScript function to make asynchronous calls inspired by jQuery's AJAX methods.

We are going to use `fetch` to make calls to the server from the client. `fetch` is not supported by all browsers yet, so we are going to need a polyfill. `isomorphic-fetch` is a polyfill that makes it work cross-browsers and in Node too!

- Run `yarn add isomorphic-fetch`

Since we're using `eslint-plugin-compat`, we need to indicate that we are using a polyfill for `fetch` to not get warnings from using it.

- Add the following to your `.eslintrc.json` file:

```json
"settings": {
  "polyfills": ["fetch"]
},
```

### 3 asynchronous actions

`sayHelloAsync` is not going to be a regular action. Asynchronous actions are usually split into 3 actions, which trigger 3 different states: a *request* action (or "loading"), a *success* action, and a *failure* action.

- Edit `src/client/action/hello.js` like so:

```js
// @flow

import 'isomorphic-fetch'

import { createAction } from 'redux-actions'
import { helloEndpointRoute } from '../../shared/routes'

export const SAY_HELLO = 'SAY_HELLO'
export const SAY_HELLO_ASYNC_REQUEST = 'SAY_HELLO_ASYNC_REQUEST'
export const SAY_HELLO_ASYNC_SUCCESS = 'SAY_HELLO_ASYNC_SUCCESS'
export const SAY_HELLO_ASYNC_FAILURE = 'SAY_HELLO_ASYNC_FAILURE'

export const sayHello = createAction(SAY_HELLO)
export const sayHelloAsyncRequest = createAction(SAY_HELLO_ASYNC_REQUEST)
export const sayHelloAsyncSuccess = createAction(SAY_HELLO_ASYNC_SUCCESS)
export const sayHelloAsyncFailure = createAction(SAY_HELLO_ASYNC_FAILURE)

export const sayHelloAsync = (num: number) => (dispatch: Function) => {
  dispatch(sayHelloAsyncRequest())
  return fetch(helloEndpointRoute(num), { method: 'GET' })
    .then((res) => {
      if (!res.ok) throw Error(res.statusText)
      return res.json()
    })
    .then((data) => {
      if (!data.serverMessage) throw Error('No message received')
      dispatch(sayHelloAsyncSuccess(data.serverMessage))
    })
    .catch(() => {
      dispatch(sayHelloAsyncFailure())
    })
}
```

Instead of returning an action, `sayHelloAsync` returns a function which launches the `fetch` call. `fetch` returns a `Promise`, which we use to *dispatch* different actions depending on the current state of our asynchronous call.

### 3 asynchronous action handlers

Let's handle these different actions in `src/client/reducer/hello.js`:

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import {
  SAY_HELLO,
  SAY_HELLO_ASYNC_REQUEST,
  SAY_HELLO_ASYNC_SUCCESS,
  SAY_HELLO_ASYNC_FAILURE,
} from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
  messageAsync: 'Initial reducer message for async call',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    case SAY_HELLO_ASYNC_REQUEST:
      return state.set('messageAsync', 'Loading...')
    case SAY_HELLO_ASYNC_SUCCESS:
      return state.set('messageAsync', action.payload)
    case SAY_HELLO_ASYNC_FAILURE:
      return state.set('messageAsync', 'No message received, please check your connection')
    default:
      return state
  }
}

export default helloReducer
```

We added a new field to our store, `messageAsync`, and we update it with different messages depending on the action we receive. During `SAY_HELLO_ASYNC_REQUEST`, we show `Loading...`. `SAY_HELLO_ASYNC_SUCCESS` updates `messageAsync` similarly to how `SAY_HELLO` updates `message`. `SAY_HELLO_ASYNC_FAILURE` gives an error message.

### Redux-thunk

In `src/client/action/hello.js`, we made `sayHelloAsync`, an action creator that returns a function. This is actually not a feature that is natively supported by Redux. In order to perform these async actions, we need to extend Redux's functionality with the `redux-thunk` *middleware*.

- Run `yarn add redux-thunk`

- Update your `src/client/index.jsx` file like so:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

// eslint-disable-next-line no-underscore-dangle
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose

const store = createStore(combineReducers({ hello: helloReducer }),
  composeEnhancers(applyMiddleware(thunkMiddleware)))

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

Here we pass `redux-thunk` to Redux's `applyMiddleware` function. In order for the Redux Devtools to keep working, we also need to use Redux's `compose` function. Don't worry too much about this part, just remember that we enhance Redux with `redux-thunk`.

- Update `src/client/app.jsx` like so:

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import HelloAsyncButton from './container/hello-async-button'
import Message from './container/message'
import MessageAsync from './container/message-async'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default App
```

🏁 Run `yarn start` and `yarn dev:wds` and you should now be able to click the "Say hello asynchronously and send 1234" button and retrieve a corresponding message from the server! Since you're working locally, the call is instantaneous, but if you open the Redux Devtools, you will notice that each click triggers both `SAY_HELLO_ASYNC_REQUEST` and `SAY_HELLO_ASYNC_SUCCESS`, making the message go through the intermediate `Loading...` state as expected.

You can congratulate yourself, that was an intense section! Let's wrap it up with some testing.

## Testing

In this section, we are going to test our actions and reducer. Let's start with the actions.

In order to isolate the logic that is specific to `action/hello.js` we are going to need to *mock* things that don't concern it, and also mock that AJAX `fetch` request which should not trigger an actual AJAX in our tests.

- Run `yarn add --dev redux-mock-store fetch-mock`

- Create a `src/client/action/hello.test.js` file containing:

```js
import fetchMock from 'fetch-mock'
import configureMockStore from 'redux-mock-store'
import thunkMiddleware from 'redux-thunk'

import {
  sayHelloAsync,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from './hello'

import { helloEndpointRoute } from '../../shared/routes'

const mockStore = configureMockStore([thunkMiddleware])

afterEach(() => {
  fetchMock.restore()
})

test('sayHelloAsync success', () => {
  fetchMock.get(helloEndpointRoute(666), { serverMessage: 'Async hello success' })
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncSuccess('Async hello success'),
      ])
    })
})

test('sayHelloAsync 404', () => {
  fetchMock.get(helloEndpointRoute(666), 404)
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})

test('sayHelloAsync data error', () => {
  fetchMock.get(helloEndpointRoute(666), {})
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})
```

Alright, Let's look at what's happening here. First we mock the Redux store using `const mockStore = configureMockStore([thunkMiddleware])`. By doing this we can dispatch actions without them triggering any reducer logic. For each test, we mock `fetch` using `fetchMock.get()` and make it return whatever we want. What we actually test using `expect()` is which series of actions have been dispatched by the store, thanks to the `store.getActions()` function from `redux-mock-store`. After each test we restore the normal behavior of `fetch` with `fetchMock.restore()`.

Let's now test our reducer, which is much easier.

- Create a `src/client/reducer/hello.test.js` file containing:

```js
import {
  sayHello,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from '../action/hello'

import helloReducer from './hello'

let helloState

beforeEach(() => {
  helloState = helloReducer(undefined, {})
})

test('handle default', () => {
  expect(helloState.get('message')).toBe('Initial reducer message')
  expect(helloState.get('messageAsync')).toBe('Initial reducer message for async call')
})

test('handle SAY_HELLO', () => {
  helloState = helloReducer(helloState, sayHello('Test'))
  expect(helloState.get('message')).toBe('Test')
})

test('handle SAY_HELLO_ASYNC_REQUEST', () => {
  helloState = helloReducer(helloState, sayHelloAsyncRequest())
  expect(helloState.get('messageAsync')).toBe('Loading...')
})

test('handle SAY_HELLO_ASYNC_SUCCESS', () => {
  helloState = helloReducer(helloState, sayHelloAsyncSuccess('Test async'))
  expect(helloState.get('messageAsync')).toBe('Test async')
})

test('handle SAY_HELLO_ASYNC_FAILURE', () => {
  helloState = helloReducer(helloState, sayHelloAsyncFailure())
  expect(helloState.get('messageAsync')).toBe('No message received, please check your connection')
})
```

Before each test, we initialize `helloState` with the default result of our reducer (the `default` case of our `switch` statement in the reducer, which returns `initialState`). The tests are then very explicit, we just make sure the reducer updates `message` and `messageAsync` correctly depending on which action it received.

🏁 Run `yarn test`. It should be all green.

Next section: [06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

Back to the [previous section](04-webpack-react-hmr.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
