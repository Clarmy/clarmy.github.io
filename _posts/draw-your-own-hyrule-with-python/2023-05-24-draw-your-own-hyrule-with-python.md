---
title: 用 Python 绘制你自己的海（nan）拉鲁
date: 2023-05-24 21:24 +08:00
modified: 2023-05-24 21:24 +08:00
tags: [DEM]
description: 说起来塞尔达2王国之泪发售快两周了，这两周我已经狂卷了60个小时在海拉鲁大地。今天我想祭出一个去年就已经写好的一段代码，用真实世界 DEM 数据（海南岛）绘制的一个海拉鲁 style 的地图。
---
说起来塞尔达2王国之泪发售快两周了，这两周我已经狂卷了60个小时在海拉鲁大地。今天我想祭出一个去年就已经写好的一段代码，用真实世界 DEM 数据（海南岛）绘制的一个海拉鲁 style 的地图。

为了方便获取 DEM 数据，我写了一个 pyterrain 的小包，可以通过制定经纬度范围和缩放等级自动下载和拼接出网格 DEM。安装方法：`pip install -U pyterrain`

```python
import copy

import numpy as np
import matplotlib.pyplot as plt
import matplotlib.colors as colors

from pyterrain import Terrain

if __name__ == "__main__":
    bbox = 108.444319, 20.161757, 111.318897, 18.05883  # 海南岛

    terrain = Terrain("qBD4m7PNT5apV-Xl7PROxA")

    xs, ys, elevation = terrain.fetch(bbox=bbox, quiet=False, coord="lonlat", zoom=12)

    land = copy.deepcopy(elevation)
    land[land < 0] = -9999

    fig = plt.figure(
        figsize=(elevation.shape[1] / 100, elevation.shape[0] / 100), dpi=100
    )
    ax = plt.Axes(fig, [0.0, 0.0, 1.0, 1.0])
    ax.set_axis_off()
    fig.add_axes(ax)

    hyrule = colors.LinearSegmentedColormap.from_list(
        "hyrule", ["#3D2E00", "#C6C7B0"]
    )  # colormap for hyrule land
    ax.contourf(xs, ys, land, cmap=hyrule, levels=np.arange(5, land.max(), 2), zorder=3)

    ax.contourf(
        xs, ys, elevation, levels=[elevation.min(), 0], colors=["#212A2D"], zorder=1
    )

    ax.contourf(xs, ys, elevation, levels=[0, 5], colors=["#41535A"], zorder=4)

    ax.contour(
        xs,
        ys,
        elevation,
        colors="#382D06",
        levels=np.arange(5, elevation.max(), 20),
        alpha=0.6,
        linewidths=0.4,
        zorder=4,
    )

    ax.contour(
        xs,
        ys,
        elevation,
        colors="#382D06",
        levels=np.arange(5, elevation.max(), 100),
        alpha=0.6,
        linewidths=0.8,
        zorder=5,
    )
    fig.savefig("./hynanrule.png")
```
程序运行的过程中，它会自动进行瓦片 DEM 的下载、拼接，最后画图。图片效果如下：

![hynanrule](/assets/img/draw-your-own-hyrule-with-python/01.png)

我们看一下细节：

![hynanrule](/assets/img/draw-your-own-hyrule-with-python/02.png)
![hynanrule](/assets/img/draw-your-own-hyrule-with-python/03.webp)

是不是有内味儿了🙃 哦对了，代码里用了我自己的一个 API key，我不能保证它永远都有效。