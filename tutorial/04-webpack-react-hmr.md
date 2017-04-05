# 04 - Webpack, React, and Hot Module Replacement

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** 是一个 *用来打包的模块*。它能把各种各样的文件打包进一个文件（通常情况下是这样）内，你只需要引用这一个文件就可以。

下面是一个用 Webpack 来打包的 *hello world* 示例。

- 在 `src/shared/config.js` 文件，添加如下内容：

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- 创建 `src/client/index.js` ：

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

如果你需要在你的代码里应用 ES 的最新特点，比如 `Promise`，那你需要先在模块里导入 [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/)。

- 运行 `yarn add babel-polyfill`

如果现在运行 ESLint，应该有错误提示 `document` 未定义。

- 修改 `.eslintrc.json` ，允许 `window` 和 `document` 等浏览器对象的使用。

```json
"env": {
  "browser": true,
  "jest": true
}
```

现在，需要把我们用 ES6 写的客户端代码打包成 ES5 文件。

- 创建 `webpack.config.babel.js` ：

```js
// @flow

import path from 'path'

import { WDS_PORT } from './src/shared/config'
import { isProd } from './src/shared/util'

export default {
  entry: [
    './src/client',
  ],
  output: {
    filename: 'js/bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: isProd ? '/static/' : `http://localhost:${WDS_PORT}/dist/`,
  },
  module: {
    rules: [
      { test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/ },
    ],
  },
  devtool: isProd ? false : 'source-map',
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    port: WDS_PORT,
  },
}
```

这个文件描述如何打包：`entry` 是我们 app 的入口，`output.filename` 是最终生成的打包文件名，`output.path` 和 `output.publicPath` 代表最终文件夹和 URL 地址。最终自动生成的内容会被打包进 `dist` 文件夹。`module.rules` 告诉 Webpack 对匹配到的文件进行何种操作。比如，我们要求 Webpack 用 `babel-loader` 来处理 `.js` 和 `.jsx` 文件; `node_modules` 文件夹里的内容不需要处理。`resolve` 指明 Webpack 自动识别哪些后缀 —— 当我们用 `import` 导入的时候，文件的扩展名就可以省略了。最后，我们声明了 Webpack 开发服务器的端口。

**注意**: `.babel.js` 扩展名利用了 Webpack 的一个特点：有这个扩展名的配置文件，会自动应用 Babel 转换，所以我们可以在配置文件中使用 ES6 语法。

`babel-loader` 是 Webpack 用来转换 ES6 代码的一个插件。我们在教程开头就做过转换代码的操作，不过这一次，代码需要运行在浏览器上，而不是跑在服务器上。

- 运行 `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` 是 `babel-loader` 的一个依赖。

- 把 `/dist/` 添加到 `.gitignore`

### 更新任务

为了在开发环境中使用热替换技术，我们需要用到 `webpack-dev-server`；在生产环境中，我们用到的则是 `webpack` 生成的包。无论在开发环境还是生产环境，`--progress` 都应该加上 —— 它会在命令行展示 Webpack 的运行状况。在生产环境，为了压缩代码，还应该加上 `-p`，并且把 `NODE_ENV` 的值设为 `production`。

`scripts` 修改后：

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

`dev:start` 会监听 `.js` 和 `.jsx` 文件的更新，但会忽略 `dist` 文件夹下的更新。

`lint` 任务还会检查 `webpack.config.babel.js` 文件的代码规范。

- 接下来，在 `src/server/render-app.js` 中，为我们的 app 创建一个容器并导出。

```js
// @flow

import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <div class="${APP_CONTAINER_CLASS}"></div>
    <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
  </body>
</html>
`

export default renderApp
```

如果是在开发环境，引用的就是 Webpack 服务器的代码；如果是生产环境，引用的则是 Webpack 打包后的代码。注意，在开发模式下，Webpack 服务器的包是 *虚拟的*，`dist/js/bundle.js` 不是从硬盘里读出来的，而是存储于内存中。Webpack 服务器的端口号应该和主服务器的端口号保持不同。


- 最后，在 `src/server/index.js` 中，把 `console.log` 信息修改成：

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

如果开发者运行了 `yarn start`，但忘记启动 Webpack 服务器，上面的 Log 信息给出了足够的提示。

我们改的东西够多了，让我们来看看运行是否成功：

