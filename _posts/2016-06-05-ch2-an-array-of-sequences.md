---
layout: post
title: "Chapter 2: An Array of Sequences"
tags: [FluentPython, Python]
---

{% include serials/fluent-python.md %}

这一章主要讲 Python 中的序列类型。

<!--more-->

Python 之父 Guido 在搞 Python 之前，对一门叫 ABC 的语言投入了很多。Python 的很多被视为 Pythonic 的特性跟 ABC 有关联。

按序列中存放的数据类型，可以把序列分为两类：

* Container sequences: 序列中的存的元素是引用，指向另外一个 Object。比如 `list`, `tuple`
* Flat sequences: 序列中的元素不是引用，是同一种数据类型，比如 `str`, `bytes`, `array.array`

又可以按是否可变分为两种类型，mutable 与 imutable。

## List comprehensions 与 map/filter

作者认为 list comprehensions 也可以用内置函数 `map`/`filter` 来实现，但是后者可读性差，而且性能并不比 list comprehensions 高很多。

## Tuple 并不只是不可变的 list

Python 的 tuple 容易被认为只是不可变的 list，但是它有几个设计上的意图

### Tuple 作为纪录（record）

比如下面的例子，等号右边的 tuple 就是一条纪录，保存了东京相关的一些信息：

{% highlight python %}
city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
{% endhighlight %}

tuple 作为纪录时，纪录中的 Field 名称是缺失的，可以用 `collection.nametuple` 来创建带名称的 tuple。`collection.nametuple` 以比较小的性能开销实现了 tuple 带名称，同时还有 `asdict()` 函数，将 tuple 转为 `OrderedDict`。

### Tuple Unpacking

tuple 经常配合 `*` 做 unpacking。各种花式用法：

{% highlight python %}
>>> x, y = (10, 24)
>>> x
10

>>> t = (20, 8)
>>> divmod(*t)
(2, 4)

>>> a, b, *rest = range(5)
>>> rest
[2, 3, 4]
>>> a, b, *rest = range(3)
>>> rest
[3]

