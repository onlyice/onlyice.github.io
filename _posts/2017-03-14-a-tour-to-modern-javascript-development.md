---
layout: post
title: "A Tour to Modern JavaScript Development"
tags: JavaScript
---

![JS World]({{ image_cdn }}/images/2017/03/js-world.png)

这篇文章描述一个后端程序员看到的难以理解的前端世界。我对前端理解不深，这篇文章大多数内容是吐槽向。如果你对现代前端的开发没有了解，可以先看看 HackerNoon 上的这篇 [文章][js-2017]。如果你想看更猛烈的吐槽，参考这篇「[在 2016 年学 JavaScript 是一种什么样的体验？](https://zhuanlan.zhihu.com/p/22782487)」。

[js-2017]: https://hackernoon.com/a-map-to-modern-javascript-development-2017-16d9eb86309c#.mldzrt1gn

<!--more-->

## ES6 And Beyond

ES6 给 JS 带来了很多很棒的语言特性（有很多 Python 的影子），但是它同时又延续了糟糕的品味。很多新特性设计得太过灵活，看得人眼花缭乱，看起来难懂用起来更是难用。本来很多 JS 代码写得就一团糟，ES6 一出来又给了大家合理的理由可以写得更糟了。

比如这个 [Destructuring assignment][destructuring-assignment] + [Rest parameters][rest-parameters] + [Spread syntax][spread-syntax] combo，实在是太太太灵活了，我不知道有几个人吃得消。destructuring 这个理念挺好，可是这里还能设默认值是几个意思：

{% highlight javascript %}
var fun = ({a, b} = {a: 1, b: 3}) => a+b;
{% endhighlight %}

Spread 操作符即可以展开 array 也可以展开 Object，感觉做过了。JS 的 Object 本来就让人感觉各种奇怪，而且在这种用原型链的复杂数据结构里，搞一个 spread 操作感觉真心不直观。

再吐糟下奇怪的 modules system。我理解 JS 需要有一个内建的模块系统，但是这个模块系统出来的也太不及时了，AMD 和 CommonJS 早在各自领域发芽生根了多年，结果 ES6 Modules 珊珊来迟，简直怕不给 JS 世界增加多一点混乱。像 Node.js 已经用上 CommonJS，要再支持一把 ES6 Modules 估计不是易事。大量的第三方 JS 库也面临同样的问题，在支持 AMD 和 CommonJS 的基础上又要再支持另外一个模块系统，不知道开发者有何想法。更奇葩的是，作为 JS 的最大运行环境——浏览器，都没有实现 ES6 模块系统（很可能短时间内都不会支持），这种语言规范和实现太不贴近的现象实在是很奇怪。

再然后，不得不吐槽一下 webpack 对 `import` 语法的滥用。本来 JS 语言特性就乱成一团，结果 webpack 一掺和更是乱得不行，强行让 import 多了很多功能。import 不仅可以加载 JS 模块，还能加载 CSS 样式，加载 SVG 文件，甚至还能加载 JSON 文件！万物皆可 import 吗。

{% highlight javascript %}
import React from 'react';
import _ from 'lodash';
import Highlighter from 'react-highlight-words';

import './App.css';
import data from './data.json';
{% endhighlight %}

JS 在变得越来越灵活，需要很多 best practice 才能用好，在这点上真的不如 Python。

## React

React 是一个让我比较惊艳的框架。它在面对前端开发这个复杂的问题时，通过组件化、单向数据流等核心理念比较好地解决了这种问题，而且看起来 Facebook 在执行上也做得不错。但是 React 也有着类似 JS 的问题，它过于灵活，使得你又需要了解它的设计理念和最佳实践，才能用好它。而且你想真正顺利地使用它，又得深入了解它的运作原理。这使得入门成本被拔高。

JSX 真是一个奇怪的设计。它看起来像 XML 但是又不是 XML 也不遵循 XML 的规则。它看起来像模板语言，可是设计者们又坚持说它不是模板语言，只是 JS 的扩展。用 JSX 时就像写一个扭曲的 HTML 和 JS 语句的揉合体。

Flux 似乎给出了一层抽象，以解决复杂的页面交互和数据变化等等一系列问题。Redux 没深入看，感觉对于小应用来说就是 overkill，而我还没有能力写大应用。

React 写起来还是没有当时 Vue.js 那种简单粗暴的快感，不过感觉 React 的生态和理念还是更胜一筹。

## Async I/O

看不懂 Promise 解决了什么问题。看不懂 Fetch API 比 AJAX 好在哪里。看不懂 `async` `await` 关键字想干啥。

## Packages / Assets Management

对于资源文件的管理，一开始大家都说 bower 好。然后不知道怎的大家都说 Bower 要死了。再然后大家又说要用 npm 来管理一切东西。

然后大家又说 npm 对项目依赖管理不好，要用 yarn。不知道你们什么时候再说 yarn 不好。

## WebStorm

在这么残酷混乱的世界中，要把 JS IDE 做好真不容易。点赞。

## VSCode

微软良心，发版本好快。

## webpack

我看不懂。但是感受到开发者很用心地在把事情搞复杂。CPU 就是来让几百个 loader 吃掉的。

## elm

[elm][] 似乎是个学术上很厉害的东西，据说 React 也借鉴了它所谓函数式的想法。我没有深入了解过它，但是看到有人说它开发体验很赞。

## GraphQL

[GraphQL][graph-ql] 的理念看起来很赞，但是超前的理念可能并不能具体落到实处，应该是个活不长的东西。

## Babel

文档写得真差。

## 结语

前端变化得实在太快了，不过同时也有很多新的理念出现。作为一个最流行的 UI 系统，搭配 Electron 等框架，能做的事情还是挺多的。而且现在 WebAssembly 似乎正在吸引更多的注意力，未来前端的能力可能会越来越强。我认为还是值得一学。

[destructuring-assignment]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
[rest-parameters]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
[spread-syntax]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator
[elm]: http://elm-lang.org/
[graph-ql]: http://graphql.org/
