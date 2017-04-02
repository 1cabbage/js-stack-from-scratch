# 05 - Redux, Immutable, and Fetch

**注意**： 本章中的 `state`，`action`，`reducer` 等术语都不会翻译~如果你之前没有用过 React 系列工具，学习这一章肯定有些吃力，甚至看不懂；但正如作者一开始所说，这个教程只是教你怎么构建一个技术栈，不是教你怎么写代码。如果需要学习 React，还是要去看文档才行~

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

上面的代码用 Immutable Map 初始化了 reducer 的 state，该 state 包含一个属性 `message`，值为 `Initial reducer message`。`helloReducer` 处理 `SAY_HELLO` actions 的方式很简单 —— 只是把 `message` 的值设置为 payload 的值。Flow 注释把 `action` 参数解构为 `type` 和 `payload`；其中，`payload` 的类型为 `any`。为了给 `state` 提供类型注释，我们用 Flow 语法 `import type` 来得到 `fromJS` 的类型。为了保持代码清晰和可读性，我们把这个类型重命名为 `Immut`，因为要是把注释写成 `state: fromJS`，让人看着头大。`import type` 和其他的 Flow 注释一样，不会影响代码运行。注意看一下 `Immutable.fromJS()` 和 `set()` 是怎么用的；在之前那个简单的例子里，我们已经用过一次了。


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

**注意**: 在这里，我们使用了 Flow 的 *类型别名*。我们自定义了 `Props` 类型，来解构组件的 `props`。

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

