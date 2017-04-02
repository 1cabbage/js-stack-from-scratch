# 06 - React Router, Server-Side Rendering, and Helmet

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/06-react-router-ssr-helmet).

本章我们将为 app 创建多个页面，并使多个页面之间可路由。

## React Router

> 💡 **[React Router](https://reacttraining.com/react-router/)** 用来在 React app 的页面间实现路由。这个包既可以运行在客户端，也能运行在服务端。



React Router 的 V4 版本更新很大，该版本还处在测试阶段。为了让本教程具有前瞻性，我们会使用这个版本。

- 运行 `yarn add react-router@next react-router-dom@next`

在客户端，要把 app 包裹在 `BrowserRouter` 组件中。

- 修改 `src/client/index.jsx` ：

```js
// [...]
import { BrowserRouter } from 'react-router-dom'
// [...]
const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <BrowserRouter>
      <AppContainer>
        <AppComponent />
      </AppContainer>
    </BrowserRouter>
  </Provider>
```

## Pages（页面）

我们的 app 会有四个页面

- Home —— 主页
- Hello 页面 —— 一个按钮加一段同步信息
- Hello 的异步页面 —— 一个按钮加一段异步信息
- 404 页面

- 创建 `src/client/component/page/home.jsx`：

```js
// @flow

import React from 'react'

const HomePage = () => <p>Home</p>

export default HomePage
```

- 创建 `src/client/component/page/hello.jsx`：

```js
// @flow

import React from 'react'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const HelloPage = () =>
  <div>
    <Message />
    <HelloButton />
  </div>

export default HelloPage

```

- 创建 `src/client/component/page/hello-async.jsx`：

```js
// @flow

import React from 'react'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const HelloAsyncPage = () =>
  <div>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage
```

- 创建 `src/client/component/page/not-found.jsx`：

```js
// @flow

import React from 'react'

const NotFoundPage = () => <p>Page not found</p>

export default NotFoundPage
```

## Navigation（导航/路由）

在前后端共享的配置文件中，加入路由配置

- 修改 `src/shared/routes.js`：

```js
// @flow

export const HOME_PAGE_ROUTE = '/'
export const HELLO_PAGE_ROUTE = '/hello'
export const HELLO_ASYNC_PAGE_ROUTE = '/hello-async'
export const NOT_FOUND_DEMO_PAGE_ROUTE = '/404'

export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

`/404` 页面本来应该是在访问不到链接的时候展示的；但为了例子简单，我们把 404 页面设置为一个固定的展示页面。

- 创建 `src/client/component/nav.jsx`：

```js
// @flow

import React from 'react'
import { NavLink } from 'react-router-dom'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../../shared/routes'

const Nav = () =>
  <nav>
    <ul>
      {[
        { route: HOME_PAGE_ROUTE, label: 'Home' },
        { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
        { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
        { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
      ].map(link => (
        <li key={link.route}>
          <NavLink to={link.route} activeStyle={{ color: 'limegreen' }} exact>{link.label}</NavLink>
        </li>
      ))}
    </ul>
  </nav>

export default Nav
```

根据前一个文件声明的路由，我们创建了一些 `NavLink` （导航链接）。

- 最后，修改 `src/client/app.jsx`：

```js
// @flow

import React from 'react'
import { Switch } from 'react-router'
import { Route } from 'react-router-dom'
import { APP_NAME } from '../shared/config'
import Nav from './component/nav'
import HomePage from './component/page/home'
import HelloPage from './component/page/hello'
import HelloAsyncPage from './component/page/hello-async'
import NotFoundPage from './component/page/not-found'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
} from '../shared/routes'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Nav />
    <Switch>
      <Route exact path={HOME_PAGE_ROUTE} render={() => <HomePage />} />
      <Route path={HELLO_PAGE_ROUTE} render={() => <HelloPage />} />
      <Route path={HELLO_ASYNC_PAGE_ROUTE} render={() => <HelloAsyncPage />} />
      <Route component={NotFoundPage} />
    </Switch>
  </div>

export default App
```

🏁 运行 `yarn start` 和 `yarn dev:wds`，浏览 `http://localhost:8000`，点击链接查看各个页面。URL 是动态更新的；使用浏览器的返回功能，看看浏览历史是否正确。

假设你现在再访问 `http://localhost:8000/hello` 页面，刷新一下页面，你会得到一个 404 错误。这是因为 Express 服务器只会相应 `/` 地址。当你在各个页面间跳转的时候，你只是用到了客户端路由。为了解决这个问题，我们要用到服务端渲染。

## Server-Side Rendering（服务端渲染）

> 💡 **服务端渲染** 意味着在页面初始化加载的时候，就已经被渲染好了（服务器返回的就是渲染好的页面），而不是依赖浏览器的渲染。

SSR 的有点是：有利于 SEO 和更好的用户体验。

首先，我们要把大部分客户端代码移动到 shared/isomorphic/universal，因为我们的 React app 还要在服务端渲染。

### 移动代码到 `shared`

- 除了 `src/client/index.jsx` 之外，`client` 文件夹下的所有文件都移动到 `shared`。

因为路径改变，我们的引用也要修改一下。

- 在 `src/client/index.jsx`中， 3 处 `'./app'` 都修改成 `'../shared/app'`， `'./reducer/hello'` 修改为 `'../shared/reducer/hello'`

- 在 `src/shared/app.jsx` 文件中，把 `'../shared/routes'` 修改为 `'./routes'`， `'../shared/config'` 修改为 `'./config'`

- 在 `src/shared/component/nav.jsx`， 把 `'../../shared/routes'` 修改为 `'../routes'`

### 服务端代码调整

- 创建 `src/server/routing.js` ：

```js
// @flow

import {
  homePage,
  helloPage,
  helloAsyncPage,
  helloEndpoint,
} from './controller'

import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  helloEndpointRoute,
} from '../shared/routes'

import renderApp from './render-app'

export default (app: Object) => {
  app.get(HOME_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, homePage()))
  })

  app.get(HELLO_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloPage()))
  })

  app.get(HELLO_ASYNC_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloAsyncPage()))
  })

  app.get(helloEndpointRoute(), (req, res) => {
    res.json(helloEndpoint(req.params.num))
  })

  app.get('/500', () => {
    throw Error('Fake Internal Server Error')
  })

  app.get('*', (req, res) => {
    res.status(404).send(renderApp(req.url))
  })

  // eslint-disable-next-line no-unused-vars
  app.use((err, req, res, next) => {
    // eslint-disable-next-line no-console
    console.error(err.stack)
    res.status(500).send('Something went wrong!')
  })
}
```

这个文件用来处理请求和相应；至于业务逻辑的处理，我们把放到 `controller` 模块中。

**注意**：你可能看到一些 React Route 示例中，用 `*` 作为服务端的路由 —— 这样做，所有的路由操作都被交给 React Router 处理。因为所有的请求都经过同样的方法，不利于开发 MVC 页面。我们的做法是，明确声明页面路由和返回值。这样做有些繁琐，但能从数据库获取数据，并且很简单地就能把值传给页面。

- 创建 `src/server/controller.js` ：

```js
// @flow

export const homePage = () => null

export const helloPage = () => ({
  hello: { message: 'Server-side preloaded message' },
})

export const helloAsyncPage = () => ({
  hello: { messageAsync: 'Server-side preloaded message for async page' },
})

export const helloEndpoint = (num: number) => ({
  serverMessage: `Hello from the server! (received ${num})`,
})
```

这就是我们的 controller。它只处理业务逻辑和数据库请求 —— 注意，为了简单，在我们的例子里，数据是写死的硬编码。这些数据被传回到 `routing` 模块，被用来初始化服务端的 Redux store。

- 创建 `src/server/init-store.js` ：

```js
// @flow

import Immutable from 'immutable'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'

import helloReducer from '../shared/reducer/hello'

const initStore = (plainPartialState: ?Object) => {
  const preloadedState = plainPartialState ? {} : undefined

  if (plainPartialState && plainPartialState.hello) {
    // flow-disable-next-line
    preloadedState.hello = helloReducer(undefined, {})
      .merge(Immutable.fromJS(plainPartialState.hello))
  }

  return createStore(combineReducers({ hello: helloReducer }),
    preloadedState, applyMiddleware(thunkMiddleware))
}

export default initStore
```

除了调用 `createStore` 和应用中间件外，我们做的唯一的事情就是把从 `controller` 接收到的 JS 对象转换为不可变对象，然后合并到 Redux state 中。

- 修改 `src/server/index.js` ：

```js
// @flow

import compression from 'compression'
import express from 'express'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

这段代码没什么特别的，我们调用 `routing(app)` 方法，而不是在这个文件中来实现路由。

- 重命名 `src/server/render-app.js` 为 `src/server/render-app.jsx` ，并修改内容：

```js
// @flow

import React from 'react'
import ReactDOMServer from 'react-dom/server'
import { Provider } from 'react-redux'
import { StaticRouter } from 'react-router'

import initStore from './init-store'
import App from './../shared/app'
import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <App />
      </StaticRouter>
    </Provider>)

  return (
    `<!doctype html>
    <html>
      <head>
        <title>FIX ME</title>
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
      <body>
        <div class="${APP_CONTAINER_CLASS}">${appHtml}</div>
        <script>
          window.__PRELOADED_STATE__ = ${JSON.stringify(store.getState())}
        </script>
        <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
      </body>
    </html>`
  )
}

export default renderApp
```

`ReactDOMServer.renderToString` 是核心方法。. React 会分析这个 `shared（前后端共享的）` `App`然后返回 HTML 元素的字符串。 `Provider` 和客户端的使用没什么区别，但在服务端，我们需要把 app 用 `StaticRouter` 包裹起来，而不是用 `BrowserRouter` 包裹。为了把 Redux store 从服务端传到客户端，我们把它传给 `window.__PRELOADED_STATE__` （变量名可以任意定义）。

**注意**: 不可变对象实现了 `toJSON()` 方法，因此你可以使用 `JSON.stringify` 来把他们转换为 JS 对象。

- 编辑 `src/client/index.jsx` ，使用预加载的 state：

```js
import Immutable from 'immutable'
// [...]

/* eslint-disable no-underscore-dangle */
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose
const preloadedState = window.__PRELOADED_STATE__
/* eslint-enable no-underscore-dangle */

const store = createStore(combineReducers(
  { hello: helloReducer }),
  { hello: Immutable.fromJS(preloadedState.hello) },
  composeEnhancers(applyMiddleware(thunkMiddleware)))
```

客户端的 store 被赋值为 `preloadedState`，这个值是服务端传过来的。

🏁 现在运行 `yarn start` 和 `yarn dev:wds`，在页面之间跳转。在 `/hello`， `/hello-async`， 和 `/404`（或者任意其他页面）刷新，应该没有之前的 404 问题了。显示的是 `message` 还是 `messageAsync`，取决于你是在客户端跳转到这个页面，还是直接从服务端拿到的这个页面。

### React Helmet

> 💡 **[React Helmet](https://github.com/nfl/react-helmet)**: 把 `head` 内容注入到 React app，可运行于客户端和服务端。

我建议你在标题写上 `FIX ME`，从而突出一个事实：虽然我们做了服务端渲染，但我们没有正确的把 `title` 标签添加进来（其他 `head` 内的标签页不对，因为它们应该是随着页面改变而改变的）。

- 运行 `yarn add react-helmet`

- 编辑 `src/server/render-app.jsx` ：

```js
import Helmet from 'react-helmet'
// [...]
const renderApp = (/* [...] */) => {

  const appHtml = ReactDOMServer.renderToString(/* [...] */)
  const head = Helmet.rewind()

  return (
    `<!doctype html>
    <html>
      <head>
        ${head.title}
        ${head.meta}
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
    [...]
    `
  )
}
```

React Helmet 使用 [react-side-effect](https://github.com/gaearon/react-side-effect)的 `rewind`，从 app 的渲染结果中拉取数据，这些数据会被 `<Helmet />` 组件使用。 我们在 `<Helmet />` 组件中为每一个页面设置 `title` 和其他 `head` 标签的值。注意 `Helmet.rewind()` *必须* 写在 `ReactDOMServer.renderToString()` 后面。

- 修改 `src/shared/app.jsx` ：

```js
import Helmet from 'react-helmet'
// [...]
const App = () =>
  <div>
    <Helmet titleTemplate={`%s | ${APP_NAME}`} defaultTitle={APP_NAME} />
    <Nav />
    // [...]
```

- 修改 `src/shared/component/page/home.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <h1>{APP_NAME}</h1>
  </div>

export default HomePage

```

- 修改 `src/shared/component/page/hello.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const title = 'Hello Page'

const HelloPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <Message />
    <HelloButton />
  </div>

export default HelloPage
```

- 修改 `src/shared/component/page/hello-async.jsx` ：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage

```

- 修改 `src/shared/component/page/not-found.jsx` ：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

const title = 'Page Not Found'

const NotFoundPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
  </div>

export default NotFoundPage
```

事实上， `<Helmet>` 组件没有渲染任何东西，它只是向 `head` 标签中插入内容，并且向服务端暴露了相同的内容。

🏁 运行 `yarn start` 和 `yarn dev:wds`，在页面之间做跳转。当你跳转页面的时候，title 应该已经变了；并且当你刷新页面的时候，title 不会更改。查看页面的源文件，研究一下 React Helmet 是怎样为服务端渲染设置 `title` 和 `meta` 值的。

下一章: [07 - Socket.IO](07-socket-io.md#readme)

回到 [上一章](05-redux-immutable-fetch.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
