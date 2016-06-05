---
layout: post
title: "Chapter 3: Dictionaries and Sets"
tags: [FluentPython, Python]
---

{% include serials/fluent-python.md %}

这一章主要读 Python 中的 `dict` 和 `set`，比较基础。

<!--more-->

## `dict` 比较相等时是基于内容的

{% highlight python %}
>>> a = dict(one=1, two=2, three=3)
>>> b = {'one': 1, 'two': 2, 'three': 3}
>>> c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
>>> d = dict([('two', 2), ('one', 1), ('three', 3)])
>>> e = dict({'three': 3, 'one': 1, 'two': 2})
>>> a == b == c == d == e
True
{% endhighlight %}

## `setdefault` 函数可以用来处理不存在的 Key

当你的 `dict` 的 `value` 为可变元素时（比如 `list`），`setdefault` 往往是个好东西。

比如你有一堆人员信息，想按年龄分组时，不用 `setdefault` 的做法：

{% highlight python %}
persons_by_age = {}
for person in persons:
    if person.age not in persons_by_age:
        persons_by_age[person.age] = [person]
    else:
        persons_by_age[person.age].append(person)
{% endhighlight %}

这样写不如 `setdefault` 优雅：

{% highlight python %}
persons_by_age = {}
for person in persons:
    persons_by_age.setdefault(person.age, []).append(person)
{% endhighlight %}

`setdefault` 在这个 key (`person.age`) 不存在时，自动建一个空列表。

## 用 `__missing__` 函数处理 key 不存在时的行为

这个能力很少需要用到。

默认的 `dict` 实现，有 `__getitem__` 时（也就是做 `d[k]` 操作时），如果 key 不存在，而且当前的 `dict` 实例有实现 `__missing__` 函数时，会调用 `__missing__` 函数并返回其结果。

比如书中实现了一个例子，用来实现传入 `int` 型 key 时，也查他对应的 `str(key)`：

{% highlight python %}
class StrKeyDict0(dict):
    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key)
        else:
            return self[str(key)]

    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()
{% endhighlight %}

留意几点：

* `__missing__` ''只会''在 `__getitem__` 时才会调用。所以你还需要实现 `__contains__` 用来做 `in` 判断。
* `__contains__` 中不能直接用 `key in self` 来判断，因为 `in` 操作符就是调用 `__contains__`，会引起函数递归调用
* `self.keys()` 在 Python 2 中返回的是一个 `list`，`in` 遍历时性能差；在 Python 3 中返回的是一个 view，跟 set 的结构类似，`in` 的性能好

这个例子写起来并不方便，Python 里面又实现了一个 `UserDict` 类用来方便用户扩展。`UserDict` 在其内部维护了一个 `dict` 属性 `self.data`，所有数据操作都对它进行：

{% highlight python %}
class StrKeyDict(collections.UserDict):
    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]

    def __contains__(self, key):
        return str(key) in self.data

    def __setitem__(self, key, item):
        self.data[str(key)] = item
{% endhighlight %}

留意一下我们的 `__contains__` 函数不再需要蛋疼地去取 `self.keys()` 了。

## 不可变的 `dict`

Python 实现了一个 `types.MappingProxyType`，用来实现只读的 `dict`。

{% highlight python %}
>>> from types import MappingProxyType
>>> d = {1: 'A'}
>>> d_proxy = MappingProxyType(d)
>>> d_proxy
mappingproxy({1: 'A'})
>>> d_proxy[1]
'A'
>>> d_proxy[2] = 'x'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
>>> d[2] = 'B'
>>> d_proxy[2]
'B'
{% endhighlight %}

留意下 `d` 发生变化时会反映到 `d_proxy` 上。这也说明了为啥它是个 "proxy"。

## 其他实用的 `dict` 变种

### `collections.OrderedDict`

`dict` 的 keys 按插入顺序排序。

### `collections.ChainMap`

可以传入多个 dict 构造一个 ChainMap。查询这个 ChainMap 时，按传入的 dict 顺序去查，如果这几个 dict 都查不到，再抛 `KeyError`。

传入的 dict 后续发生数据变化时，也会反映到 ChainMap 上。

### `collections.Counter`

方便用来计数。

### `collections.defaultdict`

另外一种形式实现默认 value。

## `set` 的操作符

`set` 有一堆操作符挺有意思，实现了丰富的集合操作。日常编码时可以考虑下。需要留意下的是，`set` 的操作符要求两个操作数都是 `set`，但是它有个同样功能的函数，可以接受任意的 iterable：

{% highlight python %}
>>> s = {1, 2, 3}
>>> z = {2, 3, 4}
>>> l = [3, 4, 5]
>>> s & z
{2, 3}
>>> s & l
TypeError: unsupported operand type(s) for &: 'set' and 'list'
>>> s.intersection(l)
{3}
{% endhighlight %}

## `dict` 和 `set` 底层实现

这部分内容是本章重点。

`dict` 和 `set` 都用了哈希表作为其底层实现。

![dict-hashtable-flowchart]({{ site.url }}/images/2016/06/dict-hashtable-flowchart.png)

### Key 需要是 hashable 的

因为使用了哈希表，所以 `dict` / `set` 的 Key 需要是 hashable 的。一个对象想要是 hashable，必须实现几个条件：

1. 它必须实现 `__hash__` 函数，返回一个整型，并且在它的生命周期内总是返回同一个值
2. 它必须实现 `__eq__` 函数以支持相等判断
3. 如果 a == b 为真，那么 hash(a) == hash(b) 也需要为真

Python 标准库中，可变类型都不是 hashable 的，不可变类型都是 hashable 的。但是对于 tuple 来讲，它里面的元素需要全部是 hashable 的，它才是 hashable 的：

{% highlight python %}
>>> a = (1, "Hello", {1, 2})
>>> hash(a)
TypeError: unhashable type: 'set'
>>> b = (1, "Hello", frozenset({1, 2}))
>>> hash(b)
6191337126678801885
{% endhighlight %}

另外，用户定义的类型都是 hashable 的，因为它的 hash 值是它的 `id()`，不同实例间的 `id()` 值不一样。

如果你的自定义类型实现的 `__eq__` 函数，你也必须实现一个合适的 `__hash__` 函数，并且保证 `a == b` 为真时， `hash(a) == hash(b)` 也为真。不然你的这个类型的实例与 `dict` / `set` 一起工作时，会有不确定的行为。同时，如果你的自定义类型不想支持 hashable，那么你要显示实现 `__hash__` 函数，使其抛出一个 `TypeError`。

### `dict` / `set` 有显著的内存消耗

由于哈希表要实现快速的搜索，所以它需要保持稀疏，会占用较多的内存空间。

Python 会预先分配一片空间，当这片空间不足够使用时（并不一定是满了，有可能是足够密集了），它又会把数据都拷贝到另外一片更大的空间。拷贝后计算一个 key 的 bucket 位置的方式会发生变化（参考前面的图）。

同时，`dict` 在保存数据库纪录之类的场景，会比 tuple 有更多的内存消耗。比如每一个 Record，在 tuple 里面并不需要保存 field name，但是在 dict 里面需求。

用户自定义类型在底层实现上，也使用了 `dict` 来维护其属性。书中后面的章节会说明如何用 `__slots__` 来节省空间。

### 其他值得说明的点

* Key 的顺序跟插入顺序有关
* 遍历的过程不要插入新元素
