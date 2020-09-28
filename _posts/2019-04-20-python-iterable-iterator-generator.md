---
layout: post
title: 一篇文章看懂 Python iterable, iterator 和 generator
description: "iterable, iterator 和 generator 究竟是什么，又有什么关系？"
image:
  background: 2019/04/lighthouse.jpg
tags: 
- 编程技术
---

Python 中的 iterable, iterator 以及 generator，一直是非常亲密但是难以区分的概念。nvie 有一个很好的 [帖子][nvie-post] 阐述了它们之间的关系，但是内容偏向于概括和总结，对于新手来说仍然难以理解。[Fluent Python][fluent-python-oreilly] 的第 14 章也有非常好的演绎，但是我认为它对「为什么要有这种语言特性」缺乏阐释。我试图从演变的角度，总结这些概念的来源和演化，以得到一个符合逻辑和容易理解的版本。

<!--more-->

## Simple Loop

几乎每一个 Python 入门教程，都会用类似下面的代码来讲述最简单的 `for` 循环：

{% highlight python %}
>>> l = [2, 1, 3]
>>> for i in l:
...     print(i)
2
1
3
{% endhighlight %}

在 Python 中，执行 `l[i]` 实际上是调用了 `l` 的 `__getitem__` [函数][get-item-docs]。`list` 类型中会实现了这个函数，用来返回某个 index 下的元素。而早期 Python 的实现上利用了这个操作。上面的 `for` 循环实际上是从 `l[0]` 开始取元素，等价于这段代码：

{% highlight python %}
i = 0
while True:
    try:
        print(l[i])  # 亦可是 print(l.__getitem__(i))
        i += 1
    except IndexError:
        break
{% endhighlight %}

Python 内置的大多数容器类型（`list`, `tuple`, `set`, `dict`）都支持 `__getitem__`[^1]，因此它们都可以用在 `for .. in` 循环中。如果你有个自定义类型也想用在循环中，你需要在类里面实现一个 `__getitem__` 函数，Python 就会识别到并使用它。Fluent Python 一书提供了一个[例子][sentence-iter-example]。

## Lazy Evaluation

在上面的代码例子中，`l` 的值是全部被加载到内存中，再在循环中被一个一个取出来的。设想这样一个场景，你要从数据库中查询出一千万条数据做处理，

* 如果全部加载到内存，可能会将内存撑满
* 在处理第一条数据前，需要等待大量时间从数据库中取出这些数据
* 一些特殊的场景下，你可能并不需要对全部的数据做处理，比如处理到第五百万条数据时即可以结束

前辈们提出了惰性求值（Lazy Evaluation）来解决这个问题。有些地方也叫它「延迟加载」「懒加载」等。它的基本理念是「按需加载」，在上面的例子中，可以将取数据过程变成一页页取，比如先取 100 条数据进行处理，处理完后再取下一个 100 条，直至全部取完。

## The Iterator Protocol

Python 为了在语言层面支持 lazy evaluation，给出了 iterator 协议。如果一个类：

* 实现了 `__next__` 函数，并且：
  * 每次调用该函数，都可以返回一个新的数据
  * 没有新的数据时，调用它抛出 `StopIteration` 异常（当然如果序列是无限长，那么可以不抛）
* 实现了 `__iter__` 函数返回 `self`，下一节中描述

那么这个类即支持 iterator 协议。于是「按需加载」，即可以通过每次 `__next__` 被调用时去实现。Python 的内置函数 `next(iterator)` 实际上是调用 `iterator.__next__`。下面的例子给出一个 iterator 的实现，用来按需地计算出下一个斐波那契数：

{% highlight python %}
>>> class FibonacciIterator:
...     def __init__(self, maximum):
...         # 为了简单，将初始值设为 1, 2 而不是 0, 1
...         self.a, self.b = 1, 2
...         self.maximum = maximum
...     def __next__(self):
...         fib = self.a
...         if fib > self.maximum:
...             raise StopIteration
...         self.a, self.b = self.b, self.a + self.b
...         return fib
        
>>> f = FibonacciIterator(5)
>>> next(f)
1
>>> next(f)
2
>>> next(f)
3
>>> next(f)
5
>>> next(f)
# StopIteration occured
{% endhighlight %}

## Enhanced iterable

上文中的 `FibonacciIterator` 已经实现了按需加载，那可以直接将它用在 `for` 循环中吗？试试：

{% highlight python %}
>>> for i in FibonacciIterator(5):
...     print(i)
... 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'FibonacciIterator' object is not iterable
{% endhighlight %}

可以看到有 is not iterable 的报错。按上一节的描述，早期的 Python 仅在一个类有 `__getitem__` 函数时 Python 才将它当成 iterable，同时为了配合新的 iterator 的机制，Python 在 2.2 版本中将 `__iter__` 协议加入了进来：