花点时间 review 一下我们的代码。首先，用 `createStore` 方法, 以 reducers 为参数，创建了一个 *store*。我们现在只有一个 reducer，但为了未来代码的扩展性，我们用 `combineReducers` 方法把 reducers 组成了一个集合。最后一个参数，是用来把 Redux 绑定到浏览器的 [开发工具](https://github.com/zalmoxisus/redux-devtools-extension) —— debug 的时候很有用。因为 `__REDUX_DEVTOOLS_EXTENSION__` 的下划线，ESLint 会报错，所以在这一行我们禁用了下划线规则。利用我们之前写的 `wrapApp` 方法，可以非常容易的把 app 包裹在 `Provider` 组件中，并向其传入 store。

🏁 现在可以用运行 `yarn start` 和 `yarn dev:wds`，然后打开 `http://localhost:8000`。页面内容是 "Initial reducer message" 和一个按钮。点击按钮，信息会变成 "Hello!"。如果你的浏览器安装了 Redux 开发者插件，就能更清楚的看到 app 的状态变化了。

恭喜！我们的 app 现在看起来终于有点像那么回事了。虽然表面上看这个 app 没什么厉害的地方，但我们知道，在底层，它是由一个相当牛逼的技术栈作支撑的。

## 用异步请求来拓展 app

接下来，我们要添加一个新按钮；点击这个按钮，会发出一个 AJAX 请求。仅作示例，这个请求会发送一个数据，然后服务器会返回硬编码的 `1234`。

### The server endpoint

- 创建 `src/shared/routes.js` 文件：

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

这个例子只是为了说明怎样向异步请求传参数，为了简单，我传的值是硬编码的 `1234` ；一般来说，这个值应该来自用户输入。

- 创建 `src/client/container/message-async.js` ：

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

在这个 container 中，我们引用了一个 `messageAsync` 属性，我们要在 reducer 中加入这个属性。

现在我们需要创建 `sayHelloAsync` action。

### Fetch

> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** 是发起异步请求的标准方法，受了 jQuery AJAX 方法的启发。

我们在向服务器发起请求的时候会用到 `fetch`，这个方法目前还没有被所有浏览器支持，所以需要 polyfill。 `isomorphic-fetch` 不单单能跨浏览器使用，甚至还能在 Node 环境使用。

- 运行 `yarn add isomorphic-fetch`

因为使用了 `eslint-plugin-compat` 插件，为了在使用 `fetch` 时关闭不必要的警告，配置文件需作出修改：

- 修改 `.eslintrc.json`：

```json
"settings": {
  "polyfills": ["fetch"]
},
```

### 3个异步 actions

`sayHelloAsync` 不是一个常规的 action。异步 actions 通常分为三部分，并触发三种状态：一个 *request（请求）* action，一个 *success（成功）* action，一个 *failure（失败）* action。（译者注：如果你用过 Promise，应该感到熟悉~）

- 编辑 `src/client/action/hello.js`：

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

`sayHelloAsync` 没返回一个 action，而是返回了一个发起 `fetch` 请求的方法。`fetch` 返回一个  `Promise`，根据异步请求的状态，这个请求会 *dispatch（分发）* 不同的 action。

### 3 异步 action 处理器

在 `src/client/reducer/hello.js` 文件中，处理不同的 actions：

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

在 store 中，我们添加了一个新值： `messageAsync`，收到的 action 不同，值也不同。 如果 action.type 是 `SAY_HELLO_ASYNC_REQUEST`，值设为 `Loading...`。 `SAY_HELLO_ASYNC_SUCCESS` 和 `SAY_HELLO` 处理 `message` 的方式一样。 如果是 `SAY_HELLO_ASYNC_FAILURE`，则给出错误信息。

### Redux-thunk

在 `src/client/action/hello.js`，我们创建了 `sayHelloAsync`，他是一个 action creator，返回一个方法。Redux 原生并不支持这样使用。为了使用异步 actions，我们用 `redux-thunk` *中间件* 来扩展 Redux 功能。

- 安装 `yarn add redux-thunk`

- 修改 `src/client/index.jsx`：

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

我们把 `redux-thunk` 传给 Redux `applyMiddleware` 方法。为了使用 Redux 开发工具，我们还需要使用 Redux 的 `compose` 方法。别太在意这一部分，只要知道我们用 `redux-thunk` 增强了 Redux 就行了。

- 修改 `src/client/app.jsx`：

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

🏁 运行 `yarn start`和 `yarn dev:wds`， 点击 "Say hello asynchronously and send 1234" 按钮，就能收到服务器返回的信息！因为你的服务器部署在本地，所以请求都是瞬间返回。要是你打开 Redux 开发工具，你就会发现每一次点击都触发了 `SAY_HELLO_ASYNC_REQUEST` and `SAY_HELLO_ASYNC_SUCCESS`；也就是说，你在浏览器里可能看不到（因为太快了），但 `Loading...` 状态确实存在过。

现在可以放松一下，因为这一章的确有点难。现在做一些测试。

## 测试

这一部分，我们会测试 actions 和 reducer。从测试 actions 开始：

为了分离 `action/hello.js` 的代码逻辑，我们需要 *mock（模拟）* 一些东西；`fetch` 操作也需要模拟，测试中我们并不用真的发起 AJAX 操作。

- 运行 `yarn add --dev redux-mock-store fetch-mock`

- 创建 `src/client/action/hello.test.js`：

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

上面的代码首先用 `const mockStore = configureMockStore([thunkMiddleware])` 这行代码模拟 Redux store。这样，当我们 dispatch（分发）actions 时，就不会触发 reducer 的逻辑了。每一个测试，我们都用 `fetchMock.get()` 来模拟 `fetch`，并且返回值是我们自定义的。我们真正测试的东西，是 store 分发的一系列 actions，这里，我们用到了 `redux-mock-store` 提供的 'store.getActions()' 方法。在每一次测试之后，我们用 `fetchMock.restore()` 方法来把 'fetch' 恢复到初始状态。

reducer 的测试相对简单：

- 创建 `src/client/reducer/hello.test.js`：

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

每次测试之前，我们都初始化了 `helloState` 的值，把它设为 reducer 的默认返回值（ `switch` 的默认返回值是 `initialState` ）。

🏁 运行 `yarn test`，所有测试通过。

下一章： [06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

回到 [上一章](04-webpack-react-hmr.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
