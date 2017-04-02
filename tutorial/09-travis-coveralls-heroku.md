# 09 - Travis, Coveralls, and Heroku

本章代码存储在 [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate) 的 `master` 分支上。

在本章，我们的应用会使用第三方服务，你可以下选择付费或者免费服务。在教程中使用这样的服务是有争议的，因为教程应该依赖于社区驱动的、免费开放的工具。为此，我维护了两个分支 [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate), `master` 和 `master-no-services（没有服务的 master 分支）`。

## Travis

> 💡 **[Travis CI](https://travis-ci.org/)** 是一个持续集成的平台，免费且开源。

如果你的 Github 项目是 public 的，那集成 Travis 相当简单。首先，用 Github 账户登录 Travis，并添加代码仓库。

- 然后，创建 `.travis.yml` ：

```yaml
language: node_js
node_js: node
script: yarn test && yarn prod:build
```

Travis 会自动检测到你使用了 Yarn —— 因为你有一个 `yarn.lock` 文件。每次向 Github 仓库提交代码的时候，`yarn test && yarn prod:build` 命令都会被执行。绿色表示代码没有出错。

## Coveralls

> 💡 **[Coveralls](https://coveralls.io)** 服务为你提供测试覆盖率的记录和统计数据。

如果你的项目在 Github 开源，并且兼容与Coveralls 提供的 Continuous Integration（持续集成） 服务，那你唯一要做的事情，就是把 Jest 生成的覆盖率文件导入到 `coveralls`。

- 运行 `yarn add --dev coveralls`

- 编辑 `.travis.yml` 文件：

```yaml
script: yarn test && yarn prod:build && cat ./coverage/lcov.info | ./node_modules/coveralls/bin/coveralls.js
```

现在，Travis 每次构建的时候，都会自动把你的 Jest 覆盖率数据提交给 Coveralls。

## Badges（徽章）

所有代码都通过 Travis 和 Coveralls 了么？很好，现在你可以炫耀一下“金闪闪”的徽章了。

你可以直接只用 Travis 或 Coveralls 提供的代码，或者使用 [shields.io](http://shields.io/) 来美化一下，当然你也可以尝试自定义。本教程选择使用 shields.io ：

- 创建 `README.md` 文件：

```md
[![Build Status](https://img.shields.io/travis/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://travis-ci.org/GITHUB-USERNAME/GITHUB-REPO)
[![Coverage Status](https://img.shields.io/coveralls/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://coveralls.io/github/GITHUB-USERNAME/GITHUB-REPO?branch=master)
```

当然，你需要用你的用户名和仓库名来替换 `GITHUB-USERNAME/GITHUB-REPO`。

## Heroku

> 💡 **[Heroku](https://www.heroku.com/)** 是一个用来部署的 [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) 。选择它，你只要专注于开发，而不用关注其他。

先声明下，本教程没有接受过 Heroku 任何形式的赞助。Heroku 只是一个平台，我选择它，只是为了演示如何部署应用。是的，当你创造了一个伟大的产品，你就会得到别人的喜爱。

**注意**：在之后，我们能会添加 AWS 部分；不过，事情得一件一件来。

### Web 设置

- 如果你还没进行过设置，先安装 [Heroku CLI](https://devcenter.heroku.com/articles/getting-started-with-nodejs) 并登陆。

- 到 [Heroku Dashboard（管理面板）](https://dashboard.heroku.com/) 创建两个应用，一个叫 `your-project`，另一个叫 `your-project-staging`。

我们要让 Heroku 用 Babel 自动转义 ES6/Flow 代码，并用 Webpack 自动为客户端文件打包。但这些包都写在 `devDependencies` 中， Yarn 在 Heroku 生产环境中不会去安装这些包。我们可以修改 `NPM_CONFIG_PRODUCTION` 环境变量来改变这一默认行为。

- 两个 app 中都要修改， Settings > Config Variables 中，添加 `NPM_CONFIG_PRODUCTION`，值设为 `false`。

- 创建一个 Pipeline，让 Heroku 获得访问 Github 的权限。

- 两个 app 都添加到 pipeline，当 `master` 分支发生改变的时候，staging app 需要自动部署；打开  Review Apps。

好，现在准备一下，把 app 部署到 Heroku。

### 本地运行生产环境

- 创建 `.env` 文件：

```.env
NODE_ENV='production'
PORT='8000'
```

这个文件用来存放本地变量；如果你的项目是私密项目，那就不要把这个文件 commit 到公共项目里。

- 添加 `/.env` 到 `.gitignore`

- 创建 `Procfile` 文件：

```Procfile
web: node lib/server
```

该文件用来声明应用的入口。

我们不再使用 PM2，而是使用 `heroku local` 来在本地运行开发环境。

- 运行 `yarn remove pm2`

- 修改 `package.json`:

```json
"prod:start": "heroku local",
```

- 删除 `package.json` 的 `prod:stop` 命令。`heroku local` 进程可以用 Ctrl+C 的方法来关掉，所以我们不再需要这条命令了。

🏁 运行 `yarn prod:build` 和 `yarn prod:start`，你的服务器应该启动成功，并输出一些 logs。

### 部署到生产

- 在 `package.json` 的 `scripts` 中添加一行：

```json
"heroku-postbuild": "yarn prod:build",
```

每次在 Heroku 部署应用，`heroku-postbuild` 命令都会执行。

你可能需要向 Heroku 指明 Node 和 Yarn 的具体版本号。

- `package.json` 文件添加如下内容：

```json
"engines": {
  "node": "7.x",
  "yarn": "0.20.3"
},
```

- 创建 `app.json` 文件：

```json
{
  "env": {
    "NPM_CONFIG_PRODUCTION": "false"
  }
}
```

这是为了 Review Apps 的使用。

Heroku Pipeline 的部署设置终于完成了。

🏁 创建一个 git 分支，改点东西然后发一个 Github Pull 请求来测试 Review App。在 Review App URL 中查看更改，如果一切正确，在 Github 的 `master` 上合并 Pull 请求。几分钟后，你的 staging app 应该就自动部署了。在 staging app URL 上查看更新，如果一切正确，提交 staging 到生产。

恭喜！你完成了整个教程的学习

为你颁发 emoji 荣誉奖牌： 🏅

回到 [上一章](08-bootstrap-jss.md#readme) 或 [目录](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