>>> metro_area = ('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
>>> name, cc, pop, (latitude, longitude) = metro_area
>>> latitude
35.689722
{% endhighlight %}

`_` 经常被用作一个 dummy variable，用来接收 tuple unpacking 中的无用部分。一般来讲用这个变量是合乎惯例（convention）的，但是在一些 I18N 的场景，这个变量被用来作特殊的用途。Python 标准库中的 `gettext` 就是一个例子。

tuple 的接口与 list 基本类似，只是比 list 少了一些修改自身的函数，比如 `append`, `pop`, `sort` 之类。

## Slice

### Slice Object

`[start:stop:step]` 其实是个 slice 对象。调用 `seq[start:stop:step]` 时，其实是调用 `seq.__getitem__(slice(start, stop, step))`。可以将 slice 命名以提升可读性：

{% highlight python %}
events = """2015-09-10 Teacher's Day
2015-10-01 National Day"""

DATE = slice(0, 10)
EVENT = slice(11, None)
for event in events.split('\n'):
    date, event = event[DATE], event[EVENT]
{% endhighlight %}

### Slice 高级形式

Python 还提供了其实几种形式的 slice，但是他们比较少出现在标准库中，大多数是给第三方数学库使用。

如多维 slice：

`seq[start1:stop1, start2:stop2]`

实际上是调用 `seq.__getitem__((slice(start1, stop1), slice(start2, stop2))`。

省略号表示法 `...`，如 `seq[1, ...]`。它跟一个 `Ellipsis` 类有关，暂不深究。

### Slice 可以用来对序列做修改

{% highlight python %}
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
{% endhighlight %}

## `+` 与 `*` 操作符应用在序列上

这些操作比较直观易懂：

{% highlight python %}
>>> l = [1]
>>> l * 5
[1, 1, 1, 1, 1]
>>> l + [2]
[1, 2]
>>> 'ab' + 'cd'
'abcd'
>>> 'ab' * 5
'ababababab'
{% endhighlight %}

但是留意一下，如果序列中的元素是引用类型，那么 `*` 操作符复制的时候也是引用。这种情况下，复制出来的元素，它们其实指向同一个 Object：

{% highlight python %}
>>> l = [[1]] * 3
>>> l
[[1], [1], [1]]
>>> l[0].append(2)
>>> l
[[1, 2], [1, 2], [1, 2]]
{% endhighlight %}

### `+`, `*` 与 `+=`, `*=`

对于序列类型，一般来讲，`+` / `*` 操作符返回的结果是个新的Object，即不会修改操作符左右两个操作数；而 `+=` / `*=` 会修改左侧被赋值的操作数。这是一个 Python 中的惯例（convention）。

这几个操作符由一些特殊函数实现：

* `+` => `__add__`
* `*` => `__mul__`
* `+=` => `__iadd__` (in-place addition)
* `*=` => `__imul__` (in-place multiple)

## `list.sort()` 与内置函数 `sorted`

这里又引入了一个惯例（convention），内置函数的 `sorted`, `reverse` 等等，不会对作为参数的序列做修改。而序列本身的 `sort`, `reverse` 函数，会对序列做修改。

## `bisect`

`bisect` 库做的是，对于一个已经排好序的序列，使用二分查找给出一个新元素插入该序列时的位置（`bisect.bisect`），或者实际地将这个元素插入（`bisect.insort`）。应用场景看下面的例子。

### 一个按区间打分的例子

{% highlight python %}
>>> def grade(score, breakpoints=[60, 70, 80, 90], grades='FDCBA'):
        i = bisect.bisect(breakpoints, score)
        return grades[i]

>>> [grade(score) for score in [33, 99, 77, 70, 89, 90, 100]]
['F', 'A', 'C', 'C', 'B', 'A', 'A']
{% endhighlight %}

来自 Python 官方文档的例子，写得简洁明了，按区间分级这么容易地解决了，值得借鉴。但是 `breakpoints` 参数的默认值是个可变类型（list）这并不是个好的实践。

## Flat Sequences

`list` 这种 container sequence 存的是对象的引用，在一些场景下性能不行，所以需要有一些存对象本身的序列类型，比如 `array.array`。

这些类似也被作者称为 flat sequences，但是这个词是他自己创造的。这些类型的序列其中的元素都必需是同个类型的。

### `array.array`

这种序列类型只能存一些基本类型。它的好处在于序列化/反序列化速度快。比如把 1000000 个浮点数的 `array.array` 序列化成文件 / bytes 速度飞快。

### Memory Views

内置的 `memoryview` 类主要用于操作其他数据结构（如 `array.array`, `list`）的内存空间，而不需要预先复制一份。这本来是个在 NumPy 中引入的概念，后来被移植到了 Python 标准库，但是去掉了跟数学相关的操作。还没深入了解这个库的用法，也许以后研究数据科学时会有机会。下面把书中的一段代码例子搬进来，增进理解：

{% highlight python %}
>>> numbers = array.array('h', [-2, -1, 0, 1, 2])
>>> memv = memoryview(numbers)
>>> len(memv)
5
>>> memv[0]
-2
>>> memv_oct = memv.cast('B')
>>> memv.tolist()
[254, 255, 255, 255, 0, 0, 1, 0, 2, 0]
>>> memv_oct[5] = 4
>>> numbers
array('h', [-2, -1, 1024, 1, 2])
{% endhighlight %}

后面书里又顺带介绍了 NumPy 跟 Scipy，并且给了一个简单的代码示例。这些内容不记入笔记了，在学习 __Python for Data Analysis__ 时再仔细说明。

## Deque 与其他队列

Deque 是 double ending queue（双端队列）的简称。对于需要在序列前端插入/删除元素的场景，它比 `list` 快很多。而且它可以设置一个最大的元素个数，插入时如果超出最大个数，另外一端的数据会被清除。

Python 同时为多线程编程引入了几个线程安全的 Queue 类，如 `Queue`, `LifoQueue` 以及 `PriorityQueue`。它们也可以设置最大元素个数，但是行为与 Deque 不太一样；如果插入时队列已达最大个数，那么这个插入请求会阻塞，等待队列元素被其他线程取出。

同样，`multiprocessing` 模块中也提供了一些 Queue 类用来做进程间通讯。`asyncio` 实现了同样功能的类，用于异步网络编程。

`heaqp` 自己本身不维护一个队列，而是提供了一些函数，让你可以将一个可变的序列变成 heap queue / priority queue。

## Slice 为啥是左闭右开？

Dijkstra 大师写了一篇 [[帖子|https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html]] 说明过这个问题，对于 2, 3, ..., 12 这个序列，下面哪一种表示法更优雅？

1. 2 <= i < 13
2. 1 < i <= 12
3. 2 <= i <= 12
4. 1 < i < 13

大师的帖子我看不太懂，但是大意是这样：这个下界跟上界之差应该表示序列的长度，所以 3,4 排除掉。同时 i 用来描述第一个元素时，它应该是个自然数，所以第一个元素用 0 来描述更优雅，所以对于上界应该用 <=，所以应该用第 1 种表示法。

## 扩展阅读

作者认为 Python Cookbook 第 2 版跟第 3 版都值得一读。第 2 版是以 Python 2.4 为基础写的，但是很多内容现在还适用。第 3 版做了相当多的重写，大多数强调了 Python 3 的新特性。

Python 的内置 `sorted` 和 `list.sort` 使用的排序算法是 Timsort。这种算法会根据数据的特点选择插入排序或者合并排序来实现排序。因为现实世界中的数据大多数是相对有序的，所以这种算法效率较高。Java / Android 也用了 Timsort。Timsort 的发明者 Tim Peters 也是 Python 的核心贡献者，还写了 Zen of Python。
