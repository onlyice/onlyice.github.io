---
layout: post
title: "如何使你的文章格式美观优雅？"
image:
  background: 2019/04/lighthouse.jpg
tags: 
- 个人成长
---

<figure>
    <img src="{{ site.image_cdn }}/images/2020/03/designer-sketching-wireframes.jpg" />
    <figcaption>
      <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@craftedbygc?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Green Chameleon"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Green Chameleon</span></a>
    </figcaption>
</figure>

高质量的内容固然是最重要的，但是好的排版会让你的内容锦上添花，让读者有一个好的阅读体验，也能体现文章背后作者的专业和认真。

比如几个排版优秀的例子：

* [如何高效实践卡片式写作？ - 少数派](https://sspai.com/post/59109)
* [Python 工匠：高效操作文件的三个建议 - Piglei](https://www.zlovezl.cn/articles/three-tips-on-writing-file-related-codes/)

都做到的内容高质量以及排版上的美观大方。

本文会讲述一些简单易用的排版技术和常见错误。

<!--more-->

<style>
.content-box {
  border-radius: 3px;
  border: 2px solid grey;
  padding: 10px;
}
</style>

## 正确使用标点符号和空格

这是最基础也是最重要的。

在中文文章中应该使用中文的标点符号，在英文文章中则应该使用英文的：

* 正确：嗨！你知道嘛？今天前台的小妹跟我说「喵」了哎！
* 错误：嗨!你知道嘛?今天前台的小妹跟我说"喵"了哎!

避免使用不规范的标点格式，比如：

* 不要用三个感叹号（`！！！`）来表达强烈情绪，一个即可
* 不要用三个句号（`。。。`）来代替省略号（`……`）

中英文之间应该有一个半角空格：

* 正确：苹果公司在 2015 年 9 月 9 日发布了 iPhone 6s
* 错误：苹果公司在2015年9月9日发布了iPhone 6s

英文专有名词的大小写应该正确书写：

* 正确：iOS、macOS、Android
* 错误：ios、MacOS、android

这方面的有优秀的规范指引，照着做即可：

* [少数派写作排版指南](https://sspai.com/post/37815)
* [中文文案排版指北](https://github.com/mzlogin/chinese-copywriting-guidelines)
* [文案风格指南 - LeanCloud](https://open.leancloud.cn/copywriting-style-guide/)

同时特别推荐这份标点符号的指南，作者实在太用心了：

* [别再用「六个点」当省略号了，这些标点都有更规范的输入方式](https://sspai.com/post/45516)

## 正确使用列表

大部分的排版系统都会提供有序列表和无序列表。有序列表的语法一般是：

{% highlight markdown %}
1. 打开冰箱门
2. 把大象放进去
3. 把冰箱门关好
{% endhighlight %}

渲染出来的效果：

<div class="content-box">
  <ol>
    <li>打开冰箱门</li>
    <li>把大象放进去</li>
    <li>把冰箱门关好</li>
  </ol>
</div>

无序列表则是：

<div class="content-box">
  <ul>
    <li>语文</li>
    <li>数学</li>
    <li>英语</li>
  </ul>
</div>

如果你的列表项目之间有顺序关系，那么你应该用有序列表，否则应该用无序列表。

同时对于 Markdown，它可以支持列表项目的内容是多行的，只需要给它的行提供额外的缩进：

{% highlight markdown %}
1. 列表第一个项目

   第一个项目内的额外行。
2. 列表第二个项目
{% endhighlight %}

渲染出来的效果是：

<div class="content-box">
  <ol>
    <li>列表第一个项目<p>第一个项目内的额外行。</li>
    <li>列表第二个项目</li>
  </ol>
</div>

另外有序列表的标号，不一定是以 1 开头。在 Markdown 中你也可以以其他数字开头：

{% highlight markdown %}
10. 我是第十项
11. 我是第十一项
{% endhighlight %}

渲染出来的效果是：

<div class="content-box">
  <ol start="10">
    <li>我是第十项</li>
    <li>我是第十一项</li>
  </ol>
</div>

在这里经常看到的问题是，中英文标点混用，比如：

{% highlight markdown %}
1、
2、

一.
二.
{% endhighlight %}

这些都是不好的例子。如果你想利用英文排版中提供的有序列表，那么你应该用数字加英文句号（如 `1.`）。如果你想用中文序号，那么应该是中文数字加顿号（如 `一、`），但无法用上排版系统提供的缩进能力。

## 不要让链接直接显示出来

由于链接往往是一长串不太可读的英文字符（如 `https://vuepress.vuejs.org/zh/guide/markdown.html#%E7%9B%AE%E5%BD%95`），应该使用链接到文本的 [能力](https://vuepress.vuejs.org/zh/guide/markdown.html#%E7%9B%AE%E5%BD%95) 而不是直接放出链接本身。

## 要不要首行缩进？

对于数字阅读时代，首行缩进是无意义的。原因在于：

* 移动设备上一行显示的字数少，如果再有两个字的缩进，阅读效率低
* 大部分的排版系统都会在段落之间用额外的空白来分隔

## 图片标题

对于放图片的场景，经常需要有用一小段文本描述图片内容或者图片出处的需求。常见的错误是新开一个段落去描述。正确的做法是使用 `<figure>` 及 `<figurecaption>`。这两个标签是 HTML 标签，在 Markdown 中往往没有对应语法。不过 Markdown 中也可以直接插入 HTML 内容。比如：

{% highlight html %}
<figure>
    <img src="/images/2020/03/custom-container.png" />
    <figcaption>留意图中的红色背景灯泡部分</figcaption>
</figure>
{% endhighlight %}

## 善用表格

表格在做对比时非常好用，比如：

<figure>
    <img src="{{ site.image_cdn }}//images/2020/03/cloud-disk-comparison.png" />
    <figcaption>图片来自少数派 <a href="https://sspai.com/post/34613">帖子</a></figcaption>
</figure>

如果善用单元格合并，也可以做出表意清晰但相对复杂的表格。下面是一个我做读书笔记的例子：

<img src="{{ site.image_cdn }}//images/2020/03/read-note.png" />

## 脚注

对于不影响正文阅读的扩充性内容，用脚注是非常好的例子。在主流的 Markdown 语法中，脚注是受支持的[^1]，用 Markdown 书写起来是这样的：

{% highlight markdown %}
我是正文[^1]

[^1]: 我是脚注
{% endhighlight %}

[^1]: 我是脚注内容

## 自定义容器

有些文档工具会提供自定义容器的功能（如 [VuePress](https://vuepress.vuejs.org/zh/guide/markdown.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AE%B9%E5%99%A8)），用来表示相对独立、不太顺序文章内容流向的内容，如：

<figure>
    <img src="{{ site.image_cdn }}/images/2020/03/custom-container.png" />
    <figcaption>留意图中的红色背景灯泡部分</figcaption>
</figure>

但大多数文档工具并没有此功能。所以很多人会用块引用（blockquote）来表达此含义。块引用在 Markdown 中的语法是（`>`），如：

{% highlight markdown %}
Arch Linux 的官方仓库已经有 GoldenDict，直接安装即可。但是默认不带构词法文件，需要手动下载添加。官网的下载页有说明。

> 对于中文翻译成英文的，目前还没有找到太好的办法，基本上是靠 Google Translate 翻译后再逐词研究。后面有需求时再看看有什么好办法。
{% endhighlight %}

渲染成：

<div class="content-box">
  <p>Arch Linux 的官方仓库已经有 GoldenDict，直接安装即可。但是默认不带构词法文件，需要手动下载添加。官网的下载页有说明。</p>
  <p><blockquote>对于中文翻译成英文的，目前还没有找到太好的办法，基本上是靠 Google Translate 翻译后再逐词研究。后面有需求时再看看有什么好办法。</blockquote></p>
</div>

虽然从语义上讲块引用并不是用来表达则含义，但是从实践角度是可以授受的。

## 后记

作为互联网创意工作者的主力军，程序员、设计师和产品经理群体往往更关注内容排版质量，比如 [少数派](https://sspai.com/) 上的内容一般都有不错的排版质量，可以参考。

能看到这里的，想必都是愿意花时间精力打磨自己内容的人。希望这篇文章对你有所帮助，如有错漏也请指出😜
