---
title: 关于如何解决 Cartopy 中的 GEOSException 问题
date: 2025-01-11 22:36:00 +08:00
modified: 2025-01-11 22:36:00 +08:00
tags: [Cartopy, Python, Matplotlib]
description: ""
comments: true
---

早在去年上半年的时候，我在使用 Cartopy 绘制 contourf 等值线图时偶尔会遇到 `GEOSException`。当时我在 Cartopy 项目的仓库里提交了一个 [Issue](https://github.com/SciTools/cartopy/issues/2370)。后来官方团队的开发者一直没有给解决，我就自己提交了一个 [PR](https://github.com/SciTools/cartopy/pull/2373) 试图去解决这个问题，在 BUG 得到复现以后官方团队开始关注该问题。经过讨论大家已经基本了解了问题的本质：shapely 在构建多边形时可能会出现冲突点，而这个情况被 contourpy、matplotlib 和 cartopy 层层继承。由于 matplotlib 的绘图并不需要检查多边形的拓扑错误也可以正确绘制图形，但是 cartopy 却需要依赖 shapely 的多边形对象判断几何关系，所以一旦出现拓扑错误就会导致程序崩溃。

在解决这个问题的思路上我和官方团队的两位开发者 Ruth 和 Greg 有一些分歧，Greg 认为应该使用 `is_valid` 方法来检查每一个多边形的合法性，但是这种方法会导致程序的性能大幅降低。而我的思路是在捕获 `GEOSException` 异常以后，对多边形进行两次 `.buffer` 操作，可以修复多边形的拓扑错误，但是这种方法不够优雅，它需要额外计算一下多边形的尺度，并在 `buffer` 操作时传入一个足够微小的值，既能修复拓扑错误，又不会影响图形的显示效果。 而 Ruth 作为这个 PR 的 reviewer，她虽然赞同 `is_valid` 方法不适用，但是也并没有支持我的方案，这个 PR 就一直僵持在那里。

在这个 PR 提交了半年多时间之后，也就是 2024 年 11 月的时候，我收到邮件通知 Greg 希望我回去继续 follow 这个问题。当时 Cartopy 已经更新了一个大版本，而 Ruth 在多边形模块做了很多代码重构，Greg 怀疑这个问题已经得到了解决，所以希望我继续推进一下这个遗留了很久的 PR，如果确实解决的话就可以关闭了。于是我回来重新开始进行测试，发现代码有很多改动但是问题依旧存在。但当我把问题重新报给 Ruth 时，她说现在在她的开发环境里做测试时这个问题已经被解决了。最后我们一起做了很多交叉验证，发现如果我们的上游依赖包 GEOS 的版本达到了 3.13，那么这个问题就会消失。于是我去 shapely 的项目里提交了一个 [PR](https://github.com/shapely/shapely/pull/2187) 去升级 GEOS 的版本到 3.13，然后这个 PR 直到现在还没有被合并，所以使用 cartopy 时仍然可能会遇到这个问题。

在遇到这个问题时，目前的解决方法是手动升级 shapely 所依赖的本地 GEOS 版本到 3.13。在 conda 环境下的具体操作是：

1. 卸载 shapely ：`pip uninstall shapely`
2. 安装 3.13 版本的 GEOS ：`conda install -c conda-forge geos=3.13`
3. 重新安装不带二进制版本的 shapely ：`pip install shapely --no-binary shapely`

建议在 Linux、MacOS 环境下使用，Windows 环境下经常会有一些未知且棘手的问题。

希望 shapely 尽快更新版本把 GEOS 的 binary 包升级到 3.13 吧，有点遗憾的是差一点就能成为 cartopy 的 contributor 了哈哈。