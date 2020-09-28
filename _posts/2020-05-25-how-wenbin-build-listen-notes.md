---
layout: post
title: "Listen Notes：只有一个员工的盈利产品"
image:
  background: background/witewall_3.png
tags:
- 个人成长
---

[Listen Notes](https://www.listennotes.com/) 是一个播客搜索引擎。它提供的功能有：

- 一个可以检索的播客数据库。它给节目生成了主题、分类、人物、语言、区域等信息
- 稍后再听的 Listen Later 功能；剪辑节目中某段内容的 Listen Clips 功能；监控某一关键词在播客节目中出现的 Listen Alert
- 一批可付费购买的数据内容，如根据语音生成的 transcript，按条件搜索出来的数据集等等

在我惊讶的地方在于，这个功能这么丰富的产品，它只有一个员工兼老板 Wenbin Fang，开发、产品、运维、运营一条龙全包。Wenbin 在 Listen Notes 的 [博客](https://www.listennotes.com/blog/) 上写了很多文章，涵盖了他在技术、产品、运营等等方面的思考，让我觉得收获很大。

<!--more-->

这里有两篇技术相关的博客文章：

- [Good enough engineering to start an Internet company](https://www.listennotes.com/blog/good-enough-engineering-to-start-an-internet-27/)，March 11, 2019
- [The boring technology behind a one-person Internet company](https://www.listennotes.com/blog/the-boring-technology-behind-a-one-person-23/)，Jan. 24, 2018

Wenbin 对待技术的态度非常务实。他在技术选型上，一直强调的是「boring technology」「good enough engineering」，不需要追新，**成熟、自己熟练、好用且可以解决问题的技术** 就可以了。他在技术决策上非常 **务实**。比如：

- 不使用 Docker / Kubernetes / serverless。作者觉得中型及以上的公司才有必要去考虑这些技术
- 不做纯粹的前后端分离，网页中有一半是后端生成（Django Template），有一半是前端渲染（React）
- 使用的都是发展多年的成熟技术，比如 PostgreSQL、Ansible 等。Wenbin 说这些技术可以让他晚上睡得好一些

这种务实不仅体现在技术决策上，也体现在 Listen Notes 的 [广告投放页](https://www.listennotes.com/ads/) 上：

<img src="{{ site.image_cdn }}/images/2020/05/listen-notes-ads.png" />

广告主关心的核心数据指标、用户画像、费用等等，都直观清晰地列了出来，没有遮掩。而相较之下，市面上大部分产品往往不愿意公开数据指标和广告费用。

Wenbin 在旧金山租了一间 WeWork 办公室给自己办公。租金不便宜，但是他乐意花钱在工作效率上，而不是在家或者咖啡厅，花更多时间完成同样的工作。

他也提到他的创业哲学：

> Most of time, the biggest obstacle of building & shipping things is over thinking. What if this, what if that. Boy, you are not important at all. Everyone is busy in their own life. No one cares about you and the things you build, until you prove that you are worth other people’s attention. Even you screw up the initial product launch, few people will notice. Think big, start small, act fast. It’s absolutely okay to use the boring technology and start something simple (even ugly), as long as you actually solve problems.
>
> Your overthinking is my opportunity.

我简单翻译一下：

> 大部分情况下，构建和发布一个产品最大的障碍是 **过度思考**。今天想着这样，明天想着那样。事实上，你的想法一点都不重要。每个人都忙于过自己的生活，没有人关心你做了些什么，直到你做的东西足够好到引起别人的注意。所以即使你一开始发布的产品不怎样，也很少人会注意到。要有宏大的构思，从小处着手，并且快速去实现。使用无聊的老技术来开始做你简单（又丑陋）的产品是绝对 OK 的，只要你的产品能实际上解决问题。
>
> 你的过度思考就是我的机会。

最后，再看看 Wenbin 的 Twitter 和 LinkedIn。我不得不说是个很有个性的人……比如 Twitter 上写的地理位置：

<img src="{{ site.image_cdn }}/images/2020/05/wenbin-twitter.png" />

LinkedIn 上这个个人评价，好一个 useful……而且读博还辍学了：

<img src="{{ site.image_cdn }}/images/2020/05/wenbin-linkedin-1.png" />

再看看语言能力。C / x86 Assembly 是什么鬼……

<img src="{{ site.image_cdn }}/images/2020/05/wenbin-linkedin-2.png" />
