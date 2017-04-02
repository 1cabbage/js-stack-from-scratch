# 03 - Express, Nodemon, and PM2

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2)。

本章我们会创建一个 web 服务器，并同时为这个服务器配置开发和生产两种模式。

## Express

> 💡 **[Express](http://expressjs.com/)** 是 Node 社区最流行的框架，API 非常简单，但被各种 *中间件* 扩展后，威力巨大。

接下来配置一个 Express 服务器来提供 HTML 和 CSS。

- 删除 `src` 目录下所有文件。

创建以下文件和文件夹

- 创建 `public/css/style.css` ：

```css
body {
  width: 960px;
  margin: auto;
  font-family: sans-serif;
}

h1 {
  color: limegreen;
}
```

- 创建 `src/client/` 文件夹

- 创建 `src/shared/` 文件夹

这个文件夹下存放 *同构/通用* 的 JS 代码 - 可以同时被客户端和服务端使用的代码。常见的例子是 *路由代码*，在该教程中，我们会在发起异步请求的时候用到路由。现在为了举例，我们只会在该文件夹下存放一些配置常量。

- 创建 `src/shared/config.js` 文件：

```js
// @flow

export const WEB_PORT = process.env.PORT || 8000
export const STATIC_PATH = '/static'
export const APP_NAME = 'Hello App'
```

如果你的 Node 进程有一个 `process.env.PORT` 环境变量 (比如当你部署到 Heroku 的时候)，端口号就是这个环境变量的值。如果没有这个环境变量，我们就把端口号设为 `8000`。

- 创建 `src/shared/util.js` 文件：

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const isProd = process.env.NODE_ENV === 'production'
```

这个简单的工具用来判断当前环境是否是生产环境。`// eslint-disable-next-line import/prefer-default-export` 这行注释是因为我们目前只导出了一个变量；当你添加了其他导出内容后，这行注释可以删掉。

- 运行 `yarn add express compression`

`compression` 是一个用来在服务端开启 Gzip 压缩的 Express 中间件。

- 创建 `src/server/index.js` ：

```js
// @flow

import compression from 'compression'
import express from 'express'

import { APP_NAME, STATIC_PATH, WEB_PORT } from '../shared/config'
import { isProd } from '../shared/util'
import renderApp from './render-app'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

app.get('/', (req, res) => {
  res.send(renderApp(APP_NAME))
})

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' : '(development)'}.`)
})
```

以上代码没什么好说的，基本就是一个 Express 的 Hello World 教程。我们使用了两个静态文件目录： `dist` 用来存放工具转换后生成的文件，`public` 用来存储固有文件。

- 创建 `src/server/render-app.js` ：

```js
// @flow

import { STATIC_PATH } from '../shared/config'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <h1>${title}</h1>
  </body>
</html>
`

export default renderApp
```

我们创建了一个方法，以 `title` 为参数，并把参数插入到页面的 `title` 和 `h1` 标签中，最后返回了 HTML 字符串。我们用 `STATIC_PATH` 常量作为所有静态资源的基础目录。

### HTML 模板字符串语法在 Atom 中高亮 (可选)

如果你的编辑器允许的话，模板字符串中的 HTML 代码可以实现语法高亮。在 Atom 中，如果你的模板字符串以 `html` 开头或结束（例如以`ilovehtml`结束）的话，Atom 会自动高亮其中的字符串。为了利用这一点，我有时候会用 `common-tags` 包的 `html` 标签。

```js
import { html } from `common-tags`

const template = html`
<div>Wow, colors!</div>
`
```

我没有在教程的模板中添加这个技巧，是因为这个技巧目前好像只在 Atom 中适用，并且不太完美。当然，一些 Atom 用户可能觉得很受用。

好了，回到主题！

- 在 `package.json` 中修改 `start` ： `"start": "babel-node src/server",`

🏁 运行 `yarn start` ，在浏览器中打开 `localhost:8000`。如果运行正常的话，你应该看到的是一个标题和内容都是 Hello App 的页面。

**注意**: 某些进程 —— 例如服务器进程 —— 可能会阻止命令行的输入。按下 **Ctrl+C** 就能结束进程。如果你想保持进程运行，也可以新开一个命令行窗口。此外，你也可以让这些进程在后台运行 —— 不过这已经超过了本教程讨论的范围。

