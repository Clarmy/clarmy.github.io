---
title: Cartopy 0.22 来了！安装难的问题将成为历史
date: 2023-08-10 21:54:00 +08:00
modified: 2023-08-10 21:54:00 +08:00
tags: [Python, Cartopy]
description: 一直以来，Cartopy 的安装都是一个老大难的事情，由于一些依赖包的问题，我们是无法直接用 pip 快速安装的，要么需要手动安装一些特殊的依赖，要么就转而用 conda 安装，但是用 conda 安装有一个很磨人的事情是它经常会因为计算包依赖之间的关系而卡在 Solving environment 步骤，导致安装过程并不丝滑。而现在，Cartopy 安装难的问题，要成为历史了。
---

一直以来，Cartopy 的安装都是一个老大难的事情，由于一些依赖包的问题，我们是无法直接用 pip 快速安装的，要么需要手动安装一些特殊的依赖，要么就转而用 conda 安装，但是用 conda 安装有一个很磨人的事情是它经常会因为计算包依赖之间的关系而卡在 Solving environment 步骤，导致安装过程并不丝滑。而现在，Cartopy 安装难的问题，要成为历史了。

就在上周五（8月4日），Cartopy 发布了 0.22.0 版本。先不说这个版本功能有什么更新，它最重要的改变那就是 开始支持预编译的轮子安装了！ 这意味着什么？意味着你可以像装其他包一样直接无脑怼 `$ pip install cartopy` 了，不需要你再去手动安装各种乱七八糟的依赖了，也不用忍受 conda 里无穷无尽的 Solving environment 了。

我们可以看下图，cartopy 已经根据不同系统进行了预编译，生成了不同版本的 .whl 文件。

![01](/assets/img/cartopy-0-22-installation-issues-become-history/01.webp)

而这些在 0.21.* 及以前的版本中是不存在的。

![02](/assets/img/cartopy-0-22-installation-issues-become-history/02.webp)

我们之前之所以觉得 Cartopy 在 pip 中难以安装，是因为它没有预编译轮子，以至于我们每次用 pip 安装的时候需要现场编译轮子，而由于上游包版本的各种复杂关系很容易导致轮子构建失败或者缺失某些库文件，现在 Cartopy 已经把各主要平台和不同 Python 版本的轮子预编译好了，我们安装的时候就是直接拿来就用，可以极大地减少安装的难度和时间。

我还特意去看了 Cartopy 的官方公告：

![03](/assets/img/cartopy-0-22-installation-issues-become-history/03.webp)

内容翻译过来：

> Cartopy v0.22是该项目开发中的一个重要进步。之前需要本地安装PROJ和GEOS库的要求已经被删除。之前的C语言PROJ库调用被pyproj替换，C语言GEOS调用被shapely替换。这意味着现在可以使用简单的 pip install cartopy 来安装Cartopy。

所以也就是说已经解决了非 Python 生态的依赖，转而直接使用已经构建了轮子的 Python 生态包，也就可以丝滑地使用 pip 安装了。从使用便利性上来说，0.22可以说是一个非常重大的里程碑了。

![03](/assets/img/cartopy-0-22-installation-issues-become-history/03.gif)
