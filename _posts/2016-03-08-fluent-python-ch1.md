---
layout: post
title: "Fluent Python: Chapter 1"
tags: [Python, Reading]
---

![Fluent Python Cover]({{ site.url }}/images/2016/03/fluent-python.png)

在 Goodreads 闲逛时看到有一本去年出版的 Python 书—— _Fluent Python_ 很受好评，刚好 Google Play Books 上面也有，于是下载了样章下来看。昨天看完第一章，感觉很赞，值得安利一下。

这本书主要面向的是有经验的 Python 程序员，主要强调 Python 里面比较独特、与其他流行语言不一样的语言特性。

这本书的特点是写得很深入浅出，代码例子很简洁容易跟上，又能够清楚地讲解内容。而且在书里面穿插了很多相关的背景故事和文化，让你更了解某些语言特性是基于什么考虑设计出来的，读起来就像是有个高手在跟你聊天一样。同时里面有很多延伸阅读的内容，作者把他觉得相关的好书、网页文章甚至是 StackOverflow 回答推荐了出来，很容易找到方向做更深入的研究。

BTW，在 Google Play Books 的安卓客户端上看这本书，体验也很棒，而且价格只有 $24，其他渠道 (O‘Reilly Store,  Amazon) 要 $40+。如果看完样章觉得不错，买下来继续看。﻿

下面整理了第一章的读书笔记。

<!--more-->

Chapter 1. The Python Data Model
================================

这一章讲了一些 Python 的特殊方法（`__getitem__`, `__len__` 等），以及它们背后的设计意图。

> You can think of the data model as a description of Python as a framework.

这句话实在说得太 **贴切** 了。

> It formalizes the interfaces of the building blocks of the language itself, such as sequences, iterators, functions, classes, context managers, and so on.

Python 的特殊函数是一种函数语言本身的接口。可以用来构建序列、迭代器、函数等等。

## 一副 Pythonic 的卡牌

书中实现了一副卡牌的 Python 类。并且通过实现 `__len__`, `__getitem__` 得到了获取长度、切片操作等能力。还可以使用标准库 `random.choice()` 的能力，从这副卡牌中随机抽选一张，不用重复造轮子。

甚至 `reversed`, `in` 操作符也被可以自动支持。Python framework 的威力使得你实现了特殊函数后，就能很好的配置上 Python 的各种语法和标准库。

## 特殊函数是怎么使用的

特殊函数很少会被直接调用。唯一一个会被经常调用的是父类的 `__init__` 函数。

举 `__len__` 为例，你不需要主动调用这个函数： `a.__len__()`，你只需要使用 `len(a)`。对于一些内置类型，比如 `list`, `str`, `bytearray`，调用 `len()` 时并不会有函数调用发生，Python 会直接返回这些内置类型的 C 结构中的 `ob_size` 字段。

不推荐在自己的代码里面写这类特殊函数 `__foo__`，因为他们有可能在以后被语言本身使用。

## 数学运算的相关函数

实现 `__abs__`, `__add__` 和 `__mul__`，可以有一种 C++ 里面重载运算符的效果。

## 字符串表示

`__repr__` vs. `__str__`:

1. `__repr__` 是用来调试或者打 log 的，需要反映这个结构的内部组成
2. `__str__` 是给用户看的、更友好的字符串表示

有个好的实践是，`__repr__` 的输出就像初始化这个类的实例一样，如：

{% highlight python %}
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Vector(%r, %r)' % (self.x, self.y)

>>> a = Vector(3, 4)
>>> print(repr(a))
Vector(3, 4)
{% endhighlight %}

同时留意下 `__repr__` 函数中的 `%r`，这是必要的，可以用来区分 '3'(字符串) 和 3 (数字)。

## 文化

### `__getitem__` 怎样发音？

有些人这样讲：under-under-getitem，作者推荐读作 dunder。书里面有个故事讲述了来源。

### "Magic Method" vs "Special Method"

有些别的语言也有类似双下划线函数这样的实现，他们有的把这种函数叫做 "Magic Method"。作者觉得恰恰相反，这些函数不是魔幻的，他们是实实在在地提供了一套 metoobject protocol (不知如何翻译) 给开发者利用的。

然后作者黑了把 JavaScript，认为它设计得很挫，连把 object 里面的某个属性设成只读都实现不了（直到 2009 年的 ECMAScript 5.1 规范），它认为 JavaScript 的很多设计才是真正的 "magic"。

### Metoobject, Aspect-oriented programming

这部分也是延伸阅读，不是太懂，如果有精力有兴趣再了解下。
