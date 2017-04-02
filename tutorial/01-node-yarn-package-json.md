# 01 - Node, Yarn, and `package.json`

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/01-node-yarn-package-json).

这一节中我们会配置 Node, Yarn, 初始的 `package.json` 文件，并尝试安装一个包。

## Node

> 💡 **[Node.js](https://nodejs.org/)** 是一个 JavaScript 运行环境。它多用于后端开发，但也可用于脚本。在前端开发环境中，它被用来做一大堆事情，比如：检查代码规范、测试以及组合文件。

在该教程中，我们处处用到 Node，所以你需要安装它。**macOS** 和 **Windows** 用户可访问[下载页](https://nodejs.org/en/download/current/) ，Linux 版本系统用户 [安装地址](https://nodejs.org/en/download/package-manager/) 。



举例，**Ubuntu / Debian** 用户，请运行下面的命令来安装 Node：

```sh
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Node 版本需要高于 6.5.0。

## Node 版本管理工具

如果你想灵活切换 Node 版本，请查看 [NVM](https://github.com/creationix/nvm) 或 [tj/n](https://github.com/tj/n)。

## NPM

NPM 是 Node 默认的包管理器。安装 Node 的时候，它也会自动安装。包管理器用来安装和管理包（包就是你或者他人写的代码模块）。在该教程中，我们会用到大量包，不过，我们用到的是另一个包管理器， Yarn。

## Yarn


> 💡 **[Yarn](https://yarnpkg.com/)** 是一个 Node.js 的包管理器，与 NPM 相比，它更快，提供离线支持，依赖关系确定性 [更多](https://yarnpkg.com/en/docs/yarn-lock).

自2016年10月[发布](https://code.facebook.com/posts/1840075619545360)以来，它一跃成为 JavaScript 社区流行的包管理器。如果你坚持使用 NPM，那么请把本教程中所有的 `yarn add` 和 `yarn add --dev` 命令，替换为 `npm install --save` 和 `npm install --save-dev`。

根据下面的[说明](https://yarnpkg.com/en/docs/install)来安装 Yarn。为了[避免](https://github.com/yarnpkg/yarn/issues/1505)依赖于其他包管理器的问题，如果你的系统是 macOS 或者 Unix，我建议你使用 **脚本安装**。

```sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## `package.json`

> 💡 **[package.json](https://yarnpkg.com/en/docs/package-json)** 用来描述和配置你的 JavaScript 项目。它包含了项目的基本信息（项目名、版本号、贡献者、证书等等），工具的配置以及一系列可运行的 *任务*。

- 创建一个新的工作文件夹， `cd` 进入。
- 运行 `yarn init` 回答问题(或者运行 `yarn init -y` 跳过所有问题)来自动创建 `package.json` 文件。

下面是一个初始化的 `package.json` 文件，我会在教程中使用它。

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT"
}
```

## Hello World

- 创建 `index.js` 文件，加入一行代码： `console.log('Hello world')`

🏁 在该文件夹下运行 `node .` (Node 默认执行 `index.js`). 命令行正确的输出应该是 "Hello world"。

**注意**：看到 🏁 这个 emoji 表情了么？每次你到达 **检查点** 的时候，我都会用这个表情。有时候，我们需要连续对代码做出一大堆更改，在到达检查点之前，你的代码可能还不能正常运行。

## `start` 脚本

运行 `node .` 来跑我们的程序有点太 low 了。接下来，我们要用 NPM/Yarn 脚本来执行代码。这样，即使我们的程序变得越来越复杂，我们也能通过 `yarn start` 来运行程序。

- 在 `package.json` 中, 添加 `scripts` 对象，如下所示：

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {
    "start": "node ."
  }
}
```

`start` 是我们给 *任务* 的名字。接下来的教程中，我们会在 `scripts` 对象中，创建一系列不同的任务。一般来说， `start` 是一个应用的默认任务名；其他标准的任务名称有 `stop` 和 `test`。

`package.json` 必须是一个正确的 JSON  文件，你不能随意添加逗号。所以，手动修改 `package.json` 的时候，一定要小心。

🏁 运行 `yarn start`。正确输出是： `Hello world`.

## Git 和 `.gitignore`

- 运行 `git init` 来初始化 git 仓库。

- 创建 `.gitignore` 文件，并添加内容。

```gitignore
.DS_Store
/*.log
```

`.DS_Store` 文件是 macOs 系统自动生成的文件，你不需要在 git 仓库中提交这些文件。

`npm-debug.log` 和 `yarn-error.log` 是包管理器出错的时候生成的文件，仓库中也不需要提交这些文件。

## 安装并使用一个包

在这一节中，我们将要安装并使用一个包。简单来说，一个“包”是他人写的一段代码，你可以在你自己的代码中使用它。这段代码可以是任何东西。下面的例子中，我们将使用一个和颜色相关的包。

- 运行 `yarn add color` 来安装社区提供的包 `color`

打开 `package.json` 会发现 Yarn 在 `dependencies` 中自动添加了 `color`。

`node_modules` 文件夹被自动创建，它用来存储刚刚下载的包。

- 在 `.gitignore` 添加 `node_modules/`

你可能已经注意到，`yarn.lock` 已经被 Yarn 自动生成了。你需要在仓库中提交这个文件，因为它保证团队成员使用的包属于同一个版本。如果你坚持使用 NPM，与 `yarn.lock` 相对应的是 *shrinkwrap*。

- 在 `index.js` 文件中添加如下代码:

```js
const color = require('color')

const redHexa = color({ r: 255, g: 0, b: 0 }).hex()

console.log(redHexa)
```

🏁 运行 `yarn start`。 正确输出： `#FF0000`。

恭喜！你已经成功地安装并运行了一个包。

本节中使用的 `color` 只是为了说明怎样使用一个简单的包。我们不再需要他，可以用命令卸载：

- 运行 `yarn remove color`

## 两种不同的依赖

包的依赖有两种，`"dependencies"` 和 `"devDependencies"`：


**Dependencies** 是那些在应用中被直接使用的包 (例如 React, Redux, Lodash, jQuery, 等等)。运行命令 `yarn add [package]` 来安装这些包。

**Dev Dependencies** 是那些开发时或者是打包时要用到的包 (例如 Webpack, SASS, linters, testing frameworks, 等等)。运行命令 `yarn add --dev [package]` 来安装这些包。

下一章: [02 - Babel, ES6, ESLint, Flow, Jest, Husky](02-babel-es6-eslint-flow-jest-husky.md#readme)

回到[目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents)。
