---
layout: post
title: 今时今日，学计算机应该看什么书？
image:
  background: 2019/05/stellar-collision-bg.jpg
tags: 
- Computer Science
---

我读大学是在 2009 - 2013 年，那个时候被认为最好的学 CS 的方法是读经典的书，比如 TCP / IP [三][tcp-1] [部][tcp-2] [头][tcp-3]、「[算法导论][introduction-to-algorithm]」、「[Unix 环境高级编程][apue]」等等。但是这些书有些写自 1980s - 1990s 甚至更早，而且大部分都很干，很难读，缺乏趣味性。我基本没有完整看过这些书，都是浅尝辄止。

10 年过去后，技术圈有了更多好书出现。他们大多数讲得深入浅出、插图精美，趣味性远强于经典著作，同时又简单易懂，学起来轻松不费力。我觉得现在的 CS 学生没有必要看这些大部头了入门了。下面推荐一些我觉得好的：

## 算法

### [Algorithms by Jeff Erickson][algorithm-jeff]

<img src="{{ site.image_cdn }}/images/2019/05/algorithm-by-jeff-cover.png" alt="algorithm-jeff" width="250"/>

当你有这些精美的插图来解释算法，甚至连习题都是这么精心准备时，有什么理由去看干巴巴的伪代码和公式呢？

![algorithm-jeff-1]({{ site.image_cdn }}/images/2019/05/algorithm-jeff-1.png)

![algorithm-jeff-2]({{ site.image_cdn }}/images/2019/05/algorithm-jeff-2.png)

这本书的一个优点（或者说缺点），是它不带有具体编程实现，只有伪代码。意味着你可以边学边用你熟悉的编程语言进行实现。

### [Algorithms 4th Edition][algorithm-princeton]

<img src="{{ site.image_cdn }}/images/2019/05/algorithm-princeton-cover.png" alt="algorithm-princeton" width="250"/>

这本来自普林斯顿大学的算法书，制作非常精良，不仅图片精致好看，同时也带了 Java 代码实现。

![algorithm-princeton]({{ site.image_cdn }}/images/2019/05/algorithm-princeton.png)

## 操作系统

### [Operating Systems: Three Easy Pieces][ostep]

![ostep]({{ site.image_cdn }}/images/2019/05/ostep.jpg)

围绕 virtualization、concurrency 以及 persistence 来讲述操作系统中的基本原理。这本书讲得非常生动，每节内容前都用教授和学生的对话带出问题，再具体分析问题、讲述概念。这个过程经常会穿插可以实际运行验证的代码。最终会有回顾和作业巩固你的知识。

北美近年来的教科书，似乎开始慢慢改变以前动不动就大而全的习惯，越来越易懂易学了。

## 其他

### [High Performance Browser Networking][hpbn]

<img src="{{ site.image_cdn }}/images/2019/05/hpbn-cover.jpg" alt="hpbn-cover" width="250"/>

浏览下它讲 [TLS][hpbn-tls] 的这一章，你大概就知道为什么我会推荐它。

### [Designing Data-Intensive Applications][ddia]

<img src="{{ site.image_cdn }}/images/2019/05/data-intensive-cover.png" alt="data-intensive-cover" width="250"/>

这本书可以说是 2018 年到现在 O'Reilly 最火的一本书了。真正做到了 dig deeper than buzzwords，带你走一遍常见的数据系统，对比各自优劣和适用场景。

### [Fluent Python][fluent-python]

<img src="{{ site.image_cdn }}/images/2019/05/fluent-python.jpg" alt="fluent-python" width="250"/>

我最喜爱的 Python 书。看看我在这里写的 [笔记][fluent-python-notes]，你可以看出我花了多少时间在这上面。Time well spent.

[tcp-1]: https://book.douban.com/subject/1088054/
[tcp-2]: https://book.douban.com/subject/1087767/
[tcp-3]: https://book.douban.com/subject/1058634/
[introduction-to-algorithm]: https://book.douban.com/subject/20432061/
[apue]: https://book.douban.com/subject/25900403/
[algorithm-jeff]: http://jeffe.cs.illinois.edu/teaching/algorithms/
[algorithm-princeton]: https://algs4.cs.princeton.edu/home/
[ostep]: http://pages.cs.wisc.edu/~remzi/OSTEP/
[hpbn]: https://hpbn.co/
[hpbn-tls]: https://hpbn.co/transport-layer-security-tls/
[ddia]: https://dataintensive.net/
[fluent-python]: http://shop.oreilly.com/product/0636920032519.do
[fluent-python-notes]: https://wiki.zhiheng.io/#Fluent%20Python
