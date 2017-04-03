# 从零开始构建 JavaScript 技术栈 - 中文版

阅读[从零开始构建 JavaScript 技术栈 - 英文版](https://github.com/verekia/js-stack-from-scratch)


[![Build Status](https://travis-ci.org/verekia/js-stack-from-scratch.svg?branch=master)](https://travis-ci.org/verekia/js-stack-from-scratch)
[![Release](https://img.shields.io/github/release/verekia/js-stack-from-scratch.svg?style=flat-square)](https://github.com/verekia/js-stack-from-scratch/releases)
[![Dependencies](https://img.shields.io/david/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate)
[![Dev Dependencies](https://img.shields.io/david/dev/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate?type=dev)
[![Gitter](https://img.shields.io/gitter/room/js-stack-from-scratch/Lobby.svg?style=flat-square)](https://gitter.im/js-stack-from-scratch/)

[![React](/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Redux](/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/img/yarn-padded-90.png)](https://yarnpkg.com/)
[![Webpack](/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Bootstrap](/img/bootstrap-padded-90.png)](http://getbootstrap.com/)

欢迎来到 JavaScript 技术栈指南： **从零开始构建 JavaScript 技术栈**.

> 🎉 **这是本教程的第二版，与2016年发布的版本相比，更新了不少东西。欢迎查看 [更新日志](/CHANGELOG.md)!**

该指南讲的都是一些可实操的知识。即使是这样，阅读者还是需要具备一些简单的编程知识和 JavaScript 基础。 **教程的核心是把各种工具结合起来** 并且为每一种工具提供了 **最简单的例子**。 学习该教程后，你可以尝试 *从零编写你自己的 JavaScript 技术栈模板*。 因为本教程的核心是结合各种工具的使用，我并没有详细地讲解每一种工具该怎么用。如果你想要进一步了解浙西工具的使用，请参考它们的文档或指南。

如果你只是想构建一个简单的 web 页面，你可能并不需要这个完整的技术栈（把 Browserify/Webpack + Babel + jQuery 结合起来就足够了）；但如果你需要构建一个可伸缩的 web app，并且需要配置各种环境，那这个教程正好适合你。

该教程的大量描述都和 React 有关。如果你是个新手并且想要学习 React，[create-react-app](https://github.com/facebookincubator/create-react-app) 预设的配置能让你迅速搭建好 React 的运行环境。对于那些刚加入使用 React 团队的人来说，我建议他们使用 create-react-app 来快速搭建学习环境。在本教程中，你不会使用任何预配置，因为我希望你能够理解那些配置到底起了什么作用。

每一章的代码示例都包含在教程中，你可以通过 `yarn && yarn start` 来运行这些例子。不过，我建议你按着 **详细指南** 来把每一行代码都自己写一遍。

最终代码在 [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate), 和 [releases](https://github.com/verekia/js-stack-from-scratch/releases). 在线示例： [live demo](https://js-stack.herokuapp.com/) 。

可运行于 Linux， macOS， 以及 Windows。

## 目录

[01 - Node, Yarn, `package.json`](/tutorial/01-node-yarn-package-json.md#readme)

[02 - Babel, ES6, ESLint, Flow, Jest, Husky](/tutorial/02-babel-es6-eslint-flow-jest-husky.md#readme)

[03 - Express, Nodemon, PM2](/tutorial/03-express-nodemon-pm2.md#readme)

[04 - Webpack, React, HMR](/tutorial/04-webpack-react-hmr.md#readme)

[05 - Redux, Immutable, Fetch](/tutorial/05-redux-immutable-fetch.md#readme)

[06 - React Router, Server-Side Rendering, Helmet](/tutorial/06-react-router-ssr-helmet.md#readme)

[07 - Socket.IO](/tutorial/07-socket-io.md#readme)

[08 - Bootstrap, JSS](/tutorial/08-bootstrap-jss.md#readme)

[09 - Travis, Coveralls, Heroku](/tutorial/09-travis-coveralls-heroku.md#readme)

## 即将添加的内容

配置你的编辑器 (Atom 优先)， MongoDB， Progressive Web App， E2E testing。

## 翻译

如果想添加你的翻译，请先阅读 [translation recommendations](/how-to-translate.md) 。

### V2

- [Italian](https://github.com/fbertone/guida-javascript-moderno) by [Fabrizio Bertone](https://github.com/fbertone) - [fbertone.it](http://fbertone.it)
- [简体中文](https://github.com/yepbug/js-stack-from-scratch/) by [@yepbug](https://github.com/yepbug)

查看 [进行中的翻译](https://github.com/verekia/js-stack-from-scratch/issues/147).

### V1

- [中文](https://github.com/pd4d10/js-stack-from-scratch) by [@pd4d10](http://github.com/pd4d10)
- [Italiano](https://github.com/fbertone/js-stack-from-scratch) by [Fabrizio Bertone](https://github.com/fbertone)
- [日本語](https://github.com/takahashim/js-stack-from-scratch) by [@takahashim](https://github.com/takahashim)
- [Русский](https://github.com/UsulPro/js-stack-from-scratch) by [React Theming](https://github.com/sm-react/react-theming)
- [ไทย](https://github.com/MicroBenz/js-stack-from-scratch) by [MicroBenz](https://github.com/MicroBenz)

## Credits

Created by [@verekia](https://twitter.com/verekia) – [verekia.com](http://verekia.com/).

License: MIT
