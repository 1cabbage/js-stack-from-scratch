# 08 - Bootstrap and JSS

本章代码在 [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate) 的分支 [`master-no-services`](https://github.com/verekia/js-stack-boilerplate/tree/master-no-services)

我们的 app 有点丑，让我们用推特的 Bootstrap 加点样式美化一下。我们会引入 CSS-in-JS 包来加入自定义的样式。

## Twitter Bootstrap

> 💡 **[Twitter Bootstrap](http://getbootstrap.com/)** 是一个 UI 组件库。

有两种方式把 Bootstrap 引入到你的 React app，两种引入方式都有支持者和反对者：

- 使用官方发布版本，**该版本使用了 jQuery 和 Tether**。
- 使用重新实现的第三方库 [React-Bootstrap](https://react-bootstrap.github.io/) 或者 [Reactstrap](https://reactstrap.github.io/).

和官方版本相比，第三方库的 React 组件用起来相当简单。虽然这么说，但我本人并不太想用第三方库。因为第三方版本总是在官方版本 *之后* 发布（有时候要隔很久才更新）。有时，第三方的库还不能和 Bootstrap 的主题相兼容，因为这些主题使用了自己的 JS。Bootstrap 的一大优点就是拥有一个庞大的设计师社区，如果这些设计者提供的主题不被第三方库支持，那实在是有点说不过去。

因为以上原因，我做出了妥协：选择官方版本，并结合 jQuery 和 Tether 使用。但这样的话，打包后的文件大小成了个问题 —— 打包后的文件大约 200KB （开启了 Gzipped 压缩）。我觉得这个大小还可以接受，但如果对你来说文件还是太大，那你可能需要找另一种方式来用 Bootstrap，或者干脆不选择 Bootstrap。

### Bootstrap's CSS

- 删除 `public/css/style.css`

- 运行 `yarn add bootstrap@4.0.0-alpha.6`

- 从 `node_modules/bootstrap/dist/css` 把 `bootstrap.min.css` 和 `bootstrap.min.css.map` 拷贝到 `public/css` 文件夹。

- 修改 `src/server/render-app.jsx`：

```html
<link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
```

### 结合了 jQuery 和 Tether 的 Bootstrap JS

Bootstrap 的样式已经会在页面上加载了；为了给组件添加行为，我们需要导入 JS。

- 运行 `yarn add jquery tether`

- 修改 `src/client/index.jsx`：

```js
import $ from 'jquery'
import Tether from 'tether'

// [right after all your imports]

window.jQuery = $
window.Tether = Tether
require('bootstrap')
```

这样 Bootstrap 的 JavaScript 代码会被加载进来。

### Bootstrap 组件

现在，你可以做一些复制粘贴的工作：

- 修改 `src/shared/component/page/hello-async.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import MessageAsync from '../../container/message-async'
import HelloAsyncButton from '../../container/hello-async-button'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <MessageAsync />
        <HelloAsyncButton />
      </div>
    </div>
  </div>

export default HelloAsyncPage
```

- 修改 `src/shared/component/page/hello.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import Message from '../../container/message'
import HelloButton from '../../container/hello-button'

const title = 'Hello Page'

const HelloPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <Message />
        <HelloButton />
      </div>
    </div>
  </div>

export default HelloPage
```

- 修改 `src/shared/component/page/home.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import ModalExample from '../modal-example'
import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <div className="jumbotron">
      <div className="container">
        <h1 className="display-3 mb-4">{APP_NAME}</h1>
      </div>
    </div>
    <div className="container">
      <div className="row">
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Bootstrap</h3>
          <p>
            <button type="button" role="button" data-toggle="modal" data-target=".js-modal-example" className="btn btn-primary">Open Modal</button>
          </p>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">JSS (soon)</h3>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Websockets</h3>
          <p>Open your browser console.</p>
        </div>
      </div>
    </div>
    <ModalExample />
  </div>

export default HomePage
```

- 修改 `src/shared/component/page/not-found.jsx`：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'
import { Link } from 'react-router-dom'
import { HOME_PAGE_ROUTE } from '../../routes'

const title = 'Page Not Found!'

const NotFoundPage = () =>
  <div className="container mt-4">
    <Helmet title={title} />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <div><Link to={HOME_PAGE_ROUTE}>Go to the homepage</Link>.</div>
      </div>
    </div>
  </div>

export default NotFoundPage
```

- 修改 `src/shared/component/button.jsx`：

```js
// [...]
<button
  onClick={handleClick}
  className="btn btn-primary"
  type="button"
  role="button"
>{label}</button>
// [...]
```

- 创建 `src/shared/component/footer.jsx`：

```js
// @flow

import React from 'react'
import { APP_NAME } from '../config'

const Footer = () =>
  <div className="container mt-5">
    <hr />
    <footer>
      <p>© {APP_NAME} 2017</p>
    </footer>
  </div>

export default Footer
```

- 创建 `src/shared/component/modal-example.jsx`：

```js
// @flow

import React from 'react'

const ModalExample = () =>
  <div className="js-modal-example modal fade">
    <div className="modal-dialog">
      <div className="modal-content">
        <div className="modal-header">
          <h5 className="modal-title">Modal title</h5>
          <button type="button" className="close" data-dismiss="modal">×</button>
        </div>
        <div className="modal-body">
          This is a Bootstrap modal. It uses jQuery.
        </div>
        <div className="modal-footer">
          <button type="button" role="button" className="btn btn-primary" data-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>

export default ModalExample
```

- 修改 `src/shared/app.jsx`：

```js
const App = () =>
  <div style={{ paddingTop: 54 }}>
```

这是一个 *React 行内样式* 的示例。

这段代码在 DOM 中会被转换成： `<div style="padding-top:54px;">`。 [React 行内样式](https://speakerdeck.com/vjeux/react-css-in-js) 把你从 CSS 全局命名空间里解放出来，让组件作用域成为可能。但是这样做也有代价：某些原生的 CSS 特点还没有被支持，比如说 `:hover`，媒体查询，动画或者 `font-face` 就不能用了。这也是我们稍后引入 CSS-in-JS，JSS 库的[原因之一](https://github.com/cssinjs/jss/blob/master/docs/benefits.md#compared-to-inline-styles)。

- 修改 `src/shared/component/nav.jsx`：

```js
// @flow

import $ from 'jquery'
import React from 'react'
import { Link, NavLink } from 'react-router-dom'
import { APP_NAME } from '../config'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../routes'

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

const Nav = () =>
  <nav className="navbar navbar-toggleable-md navbar-inverse fixed-top bg-inverse">
    <button className="navbar-toggler navbar-toggler-right" type="button" role="button" data-toggle="collapse" data-target=".js-navbar-collapse">
      <span className="navbar-toggler-icon" />
    </button>
    <Link to={HOME_PAGE_ROUTE} className="navbar-brand">{APP_NAME}</Link>
    <div className="js-navbar-collapse collapse navbar-collapse">
      <ul className="navbar-nav mr-auto">
        {[
          { route: HOME_PAGE_ROUTE, label: 'Home' },
          { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
          { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
          { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
        ].map(link => (
          <li className="nav-item" key={link.route}>
            <NavLink to={link.route} className="nav-link" activeStyle={{ color: 'white' }} exact onClick={handleNavLinkClick}>{link.label}</NavLink>
          </li>
        ))}
      </ul>
    </div>
  </nav>

export default Nav
```

这里添加了点新东西：`handleNavLinkClick`。在开发 SPA(单页面应用)时，我用 Bootstrap 的 `navbar` 时遇到了一个问题：在手机上点击链接的时候，菜单栏不会折叠，而且没有滚动到页面顶部。这正好给我机会，向你演示一下怎样在你的 app 中结合使用 jQuery 和 Bootstrap 的某些代码。

```js
import $ from 'jquery'
// [...]

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

<NavLink /* [...] */ onClick={handleNavLinkClick}>
```

**注意**: 为了 *本教程代码* 的可读性，我移除了一些易访问性相关的属性 (比如 `aria` 属性)。在实际开发时，**你当然应该把这些属性加回来**。阅读 Bootstrap 文档和代码示例，研究下怎么使用它们。

🏁 现在你的 app 终于用了 Bootstrap 的样式。

## CSS 发展现状

2016 年的 JavaScript 技术栈之争已经尘埃落定。在本教程中使用到的库和工具应该能让你站在 *工业标准的前沿阵地*（*然而 —— 即使是这样，本教程还是可能在一年后完全过时 —— O__O*）。必须承认，这个技术栈设置起来有点复杂；但是，至少大多数前端开发者认为 React-Redux-Webpack 是前端的发展方向。说到 CSS，我就有点悲观了 —— 什么都没定下来，没有标准化的方向，也没有标准的技术栈。

SASS， BEM， SMACSS， SUIT， Bass CSS， React Inline Styles， LESS， Styled Components， CSSX， JSS， Radium， Web Components， CSS Modules， OOCSS， Tachyons， Stylus， Atomic CSS， PostCSS， Aphrodite， React Native for Web（都是术语，就不翻译啦 O——0 ），还有很多我已经忘了名字，不过照样能完成工作的工具。这些工具都很棒，但问题是，没有一种工具占压倒性优势，这就让人头大了。

React 党偏爱行内样式，CSS-in-JS 或者 CSS Modules，因为这些工具能和 React 组合得完美无瑕；而且还能用编程的方式来解决 CSS 的一些常见 [问题](https://speakerdeck.com/vjeux/react-css-in-js)。

CSS Modules 挺好用，但它不能完全发挥 JavaScript 的威力。它只是提供了不错的封装，但在我看来，React 行内样式和 CSS-in-JS 完全把写样式带到了一个新高度。我个人建议是普通的样式就用 React 行内样式（你在 React Native 中也是用它）；当要用 `:hover` 或者媒体查询的时候，就用 CSS-in-JS。

有 [太多 CSS-in-JS 的库](https://github.com/MicheleBertoli/css-in-js)了。JSS 是一个功能全面、写法简单、 [性能优异](https://github.com/cssinjs/jss/blob/master/docs/performance.md) 的库。

## JSS

> 💡 **[JSS](http://cssinjs.org/)** 是一个用 JavaScript 来写样式，并把样式插入 app 中的 CSS-in-JS 库。

基本的 Bootstrap 模板已经定义好了，现在我们加入一些自定义的样式。我之前提到过 React 行内样式处理不了 `:hover` 和媒体查询，所以我们就在首页展示一下怎么用 JSS 来解决这些问题。JSS 可以通过 `react-jss` 实现。

- 运行 `yarn add react-jss`

以下内容添加到 `.flowconfig` 文件，因为 Flow 目前和 JSS 兼容还有点 [问题](https://github.com/cssinjs/jss/issues/411)：

```flowconfig
[ignore]
.*/node_modules/jss/.*
```

### Server-side（服务端）

JSS 可以在服务端初始化的时候渲染样式。

- 以下变量添加到 `src/shared/config.js`：

```js
export const JSS_SSR_CLASS = 'jss-ssr'
export const JSS_SSR_SELECTOR = `.${JSS_SSR_CLASS}`
```

- 修改 `src/server/render-app.jsx`：

```js
import { SheetsRegistry, SheetsRegistryProvider } from 'react-jss'
// [...]
import { APP_CONTAINER_CLASS, JSS_SSR_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
// [...]
const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const sheets = new SheetsRegistry()
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <SheetsRegistryProvider registry={sheets}>
          <App />
        </SheetsRegistryProvider>
      </StaticRouter>
    </Provider>)
  // [...]
      <link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
      <style class="${JSS_SSR_CLASS}">${sheets.toString()}</style>
  // [...]
```

## Client-side（客户端）

客户端渲染 app 后的第一件事情，就是处理服务端生成的 JSS 样式。

- 以下内容放在 `src/client/index.jsx` 文件的 `ReactDOM.render` 方法后面（在 `setUpSocket(store)` 之前）：

```js
import { APP_CONTAINER_SELECTOR, JSS_SSR_SELECTOR } from '../shared/config'
// [...]

const jssServerSide = document.querySelector(JSS_SSR_SELECTOR)
// flow-disable-next-line
jssServerSide.parentNode.removeChild(jssServerSide)

setUpSocket(store)
```

修改 `src/shared/component/page/home.jsx` ：

```js
import injectSheet from 'react-jss'
// [...]
const styles = {
  hoverMe: {
    '&:hover': {
      color: 'red',
    },
  },
  '@media (max-width: 800px)': {
    resizeMe: {
      color: 'red',
    },
  },
  specialButton: {
    composes: ['btn', 'btn-primary'],
    backgroundColor: 'limegreen',
  },
}

const HomePage = ({ classes }: { classes: Object }) =>
  // [...]
  <div className="col-md-4 mb-4">
    <h3 className="mb-3">JSS</h3>
    <p className={classes.hoverMe}>Hover me.</p>
    <p className={classes.resizeMe}>Resize the window.</p>
    <button className={classes.specialButton}>Composition</button>
  </div>
  // [...]

export default injectSheet(styles)(HomePage)
```

和 React 行内样式不同的是，JSS 使用了 class。样式作为参数传递给 `injectSheet`，最终，CSS 的 class 作为属性传递给组件。

🏁 运行 `yarn start` 和 `yarn dev:wds`。打开主页，查看页面源文件（不是审查元素）。你会发现，初始化渲染的时候，JSS 是在 DOM 中的。初始化时，JSS 在 `<style class="jss-ssr">` 元素中（只有主页是这样）。审查元素的方式，就找不到 JSS 了，因为它已经被 `<style type="text/css" data-jss data-meta="HomePage">` 替换掉了。

**注意**：在生产环境中，`data-meta` 会被混淆。酷！

鼠标悬停在 "Hover me" 标签，标签会变成红色。resize 浏览器窗口大小，使浏览器宽度小于 800px， "Resize your window" 标签会变成红色。绿色的按钮用 JSS 的 `composes` 属性扩展了 Bootstrap 的 CSS classes。

下一章： [09 - Travis, Coveralls, Heroku](09-travis-coveralls-heroku.md#readme)

回到 [上一章](07-socket-io.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