🏁 命令行运行 `yarn start`，打开另一个命令行窗口，运行 `yarn dev:wds` 。等 Webpack 打完包并且生成好 sourcemaps (两个文件应该都在 600kB 左右)，浏览器访问 `http://localhost:8000/`，看到的该是 "Hello Webpack!"。打开 Chrome 开发者模式，在 Source 面板下，看看哪些文件被引入了。`localhost:8000/` 域名下只有 `static/css/style.css` 文件；所有 ES 代码都属于 `webpack://./src`。这说明 sourcemaps 没出错。编辑 `src/client/index.js`，把 `Hello Webpack!` 改成其他的字符串；你一保存修改，Webpack 服务器就会生成一个新的包，Chrome 也会自动重新加载。

- Ctrl+C 关掉进程，运行 `yarn prod:build` 后再运行 `yarn prod:start`。 浏览器打开 `http://localhost:8000/`，查看 Source 面板。现在 `static/js/bundle.js` 应该是属于 `localhost:8000/`，而不是 `webpack://` 了。浏览 `bundle.js`，看看代码是否已经压缩了。使用 `yarn prod:stop` 来结束进程。

干的漂亮！这部分内容有点多，你可以休息下~接下来的内容，相对简单一些。

**注意**：我建议至少打开三个命令行窗口，一个用来运行 Express 服务器，一个用来运行 Webpack 服务器，另一个用来操作 Git，测试和其他常规操作（比如用 `yarn` 安装包）。

## React

> 💡 **[React](https://facebook.github.io/react/)** 是 Facebook 提供的一个构建前端页面的库。 **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** 语法让我们能够在 JS 里创建 HTML 元素和组件。

本节，我们会用 React 和 JSX 来简单渲染一些文本。

首先，安装 React 和 ReactDOM：

- 运行 `yarn add react react-dom`

把 `src/client/index.js` 文件重命名为 `src/client/index.jsx` ，并修改代码：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- 创建 `src/client/app.jsx` ：

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

既然我们用了 JSX 语法，我们就要通知 Babel 用 `babel-preset-react` 来进行转换；为了对 React 组件进行类型检查，我们需要安装 `flow-react-proptypes` 插件。

- 运行 `yarn add --dev babel-preset-react babel-plugin-flow-react-proptypes`， 然后修改 `.babelrc`：

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ],
  "plugins": [
    "flow-react-proptypes"
  ]
}
```

🏁 运行 `yarn start` 和 `yarn dev:wds`，浏览器中访问 `http://localhost:8000`，应该看到 "Hello React!"。

修改 `src/client/app.jsx` 中的文本，Webpack 会自动重新加载页面；这已经非常简单了，但接下来，我们要做得更好。

## Hot Module Replacement（热替换）

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) —— 不用重新加载全部资源，就能进行实时更新。

为了搭配 HMR 使用 React，我们需要做一些小调整

- 运行 `yarn add react-hot-loader@next`

- 修改 `webpack.config.babel.js` ：

```js
import webpack from 'webpack'
// [...]
entry: [
  'react-hot-loader/patch',
  './src/client',
],
// [...]
devServer: {
  port: WDS_PORT,
  hot: true,
},
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin(),
  new webpack.NoEmitOnErrorsPlugin(),
],
```

- 修改 `src/client/index.jsx` ：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = AppComponent =>
  <AppContainer>
    <AppComponent />
  </AppContainer>

ReactDOM.render(wrapApp(App), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp), rootEl)
  })
}
```

`App` 必须是 `react-hot-loader` 导出的 `AppContainer` 的一个子元素；热更新的时候，我们需要把 `App` 的最新版本重新 `require`。为了保持代码整洁和 DRY，我们创建了一个名为 `wrapApp` 的方法；在两处需要渲染 `App` 的地方，都用到了这个方法。出于代码可读性的考虑，你可以把 `eslint-disable global-require` 写在该文件的最顶部。

🏁 重启 `yarn dev:wds` 进程并在浏览器访问 `localhost:8000`。在开发者模式下，你会看到浏览器输出了一些和 HMR 相关的日志。随便修改点 `src/client/app.jsx` 文件中的内容，你的修改会很快投射到浏览器中，并且没有整页刷新。

下一章： [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

回到 [上一章](03-express-nodemon-pm2.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
