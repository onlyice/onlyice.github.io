---
layout: post
title: "React + Ant Design 快速上手"
tags: 
- 编程技术
---

## 背景及目标

日常开发过程中，经常需要做一些 Web 管理平台供同事或自己使用。作为一个后台开发前台知识不扎实，很难简单快速做出易用的 Web 应用。但是 React 和 Ant Design 的出现使得这种情况有所改善。

这篇文章提供了一个路线图，供没有太多前台经验的开发快速上手，做出有一些交互的 Web 管理端。重点集中在以下几点：

*   提供两个学习路径，分别针对「快速上手」和「系统学习」两个场景
*   提供足够优秀的学习资源，覆盖重点内容
*   提供一种避开其他不必要的复杂性的方法，比如尽量避开前端构建工具

<!--more-->

## 为什么要用 React + Ant Design？

React 的优点：

1.  组件化，写一个 UI 组件可以到处用（当然写通用的组件也不容易）
2.  单向数据流，组件化的基本条件，使用组件跟函数调用一样简单

Ant Design 的优点：

1.  提供了一堆高质量的 [UI 组件](https://ant.design/docs/react/introduce-cn)，应有尽有

## 使用 React 要求的背景知识

使用 React 主要要求这些背景知识：

*   JavaScript
*   前端构建过程
    *   包管理器（Package Manager）
    *   转译器（Transpiler）
    *   构建、打包工具（Bundler）

下面的内容会提供两种学习的途径，一种是「快速上手」，适合简单粗暴地学一点就上手用；另外一种是「系统学习」，适合系统性地了解。

### JavaScript

JavaScript 语言有几版：

|-----+------+---------|
| 规范 | 时间 | 支持程度 |
|:-----|:----|:---------|
| ECMAScript 3 (ES3) | December 1999 | 广泛支持 |
| ECMAScript 5 (ES5) | December 2009 | [ES5 Compatibility Table](https://kangax.github.io/compat-table/es5/) |
| ECMAScript 6 (ES6) / ECMAScript 2015 (ES2015) | June 2015 | [ES6 Compatibility Table](https://kangax.github.io/compat-table/es6/) |
| ECMAScript 7 (ES7) / ECMAScript 2016 (ES2016) | June 2016 | [ES.Next Compatibility Table](https://kangax.github.io/compat-table/es2016plus/) |

其中比较关键的节点是 ES6 (ES2015)，加入了非常多的特性，更容易写 JS 代码的同时也引入了很多复杂度。对于 React 只需要了解 ES6 中的部分特性即可，但是 ES5 及之前的 JS 基础是需要掌握的。

如果你完全没有 JS 经验，那么：

*   ES5 及以前：
    *   快速上手：模仿别人写的代码照着写，以写 Python 的风格写
    *   系统学习：看 _[Speaking JavaScript](http://speakingjs.com/es5/)_ Chapter 1~3
*   ES6 及以后：
    *   快速上手：这些概念，过一遍代码例子，写代码时模仿下别人
        *   [Arrow Function](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
        *   [Spread Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator), [Rest Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters), [Destructuring Assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
        *   [Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes): 注意 JS 的 class 本质上还是基于原型链的 Object 结构演化而来，所以跟 Python / C++ 的 class 很不一样（比如没有类的静态变量）
        *   Modules: 参考我写的 `JavaScript Module System.pdf`
    *   系统学习：
        *   [ES6 In Depth Articles](https://hacks.mozilla.org/category/es6-in-depth/)
        *   [You Don't Know JavaScript](https://github.com/getify/You-Dont-Know-JS)
        *   [Modules](http://exploringjs.com/es6/ch_modules.html)

### 包管理器

前端构建过程主要用到的工具都是 Node.js 编写的。因此你需要知道怎样管理 Node.js 包。

*   快速上手：Windows 环境下安装部署 npm 及 Yarn
    *   安装 Node.js 及 npm （[下载链接](https://nodejs.org/en/download/)）
    *   安装 Yarn（[下载链接](https://yarnpkg.com/lang/en/docs/install/)）
    *   命令行运行 `npm config set registry https://registry.npm.taobao.org`，设置淘宝提供的镜像加速访问
    *   检查 `HTTP_PROXY` `HTTPS_PROXY` 环境变量设了没
    *   掌握这些命令：
        *   往当前项目安装一个包：`yarn add <package_name>`
        *   安装当前项目指定的依赖：`yarn install`
        *   `package.json` 和 `yarn.lock` 都要提交到 SVN
*   系统学习：查看 npm 及 Yarn 的文档，理解 Yarn 解决了什么问题

### 转译器

JS 社区标准的转译器叫 Babel，它将高版本的 JS 代码（比如 ES6），转译成低版本的 JS 代码（比如 ES5），以在仍不支持 ES6 的运行环境上（比如老版本的浏览器）运行 JS 代码。

*   快速上手：不用管它，React 的构建工具帮你搞定了
*   系统学习：看看它的文档

### 构建、打包工具

现在最流行的是 webpack。它很复杂。

*   快速上手：
    *   知道 `import 'file.css'` 是它的 [css-loader](https://github.com/webpack-contrib/css-loader) 在起作用，并且 `import` 进来的 CSS / JS 文件最终会被 webpack 打包成一个大的 CSS / JS 文件
    *   其他的不用管它，React 的构建工具帮你搞定了
*   系统学习：看看它的文档

### 开发工具

用 JetBrains 家的 [WebStorm](https://www.jetbrains.com/webstorm/)。

## 学习 React 本身

学习 React 最重要的点在于，理解：

*   JSX
*   组件化及单向数据流
*   Virtual Dom
*   快速上手：
    *   学 _[Fullstack React](https://www.fullstackreact.com/)_ 前 4 章
    *   通读 [官方文档](https://facebook.github.io/react/docs/hello-world.html) 中的 Quick Start 部分
    *   了解脚手架 [create-react-app](https://github.com/facebookincubator/create-react-app) 的基础用法
        *   `npm start`
        *   `npm run build`
*   系统学习：在快速上手的基础上，
    *   通读 [官方文档](https://facebook.github.io/react/docs/hello-world.html) 中的 Advanced Guides 部分
    *   学点源码，推荐 [SoundRedux](https://github.com/andrewngu/sound-redux)
    *   了解 Flux / Redux / MobX 等
*   深入了解：
    *   主流框架间的比较：
        *   [Comparison with Other Frameworks - Vue.js](https://vuejs.org/v2/guide/comparison.html)
        *   [和 Vue.js 框架的作者聊聊前端框架开发背后的故事](http://teahour.fm/2015/08/16/vuejs-creator-evan-you.html)

一些关键的难理解的点：

*   与 Vue.js / Angular 不同，React 没有双向绑定（即 DOM 元素与其数据绑定），而是使用单向数据流。数据往下流通过 props，数据向上浮通过事件
*   JSX 里面可以嵌套 JS 代码。不要把 JSX 理解成一种模板语言，它就是一种可以转译成 JS 的、用来简化书写的代码

## 学习 Ant Design

用 Ant Design 就跟你之前用 Bootstrap 没什么区别：

1.  参考 [这里](https://ant.design/docs/react/use-with-create-react-app-cn) 看看怎样把它加到项目来（很可能我已经帮你加好了）
2.  把它的组件全部 [过一遍](https://ant.design/docs/react/introduce-cn)，大概知道有什么组件可以用
3.  懂它的布局系统（24 列系统）
4.  想想你的场景适合什么组件，去相应的地方拷代码用

## 常见问题和好的实践

### JS 写着很不爽啊

JS 太垃圾不方便，但是有个非常好用的 [Lodash](https://lodash.com/docs/) 库，提供了很多方便的函数，写起来可以很像 Python。如果你也不知道怎么用 Lodash，那么你在 Google 代码例子时，加上 lodash 关键词，比如 "lodash iterate object"。

在代码里使用 Lodash：

1.  `yarn add lodash`
2.  `import _ from 'lodash'`

### 如何找合适的第三方库？

1.  参考 Ant Design 文档中的 [社区精选组件](https://ant.design/docs/react/recommendation-cn)，看看有无适合你的库
2.  Google 相关的关键词，看看相应的库 GitHub 星星数等
3.  [Awesome React](https://github.com/enaqx/awesome-react) 里找找

### 去哪查文档？

JS, React, Lodash 都不熟，经常要看文档怎么办？[DevDocs](http://devdocs.io/) 上这几个文档都有，Enable 之后将内容下载到离线使用，查阅起来就很方便。