1. 一个类如果实现了 `__iter__` 并返回一个 iterator，那么它是 iterable 的
2. 如果没有实现 `__iter__`，但是有 `__getitem__`，那么它仍然是 iterable 的

回想上文对于 iterator 的定义，它是实现了 `__iter__` 的，所以可以用在 `for` 循环中：

{% highlight python %}
>>> class FibonacciIterator:
...     # 其他函数实现省略，见上文
...     def __iter__(self):
...         return self
>>> for i in FibonacciIterator(5):
...     print(i)
1
2
3
5
{% endhighlight %}

## Generator Function

上面虽然花了挺大篇幅讲述 iterator 的机制，但是事实上 Python 中以 `__next__` 方式来实现 iterator 的并不多。Python 在 2.2 版本中支持了 iterator，但是也同时给出了另外一种更灵活方便，也更重要的机制 —— generator。

识别 generator 的标志在 `yield` 关键字。上文的斐波那契数列，用 generator 来实现是：

{% highlight python %}
>>> def fib(maximum):
...     a, b = 1, 2
...     while a <= maximum:
...         yield a
...         a, b = b, a+b
...
>>> for i in fib(5):
...     print(i)
1
2
3
5
{% endhighlight %}

上面的 `fib()` 虽然也是用 `def` 定义，但是它的函数体中有 `yield` 关键字，因此它不是个普通函数，而是个 generator function。它返回的是一个 generator object，即 `fib(5)` 处。generator 是一种在语言层面被支持的 iterator，它的规则是：

* `next()` 调用一个 generator 时，Python 会执行函数体到下一个 `yield` 语句，并将 `yield` 后的值作用 `next()` 的返回值；然后该函数的执行暂停，直至下一次 `next()` 调用时，继续执行到下一个 `yield`
* 当整个函数体被执行完毕时，抛出 `StopIteration` 异常

这套规则清晰直观，可以将它套用在上面代码中验证一下。`yield` 及 generator 是非常重要的机制，不仅仅在于它比 iterator 更简单直观，而在于它同时引入了一种控制语言运行的机制。对于普通函数，一旦执行则必须从函数头执行到函数尾，之后才把控制权交给调用方；但是对于 generator function，你可以只执行一小段代码，即把控制权交回调用方（`yield` 时）。这种机制也对后面提出 coroutine 及 asyncio 中起到了重要的作用。

## Generator Expression

试试运行下面的代码：

{% highlight python %}
>>> sum([i**2 for i in range(11)])
385
>>> sum((i**2 for i in range(11)))
385
>>> sum(i**2 for i in range(11))
385
{% endhighlight %}

后两种写法，跟第一种有什么区别呢？后两种即是 generator expression，是一种方便生成 generator 的语法糖，形式上是用括号包裹的 list comprehension。背后的理念大概是这样：list comprehension 是用来生成元素的，generator 也是用来生成元素的，那为什么不提供一种类似 list comprehension 语法的 expression 来表示 generator 呢？它跟下面的代码是等价的：

{% highlight python %}
def gen():
    for i in range(11):
        yield i**2

sum(gen())
{% endhighlight %}

至于第三种写法为啥不用括号包起来，是 Python 为了可读性故意设计的，如果作为唯一的函数参数使用，则可以省略。

## 总结

定义上：

* 实现了 `__iter__` 或 `__getitem__`，并满足一定规则的类型是 iterable 的，它的实例是个 iterable
* 实现了 `__next__` 并满足一定规则的类型，它是一种 iterator，它的实例是个 iterator
* 在函数体中使用了 `yield` 的函数是 generator function；使用了括号包裹的类 list comprehension 是 generator expression。它们都会产生 generator

语言机制上：

* `for .. in` 循环所消费的对象，需要是个 iterable

它们之间的关联：

* 容器类型（`list`, `dict`, etc.）大部分是 iterable；`dict` 有多个不同函数生成不同用途的 iterator
* iterator 大部分情况下是个 iterable
* generator 是个 iterator，同时是个 iterable

nvie [帖子][nvie-post] 中的图片，形象地表达了上面的内容：

![relationships]({{ site.image_cdn }}/images/2019/04/relationships.png)


[nvie-post]: https://nvie.com/posts/iterators-vs-generators/
[fluent-python-oreilly]: http://shop.oreilly.com/product/0636920032519.do
[get-item-docs]: https://docs.python.org/3/reference/datamodel.html#object.__getitem__
[sentence-iter-example]: https://github.com/fluentpython/example-code/blob/master/14-it-generator/sentence_iter.py

[^1]: 但是为了性能，实际上他们会用 C 来实现此函数，所以你在源码中找不到一个 `list.__getitem__` 函数。