## Nodemon

> 💡 **[Nodemon](https://nodemon.io/)** 当文件更改时，这个工具会自动重启服务器。

我们会在 **开发模式** 下使用这个包。

- 运行 `yarn add --dev nodemon`

- 修改 `scripts` ：

```json
"start": "yarn dev:start",
"dev:start": "nodemon --ignore lib --exec babel-node src/server",
```

`start` 任务现在只是指向另一个任务 —— `dev:start`；这为我们提供了一层抽象，使我们之后可以切换默认任务。

在 `dev:start` 中， `--ignore lib` 的作用是忽略 `lib` 文件夹的修改；如果该文件夹下的文件发生更改， *无需* 重启服务器。现在你还没有这个目录，不过我们将在下一节中生成它。Nodemon 的运行默认依赖于 `node` ，但我们让它使用 `babel-node` ，这样我们就能愉快地书写 ES6/Flow 代码了。

🏁 运行 `yarn start` 然后打开 `localhost:8000`。 修改 `src/shared/config.js` 文件下的 `APP_NAME` 常量，我们的修改会触发服务器重启。刷新页面，就能看到更改已经生效。注意，服务器重启和我们即将要说的 *热更新* 是不同的概念。现在我们还需要手动刷新页面，但至少不需要干掉进程然后手动重启了。

## PM2

> 💡 **[PM2](http://pm2.keymetrics.io/)** 是 Node 的进程管理器，保证生产环境下进程的正常运行，同时允许你处理和监控进程。

我们会在 **生产模式** 下使用 PM2。

- 运行 `yarn add --dev pm2`

生产环境下，服务器越高效越好。每一次执行代码，`babel-node` 都会触发 Babel 的代码转换，因此，生产环境下不能用 `babel-node`。我们需要事先用 Babel 转换好代码，运行在服务器上的，应该是已经转换好的 ES5 代码。

Babel 的一大应用就是把一个文件夹（通常命名为 `src`）下的 ES6 代码转换为 ES5 代码，并保存在另一个文件夹下（通常命名为 `lib`）。

`lib` 文件夹是自动生成的；每次生成新文件前，先删除老文件，是公认的最佳实践 —— 否则，一些老文件可能还保留在该文件夹下。`rimraf` 是一个用来实现删除功能的包，特点是用法简单且跨平台。

- 运行 `yarn add --dev rimraf`

把 `prod:build` 任务添加到 `scripts`：

```json
"prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
```

- 运行 `yarn prod:build` 命令，除了 `.test.js`（`.test.jsx`文件也会被忽略）文件外，其他文件应该都被转化并保存于 `lib` 文件夹下了。

- 把 `/lib/` 添加到 `.gitignore`

最后一件事：我们需要传一个名为 `NODE_ENV` 的环境变量给 PM2。Unix 系统用户可以直接用  `NODE_ENV=production pm2` ，但 Windows 用的却是另一个语法。为了通用，我们需要安装 `cross-env` 包。

- 运行 `yarn add --dev cross-env`

修改 `package.json` ：

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon --ignore lib --exec babel-node src/server",
  "prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 运行 `yarn prod:build`，然后运行 `yarn prod:start`。浏览 `http://localhost:8000/` 就能看到你的 APP 了。命令行输出的 logs 内容，应该是 "Server running on port 8000 (production)."。注意：使用 PM2，你的进程是在后台运行的。如果你按下 Ctrl+C，只会终止 `pm2 logs` 命令，但是你的服务器仍在运行。如果想停掉服务器，应该运行 `yarn prod:stop` 命令。

在 push 代码到仓库前，为了保证代码能正常编译，我们应该先运行一下 `prod:build`。但并不是每一次 commit 操作都需要重新编译，所以我建议把它添加到 `prepush` 任务：

```json
"prepush": "yarn test && yarn prod:build"
```

🏁 运行 `yarn prepush` 或者只是 push 代码来触发编译操作。

**注意**: 因为我们没有进行测试，所以 Jest 可能会“抱怨”一下~暂时不去管它！

下一章: [04 - Webpack, React, HMR](04-webpack-react-hmr.md#readme)

回到 [上一章](02-babel-es6-eslint-flow-jest-husky.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
