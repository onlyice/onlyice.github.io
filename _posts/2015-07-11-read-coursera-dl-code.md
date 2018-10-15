---
layout: post
title: 阅读 coursera-dl 源码
tags: [Python]
---

![Coursera Logo]({{ site.image_cdn }}/images/2015/07/coursera.png)

[coursera-dl][] 是一个 GitHub 上的开源项目，用于方便地下载 coursera 上的课程资源（视频、字幕、幻灯片）等。

<!--more-->

比如你报名参加了 [Interactive Programming in Python][interactive-programming-in-python]，你可以用 `coursera-dl` 工具下载它的全部资源：

{% highlight bash %}
> $ coursera-dl -u your_coursera_email -p your_coursera_password interactivepython1-003
downloading...
> $ tree interactivepython1-003
interactivepython1-003
└── 01_Week_0a_-_Expressions
    ├── 01_Introduction_14-29.mp4
    ├── 01_Introduction_14-29.srt
    ├── 01_Introduction_14-29.txt
    ├── 02_CodeSkulptor_11-22.mp4
    ├── 02_CodeSkulptor_11-22.srt
    ├── 02_CodeSkulptor_11-22.txt
    ├── 03_Arithmetic_expressions_13-24.mp4
    ├── 03_Arithmetic_expressions_13-24.srt
    └── 03_Arithmetic_expressions_13-24.txt
{% endhighlight %}

[coursera-dl]: https://github.com/coursera-dl/coursera
[interactive-programming-in-python]: https://class.coursera.org/interactivepython1-003

看完 `coursera-dl` 的代码后，发现其实现并不复杂，但是由于 coursera 的广泛流行，这个项目在 GitHub 上已经有了 2300+ 个 star。

这篇文章会讲解它的代码实现中的重点部分，如果你有基础的 Python 以及爬虫知识，看完后应该可以借鉴一下思路，写出一个诸如“网易云课堂”下载器之类的项目。

## 代码流程

1. 解析命令行参数，获取用户名、密码、要下载的课程等
2. 登陆 coursera
3. 访问课程相关的 JSON 接口，获取视频列表以及视频、幻灯片的下载地址
4. 使用 wget 或者纯 Python 形式进行下载

## 解析命令行参数

代码：[`coursera/coursera_dl.py::parse_args`][parse_args]

解析命令行参数使用的是 Python 内置的 `argparse` 库。代码中把参数分成了几类，用来表示要下载的课程，使用什么方式做登陆验证，使用什么下载器等等。比如：

{% highlight python %}
group_basic = parser.add_argument_group('Basic options')
group_basic.add_argument('class_names',
                         action='store',
                         nargs='+',
                         help='name(s) of the class(es) (e.g. "ml-005")')

group_adv_auth = parser.add_argument_group('Advanced authentication options')
group_adv_auth.add_argument('-k',
                            '--keyring',
                            dest='use_keyring',
                            action='store_true',
                            default=False,
                            help='use keyring provided by operating system to '
                            'save and load credentials')
{% endhighlight %}

[argparse-crash-course]: /2015/06/08/argparse-5-minutes-crash-course/
[parse_args]: https://github.com/coursera-dl/coursera/blob/ab276004db063345f79083abbef80bf0397abb38/coursera/coursera_dl.py#L589

## 登陆验证

代码：[`coursera/cookies.py::login`][login]

`coursera-dl` 做的登陆验证，本质上是模拟网页版的 coursera 登陆，向 coursera 服务器发送登陆请求来拿到登陆后的票据。登陆后的票据以 cookies 的形式通过 coursera 传回，有了这个票据后，后续对课程页面的请求就可以通过 coursera 的验证。

如果你对这种过程没有过了解，你可以在 Chrome 隐身模式下观察一下整个登陆过程是怎样的（FireFox 等其他浏览器同理）：

1. 打开一个新的 Chrome 隐身窗口
2. 打开开发者工具（F12），切换到 Network tab，勾选 Preserve log
3. 在地址栏输入 https://www.coursera.org/ 进行访问，留意 Network Tab 中 coursera 服务器的 HTTP 回包
4. 点击 Login 进行登陆

如果你仔细观察了第 3 步中的 HTTP 回包，你会发现有一个 `Set-Cookie`：

    Set-Cookie:CSRF3-Token=1437446864.f2FXeqcfI3mOAVaL; Expires=Tue, 21 Jul 2015 02:47:44 GMT; Path=/; Domain=.coursera.org

这个 `CSRF3-Token` 是用来防跨站请求伪造（这里不深究），会在登陆时带过去给服务器。登陆请求在你点击登陆按钮后发起，URL 是：

    https://www.coursera.org/api/login/v3Ssr?csrf3-token=1437446864.f2FXeqcfI3mOAVaL

