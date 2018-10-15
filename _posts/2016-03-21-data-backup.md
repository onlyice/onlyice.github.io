---
layout: post
title: "数据备份"
tags: [Management]
---

![data]({{ image_cdn }}/images/2016/03/data.jpg)

数据备份的重要性，被断电/误删/电脑死机搞丢过文档的童鞋应该都懂。这篇文章描述下我的数据备份解决方案。

<!--more-->

## 备份到网盘

目前我使用了两个网盘做备份：

1. 重要数据通过 Dropbox 同步
2. 照片通过 Google Photos 备份
3. Dropbox 的数据再单向备份一份到 Google Drive

使用 Dropbox 的原因：

* 稳定可靠，基本不丢数据
* 同步数据快，对于 Word 文件等也能做增量同步
* 我的主力平台上全部支持，Windows / Linux / Android
* 文件支持多版本
* 目前不用太担心产品会死
* 产品又做得即漂亮又简单易用

但是它有个缺点就是容量太小。所以不适合放照片等大体积的资料。如果后面有需求会上 Dropbox Pro，1T 的容量应该足够用很久。

照片使用了 Google Photos，原因有几点：

* 只要接受照片被压缩，可以无限量上传图片
* 压缩对画质的损失很小
* 搜索功能实在太牛逼了，可以根据物品/场景等搜索照片
* 自动对同一脸孔做照片聚合

## 使用 rclone 自动同步 Dropbox 到 Google Drive

为了避免鸡蛋全部放在一个篮子里，我在一台 Arch Linux VPS 上使用了 rclone 将 Dropbox 的数据定时同步到 Google Drive。

rclone 应该是目前最好的网盘同步工具了。它的原理是使用各家存储服务提供的 API，遍历文件目录并获取文件 Hash 以判断文件是否相同。它有一个 sync 命令可以将一个存储服务中的文件单向同步到另外一个服务。

[GitHub][1] \| [官网][2]

### 安装

Arch Linux 上已经有现在的 rclone 包。直接 pacman 安装即可。

{% highlight bash %}
$ sudo pacman -S rclone
{% endhighlight %}

### 用法

rclone 官网的文档写得很详细，用法可以直接参照它的文档。

首先运行 `rclone config` 配置 Dropbox 与 Google Drive。最终生成一个名为 "Dropbox" 和一个名为 "Google Drive" 的两个 remote 配置。

配置默认放在了个人 home 目录下（`/home/onlyice/.rclone.conf`）。

最终运行这条命令，可以把 Dropbox 内容同步到 Google Drive，最终在你的 Google Drive 根目录下会有一个名为 `Dropbox Backup` 的文件夹，里面有你的 Dropbox 全部数据。

{% highlight bash %}
$ rclone sync Dropbox:/ "Google Drive":"/Dropbox Backup"
{% endhighlight %}

### 定时同步

这里使用 systemd 的 Timer 能力来做定时同步。

新建 `/usr/lib/systemd/system/backup-dropbox.service` 文件：

{% highlight ini %}
[Unit]
Description=Backup Dropbox to Google Drive

[Service]
Type=simple
ExecStart=/usr/bin/rclone sync Dropbox:/ "Google Drive":"/Dropbox Backup"
User=onlyice
KillMode=process

[Install]
WantedBy=default.target
{% endhighlight %}

再新建 `/usr/lib/systemd/system/backup-dropbox.timer` 文件：

{% highlight ini %}
[Unit]
Description=Daily backup Dropbox to Google Drive

[Timer]
OnBootSec=5min
OnUnitActiveSec=1d
TimeoutStartSec=1h

[Install]
WantedBy=timers.target
{% endhighlight %}

参数说明：

* `OnBootSec` 表示启动后 5 分钟开始执行
* `OnUnitActiveSec` 表示上次执行 1 天后再执行
* `TimeoutStartSec` 让 systemd 一小时后再去关心任务的执行情况
* `TimeoutStartSec` 值默认是 90s，90s 不足够让 rclone 完成同步，这时 systemd 会认为任务超时失败

之后再使用 systemctl 启用这个定时任务：

{% highlight bash %}
$ sudo systemctl enable backup-dropbox.timer
$ sudo systemctl start backup-dropbox.timer
{% endhighlight %}

这样就可以每天定时同步文件了。

### 加密

因为 rclone 申请的网盘权限比较多，相关的 API 参数放在 `.rclone.conf` 中，如果没有加密可能被坏人盗走，坏人就可以读取你全部的文件内容。

rclone 提供了一种设置密码的手段，会把 `.rclone.conf` 整个文件加密。但是代价是每次运行 rclone 命令行时都需要输入密码。因为我需要脚本每天同步 Dropbox 内容到 Google Drive，所以这里暂时没有加密上。

`.rclone.conf` 默认是 0600 权限的，即是只有你或者 root 用户可以查看它的内容。如果你的 SSH 登陆过程不会被攻破，那么这个文件应该是基本安全的。

[1]: https://github.com/ncw/rclone
[2]: http://rclone.org/
