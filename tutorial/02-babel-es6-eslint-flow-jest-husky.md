# 02 - Babel, ES6, ESLint, Flow, Jest, and Husky

本章代码在 [这里](https://github.com/verekia/js-stack-walkthrough/tree/master/02-babel-es6-eslint-flow-jest-husky)。

我们将在本章使用 ES6 语法，与 ES5 语法相比，它更加优雅。几乎所有的浏览器和 JS 环境都能理解 ES5，但 ES6 还没有得到广泛支持。为此，一个名为 Babel 的工具应运而生。

## Babel

> 💡 **[Babel](https://babeljs.io/)** 是一个将 ES6 代码 转换为 ES5 代码的编译器（一些其他语法，比如 JSX 语法也能够被编译）。 它非常模块化，可被运用于各种 [环境](https://babeljs.io/docs/setup/)。目前为止，它是 React 社区最受推崇的 ES5 编译器。

- 把 `index.js` 移动到新创建的 `src` 文件夹下。 在该文件夹下的文件，使用 ES6 语法。 删除和 `color` 有关的代码，用下面的内容替换：

```js
const str = 'ES6'
console.log(`Hello ${str}`)
```

我们这里使用的 ES6 语法是 *模板字符串*，它允许我们在字符串中通过 `${}` 来插入变量。注意模板语法使用 **反引号**.

- 运行 `yarn add --dev babel-cli` 安装 Babel CLI（命令行工具）。

Babel CLI 有 [两种执行方式](https://babeljs.io/docs/usage/cli/): `babel`，将 ES6 文件转换为 ES5 文件； `babel-node` 可以替换 `node`，用来轻量级地直接执行 ES6 文件。 `babel-node` 适用于开发，但对于生产环境来说，它太笨重了。在本章中，我们使用 `babel-node` 来配置开发环境；接下来，我们将使用 `babel` 来为生产环境打包 ES5 文件。

- 在 `package.json`  `start` 脚本中, 用 `babel-node src` 来替换 `node .` (`index.js` 是 Node 默认执行的文件，因此我们可以省略 `index.js`)。

如果你现在运行 `yarn start` ，输出应该没啥问题；不过 Babel 这时候并起到什么作用。因为我们还没有告诉 Babel 进行何种转换。输出正确的唯一原因是 Node 原生支持了 ES6 语法。然而，某些浏览器以及老版本的 Node 却不一定支持这些最新的语法。

- 运行 `yarn add --dev babel-preset-env` 来安装一个叫 `env` 的 Babel preset 包，它包含了大部分 Babel 支持转化的 ECMAScript 语法配置。

- 在根目录创建 `.babelrc` 文件，它是一个用来配置 Babel 的 JSON 文件。 添加如下内容，使 Babel 使用 `env` preset：

```json
{
  "presets": [
    "env"
  ]
}
```

🏁 `yarn start` 现在能够运行成功，并且 Babel 确实起了点作用。但是为了轻量开发，我们使用了 `babel-node`，所以我们还不能搞清楚 Babel 到底干了什么。 不过，当你读到这一节 [ES6 模块语法](#the-es6-modules-syntax) 时，你就会明白 Babel 的作用了。

## ES6

> 💡 **[ES6](http://es6-features.org/)**: ES6 是 JavaScript 最振奋人心的更新。ES6 新语法太多，我们不可能完整地罗列在这里；不过，我们会简单地介绍一下 `class`， `const` ， `let`， 字符串语法以及箭头函数 (`(text) => { console.log(text) }`)。

### 创建 ES6 class

- 创建一个新文件 `src/dog.js`，使用 ES6 class 语法：

```js
class Dog {
  constructor(name) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

module.exports = Dog
```

如果你之前用过面向对象语言，上面的代码对你来说应该并不陌生。但对 JavaScript 来说，这种语法确实挺新奇的。我们把这个类赋值给 `module.exports`，就导出了它。

在 `src/index.js` 文件中，添加如下代码：

```js
const Dog = require('./dog')

const toby = new Dog('Toby')

console.log(toby.bark())
```

注意，使用我们自己写的包，和使用社区提供的包 `color` 时，引用包的方式是不一样的。当引用我们自己的文件时，我们在 `require()` 中，使用的是相对路径 `./`。

🏁 运行 `yarn start`，正确输出 "Wah wah, I am Toby"。

### ES6 模块语法

我们将用 `import Dog from './dog'` 来替换 `const Dog = require('./dog')`，这是最新的 ES6 模块语法(和 "CommonJS" 模块与法相对应)。但现在 NodeJS 还并不支持这种语法，所以如果代码仍能正常运行的话，就说明 Babel 确实正确处理了我们的代码。

同时，在 `dog.js` 文件中, 用 `export default Dog` 替换 `module.exports = Dog` 。

🏁 `yarn start` 应该能够正常运行，并输出 "Wah wah, I am Toby"。

## ESLint

> 💡 **[ESLint](http://eslint.org)** 是 ES6 代码的检查器。它会根据代码规范，给出合理的提示。ESLint 给出的提示，也能帮你更好地学习 JavaScript。

ESLint 需要和各种 *规范* 结合使用，[规范](http://eslint.org/docs/rules/) 可不少哦。我选择的是 Airbnb 提供的代码规范。为了使用规范，我们需要先安装一些插件。

查阅 Airbnb 最新 [说明](https://www.npmjs.com/package/eslint-config-airbnb)，安装配置包以及它的依赖包。 截至 2017-02-03, Airbnb 建议使用以下命令行：

```sh
npm info eslint-config-airbnb@latest peerDependencies --json | command sed 's/[\{\},]//g ; s/: /@/g' | xargs yarn add --dev eslint-config-airbnb@latest
```

这条命令能安装好所有需要的包，并且在 `package.json` 文件中自动添加了 `eslint-config-airbnb`， `eslint-plugin-import`， `eslint-plugin-jsx-a11y` 以及 `eslint-plugin-react` 。

**注意**: 我把命令中的 `npm install` 替换为 `yarn add`，然而，Windows 系统并不支持。所以如果你遇到了这个问题，请查看 `package.json` 文件，然后用 `yarn add --dev packagename@^#.#.#` 来手动安装所需要的包。 `#.#.#` 代表 `package.json` 每个包的版本号。

- 项目根目录下，创建 `.eslintrc.json`，并添加如下代码：

```json
{
  "extends": "airbnb"
}
```

为了使用 `eslint` 命令行，我们需要安装这个包：

- 运行 `yarn add --dev eslint`

修改 `package.json` 的 `scripts` ，添加一个 `test` 任务：

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src"
},
```

这条命令的意思是检测 `src` 目录下所有的 JS 文件。

我们会用 `test` 任务运行一系列命令来验证我们的代码，包括之后的类型检查和单元测试。

- 运行 `yarn test`，你应该看到 `index.js` 文件中的错误有：分号缺失，使用 `console.log()` 。该文件顶部添加 `/* eslint-disable no-console */` 来允许我们在该文件中使用 `console` 。

**注意**: 如果你是 Windows 系统用户，请保证你已经设置你的 Git 和编辑器使用 Unix 的 LF 换行模式，而不是 Windows 默认的 CRLF 换行模式。如果你的项目只会跑在 Windows 上，你可以在 ESLint 配置文件的 `rules` 中添加 `"linebreak-style": [2, "windows"]` 来强制使用 CRLF 换行模式。

### 分号

这可能是 JS 社区中最容易引起撕逼的话题了，我们也来唠唠。因为 JS 的自动分号插入机制，你可以省略分号。我觉得这个问题只关个人喜好，而无关对错。如果你喜欢 Python， Ruby， 或者 Scala，你可能挺享受省略分号的写法。 但要是你喜欢的是 Java， C#， 或者 PHP，那你应该更喜欢加上分号。

大多数人出于习惯会在写 JS 的时候加上分号。最开始我也是分号党，直到有一天我看了 Redux 文档中的代码示例，然后尝试了一下不写分号。刚开始的感觉有点奇怪，但只是因为不习惯；不过在写了一天代码后，我就变成了一个不折不扣的无分号党。在我看来，不写分号更直观，写起来也更简单。

我建议你读一下 [关于分号的 ESLint 文档](http://eslint.org/docs/rules/semi)。正如在文档中提到的，如果你也想成为一个无分号党，你应该知道在一些极端情况下，分号又是必要的。 ESLint 能够帮助你应对这些极端情况。你需要在 `.eslintrc.json` 中添加 `no-unexpected-multiline` 规则：

```json
{
  "extends": "airbnb",
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

🏁 现在再运行 `yarn test`，就应该没什么错误提示了。在不需要分号的地方加一下分号，看看有没有错误提示。

如果你们坚持使用分号，我表示理解；不过，这可能让你们在使用这个教程的时候不太方便。如果你看这个教程只是为了学习一下，那我保证你在学习过程中不使用分号，是可以忍一忍的；如果你想用教程提供的模板并且是个分号党，你可能要在某些地方做些修改 —— 有了 ESLint 的提示，你改得应该也挺快的。

### Compat

如果你想让你的项目支持更多的浏览器，但不知道要支持的浏览器是否支持某个 API 怎么办？用 [Compat](https://github.com/amilajack/eslint-plugin-compat)！请查看根据 [Can I Use](http://caniuse.com/) 制定的 [浏览器列表](https://github.com/ai/browserslist)

- 运行 `yarn add --dev eslint-plugin-compat`

- 在 `package.json` 中添加如下内容，告诉插件，只要是市场占有率超过百分之一的浏览器，我们都想支持。

```json
"browserslist": ["> 1%"],
```

- 相应的，`.eslintrc.json` 文件也要做出修改：

```json
{
  "extends": "airbnb",
  "plugins": [
    "compat"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2,
    "compat/compat": 2
  }
}
```

为了试试插件好不好用，你可以在你的代码中使用 `navigator.serviceWorker` 或 `fetch` 。一般来说， ESLint 这时候会给出错误提示。

### 编辑器中的 ESLint

本章中我们在命令行中配置了 ESLint，在构建代码或提交代码的时候，会得到错误提示。其实，你也可以让你的 IDE 使用你的配置，这样你就能得到更及时的错误反馈了。别使用 IDE 原生的 ES6 错误提示！通过相应的配置，让编辑器使用你指定的包；这样，你才能在使用其他的编辑器时，得到同样的提示。

（译者注：如果你的编辑器是 Atom 并且安装了 lint 插件，那么在配置完之后，编辑器会自动根据 `.eslintrc.json` 中的配置来检测代码；初次生成配置文件，可能不会检测，可以尝试重启 Atom）

## Flow

> 💡 **[Flow](https://flowtype.org/)**: Facebook 提供的一个静态类型检查器。举个例子，如果你把一个字符串类型的值赋值给一个数值类型的变量，它就会报错。

现在，我们的 JS 代码是标准的 ES6 格式。Flow 能够检查这样的代码，但为了发挥它的最大威力，我们要在我们的代码中加入注释；但这样，我们的代码就不那么符合标准了。为了让 Babel 和 ESLint 在解析我们的代码时不崩溃，我们需要通过配置让它们理解注释。

- 运行 `yarn add --dev flow-bin babel-preset-flow babel-eslint eslint-plugin-flowtype`

`flow-bin` 是在 `scripts` 任务中用的, `babel-preset-flow` 帮助 Babel 理解 Flow 注释， `babel-eslint` 让 ESLint *依赖于 Babel 解析器*， `eslint-plugin-flowtype` 是一个用来检查注释错误的 ESLint 插件。

- 像下面这样修改 `.babelrc` 文件：

```json
{
  "presets": [
    "env",
    "flow"
  ]
}
```

- `.eslintrc.json` 也得改：

```json
{
  "extends": [
    "airbnb",
    "plugin:flowtype/recommended"
  ],
  "plugins": [
    "flowtype",
    "compat"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2,
    "compat/compat": 2
  }
}
```

**注意**: `plugin:flowtype/recommended` 已经告诉 Babel 该用什么解析器了；不过，要是你想更明确点，可以在 `.eslintrc.json` 加上 `"parser": "babel-eslint"` 。

这一节的东西好像有点多，你可以先花几分钟消化一下。我到现在还感到很惊奇，ESLint 竟然能用 Babel 的解析器来解析 Flow 的注释！这俩工具实在是太模块化了。

- 把 `flow` 加入到 `test` 任务：

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow"
},
```

- 根目录下创建一个 `.flowconfig` 文件：

```flowconfig
[options]
suppress_comment= \\(.\\|\n\\)*\\flow-disable-next-line
```

如果你想让 Flow 忽略下一行代码，你可以用上面的注释方法；它的用法和 `eslint-disable` 很像：

```js
// flow-disable-next-line
something.flow(doesnt.like).for.instance()
```

配置部分差不多可以告一段落了。

- 在 `src/dog.js` 加入注释：

```js
// @flow

class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

export default Dog
```

`// @flow` 注释告诉 Flow： 这个文件需要进行类型检查。为了测试，我们的注释基本上就是在参数或方法名后加一个冒号，关于 Flow 注释的更多使用方法，请查看 [文档](https://flowtype.org/docs/quick-reference.html) 。

- 在 `index.js` 文件里也加上  `// @flow` 。

`yarn test` 现在不光进行规范检查，还进行类型检查。

你可以进行下面的尝试：

- 在 `dog.js` 文件中，替换 `constructor(name: string)` 为 `constructor(name: number)`， 然后运行 `yarn test`。如果 **Flow** 提示类型出错，那说明 Flow 运行成功了。

- 替换 `constructor(name: string)` 为 `constructor(name:string)`，运行 `yarn test`。如果  **ESLint** 运行成功地话，它会提示你在冒号后添加一个空格。

🏁 如果你能得到以上两个错误，那说明你的 ESLint 和 Flow 配置成功了；记得把刚刚修改的东西改回去。

### 在编辑器中配置 Flow

和 ESLint 一样，你也应该在你的编辑器中配置 Flow，从而得到及时反馈。

## Jest

> 💡 **[Jest](https://facebook.github.io/jest/)**: Facebook 提供的一个 JS 测试库，配置简单，一步到位，甚至还能用来测试 React 组件。

- 运行 `yarn add --dev jest babel-jest` 安装 Jest 以及对应的 Babel 包。

- 在 `.eslintrc.json` 加入如下内容之后，你就不用再在测试文件里引用 Jest 包了。

```json
"env": {
  "jest": true
}
```

- 创建 `src/dog.test.js` 文件，代码如下：

```js
import Dog from './dog'

test('Dog.bark', () => {
  const testDog = new Dog('Test')
  expect(testDog.bark()).toBe('Wah wah, I am Test')
})
```

- 把 `jest` 添加到 `test` 任务：

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage"
},
```

`--coverage` 让 Jest 自动生成测试覆盖率信息。观察覆盖率信息，就能知道哪些文件缺乏测试了。覆盖率信息保存在 `coverage` 文件夹下。

- 把 `/coverage/` 添加到 `.gitignore`

🏁 运行 `yarn test`，在规范检测和类型检测之后，它应该会进行覆盖率测试并展示覆盖率表，因为测试通过，展示的结果应该是绿色的。

## 用 Husky 添加 Git 钩子

> 💡 **[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)**: 在特定操作（例如 push 或者 commit 操作）前会被执行的操作。

目前，`test` 任务帮助我们进行代码测试；为了避免向代码仓库中提交垃圾代码，我们需要在每一次 `git commit` 和 `git push` 前，自动检测代码。

[Husky](https://github.com/typicode/husky) 一个便捷的设置 Git 钩子的包。

- 运行 `yarn add --dev husky`

我们只需要在 `scripts` 添加两个任务 —— `precommit` 和 `prepush`：

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 现在试着 commit 或者 push 代码， `test` 任务就会自动执行。

如果运行出错，那一般是因为 `yarn add --dev husky` 这个命令没有正确安装 Git 钩子。我自己没遇到过这种情况，但其他运气不怎么好的人遇到过。 如果你是不幸者之一，试试 `yarn add --dev husky --force`, 可以的话，记得在这里说明一下你的状况 [this issue](https://github.com/typicode/husky/issues/84)。

**注意**：如果你是在 commit 之后执行 push 操作，为了避免重复测试，你应该使用 `git push --no-verify` 命令。

下一章： [03 - Express, Nodemon, PM2](03-express-nodemon-pm2.md#readme)

回到 [上一章](01-node-yarn-package-json.md#readme) 或者 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