观察它的请求包，你会发现用户名和密码放在了 POST body 中，然后 coursera 的回包中会带回一个 `CAUTH` 的 cookie。有了这个 cookie，我们后面就可以访问课程主页、列表页等。于是整个登陆验证的过程就结束了。

`coursera-dl` 中使用的登陆 URL 是 `https://www.coursera.org/api/login/v3` ，可能是 coursera 手机客户端使用的接口；登陆流程也与我前面描述的略有不同，但是原理是类似的。

同时留意一下，代码中用了 [`requests.Session`][requests-session] 用来使登陆后拿到的 cookie，在后续的 HTTP 请求中仍可用。

[login]: https://github.com/coursera-dl/coursera/blob/ab276004db063345f79083abbef80bf0397abb38/coursera/cookies.py#L71
[requests-session]: http://docs.python-requests.org/en/latest/user/advanced/#session-objects

## 多样的获取凭证方式

代码：[`coursera/credentials.py::get_credentials`][get_credentials]

`coursera-dl` 提供了 4 种获取凭证（即用户名、密码）的方式：

1. 直接在命令行提供用户名密码
2. 使用 `netrc` 文件
3. 使用 keyring
4. 使用 cookie 文件

其中 `netrc` 和 keyring 方法可以使你不需要在命令行上输入用户名密码，相对减少了一些无意泄露密码的危险。cookie 文件用处不大，可能适合开发者调试。

这四种方式相关的信息都可以简单 Google 以及看代码得到，而且 2~4 并不是很重要，这里不做过多叙述。

[get_credentials]: https://github.com/coursera-dl/coursera/blob/ab276004db063345f79083abbef80bf0397abb38/coursera/credentials.py#L139

## 获取课程相关信息和资源下载 URL

代码：[`coursera/coursera_dl.py::download_class`][download_class]

这里的代码通过 coursera 提供的 API 抓取课程相关的数据和 URL 用来组织数据和下载。

比如在 [lecture][] 页提取视频地址，或者在 Learn Chinese 课程的 [API 接口][learn-chinese]中提取相关的信息。

但是如何找到我们需要的 API 地址呢？

API 大致分为两类，一类返回渲染好的 HTML ，一般用在 Web 服务上；另一类返回结构化的数据（比如 JSON, XML），一般用于手机 App 上。

所以你可以通过观察浏览器中的 HTTP 请求来判断 API 地址；或者通过在手机上抓包，来判断手机上的 coursera app 使用的是否是 HTTP 协议，以及 App 使用的 API 是怎样的。

拿到 API 地址及用法后，就可以进行解析了。如果 API 返回的是 HTML 数据，可以用 [`BeautifulSoup4`][bs4], [`pyquery`][pyquery] 等包来解析；如果是 JSON 数据，你知道该怎么做。

[download_class]: https://github.com/coursera-dl/coursera/blob/ab276004db063345f79083abbef80bf0397abb38/coursera/coursera_dl.py#L888
[lecture]: https://class.coursera.org/interactivepython1-003/lecture
[learn-chinese]: https://www.coursera.org/api/opencourse.v1/course/learn-chinese
[bs4]: http://www.crummy.com/software/BeautifulSoup/bs4/doc/
[pyquery]: https://pythonhosted.org/pyquery/

## 下载器类的实现

代码：[`coursera/downloaders.py`][downloaders]

`coursera-dl` 即可以使用外部下载器（wget, curl 等），也可以使用 `native` 的下载器（用 requests 直接下载）。对于外部下载器，代码中使用了 `subprocess` 来呼起下载。

`ExternalDownloader` 的实现比较有意思。对于呼起外部下载程序， `coursera-dl` 的代码需要做两个事情：

1. 将 cookie 传给外部下载器
2. 拼装命令行参数供 subprocess 使用

于是 `ExternalDownloader` 类定义了两个接口用来实现，分别是 `_add_cookies` 和 `_create_command`：

{% highlight python %}
class WgetDownloader(ExternalDownloader):
    bin = 'wget'

    def _add_cookies(self, command, cookie_values):
        command.extend(['--header', "Cookie: " + cookie_values])

    def _create_command(self, url, filename):
        return [self.bin, url, '-O', filename, '--no-cookies',
                '--no-check-certificate']
{% endhighlight %}

这里的设计巧妙而简洁，值得借鉴。

[downloaders]: https://github.com/coursera-dl/coursera/blob/ab276004db063345f79083abbef80bf0397abb38/coursera/downloaders.py

## 结束语

如果你看完这篇教程后也想自己动手做点东西，请尽管尝试，并把你的成果告诉我。
