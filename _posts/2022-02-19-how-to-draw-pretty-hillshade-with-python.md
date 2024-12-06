---
title: 用 Python 绘制炫酷的立体地形图
date: 2022-02-19 00:00:00 +08:00
modified: 2022-02-19 00:00:00 +08:00
tags: [Python, Matplotlib, Cartopy, cnmaps]
description: 众所周知，Python 的 matplotlib 是一个非常全面的制图库，它不仅可以绘制图表、地图，还可以绘制3D效果图，试想一下，如果你在写论文的时候，可以将立体地形图作为底图，那逼格噌一下子就上来了，今天我就来教大家画一个带有地理坐标属性的立体地形图。
---
众所周知，Python 的 matplotlib 是一个非常全面的制图库，它不仅可以绘制图表、地图，还可以绘制 3D 效果图，试想一下，如果你在写论文的时候，可以将立体地形图作为底图，那逼格噌一下子就上来了，今天我就来教大家画一个带有地理坐标属性的立体地形图，啥也不说，咱先上效果图：

![hillshades](/assets/img/how-to-draw-pretty-hillshade-with-python/hillshades.gif)

上面这张图是展示了基于 matplotlib + cartopy 的山地阴影图在不同光影参数下的变化效果。这个变化效果有利于我们理解 matplotlib 对该效果的设计理念。

在我讲解之前，我推荐大家读一下 matplotlib 官方文档库里的这一篇文章：[Topographic hillshading](https://matplotlib.org/stable/gallery/specialty_plots/topographic_hillshading.html)，该文章已经介绍了如何单独基于 matplotlib 绘制山地阴影图，并给出了不同渲染参数下的渲染效果图。我当初对山地立体图的学习就是从这篇文章开始的。

本教程代码所需依赖：
```bash
matplotlib
cartopy>=0.19.0
cnmaps==0.2.1
netCDF4
numpy
```
本教程使用的 DEM 数据：[和鲸社区数据集: CHINA_DEM](https://www.heywhale.com/mw/dataset/620f914468364e0017a3b57a/file?shareby=620ce369c1ae5e001747085e) 

## 神说：要有光
光，是三维世界最重要的东西，要绘制山地立体图，首先需要理解 matplotlib 中的 `LightSource` 对象，顾名思义，这个对象就是“光源”，与 3D 建模里的光源是同一个东西，它的调用方法是：
```python
from matplotlib.colors import LightSource
ls = LightSource(azdeg=360, altdeg=30)
```
其中 `azdeg` 是方位角，`altdeg` 是高度角，这两个参数可以确定一个光源的投射方向，进而可以知道被光源投射的物体，哪一部分应该是光，哪一部分应该是影，而光影便是实现地形立体效果的金钥匙。

在我们创建了光源以后，就需要基于该光源对地形数据生成光影对象，通常情况下，对于山地阴影，我们有两个方法可以选择，一个是 `hillshade`，另一个是 `shade`，其中 `hillshade` 返回的是以0-1的数字代表的光影明暗特征，你可以把它理解为一个灰度图，而 `shade` 返回的是一个RGBA数组，也就是彩图，下面我们使用 `shade` 来看一个实际的例子：
```python
import netCDF4 as nc
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
from matplotlib.colors import LightSource
from cnmaps import get_map, draw_map

ds = nc.Dataset('/assets/img/how-to-draw-pretty-hillshade-with-python/data/cldasgrid_dem.nc')

_lon = ds.variables['LON'][:]
_lat = ds.variables['LAT'][:]
_dem = ds.variables['elevation'][:]

lon = _lon[4032: 4662]
lat = _lat[1635: 2134]

dem = _dem[1635: 2134, 4032: 4662]

ls = LightSource(azdeg=360, altdeg=30)

rgb = ls.shade(dem[::-1], cmap=plt.cm.gist_earth, blend_mode='overlay',
               vert_exag=0.5, dx=10, dy=10, fraction=1.5, vmin=-2300)

fig = plt.figure(figsize=(8, 8))
ax = fig.add_subplot(111, projection=ccrs.PlateCarree())

img = ax.imshow(rgb, extent=(lon.min(),lon.max(),lat.min(),lat.max()), transform=ccrs.PlateCarree())

draw_map(get_map('河南'), color='w', linewidth=2)

ax.set_extent(get_map('河南').get_extent(buffer=0))

plt.show()
```

![henan](/assets/img/how-to-draw-pretty-hillshade-with-python/henan.png)

这样，我们的第一张立体地形图就出来了，是不是很炫酷？此外，如果你调整 `azdeg` 和 `altdeg` 的值，阴影的方位就会随之改变，就像文章开头那张动图一样，它就是通过修改 `azdeg` 的值以达到光线旋转照射的效果的。

## 光影参数详解
接下来，我们需要了解一下 `ls.shade` 方法的各个参数是干什么的，首先第一个位置函数肯定是我们的 dem 数据，这里需要注意的是，你必须把 dem 的纬度顺序调整为低纬->高纬的顺序，否则渲染出来的图片是反的。`cmap` 是色标这个大家应该都知道就不赘述了，`blend_mode` 这个参数大家会比较陌生，它是一种渲染模式选择，预置选项有：`'hsv'`，`'overlay'`，`'soft'`。它们分别是什么意思呢？官方文档在这个参数上的解释一大堆，但是根据我的理解，这个参数其实就类似于“滤镜”，你可以调整这三个滤镜来看看哪个是你喜欢的效果，我们来试一下：

```python
import netCDF4 as nc
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
from matplotlib.colors import LightSource
from cnmaps import get_map, draw_map

ds = nc.Dataset('/assets/img/how-to-draw-pretty-hillshade-with-python/data/cldasgrid_dem.nc')

_lon = ds.variables['LON'][:]
_lat = ds.variables['LAT'][:]
_dem = ds.variables['elevation'][:]

lon = _lon[4032: 4662]
lat = _lat[1635: 2134]

dem = _dem[1635: 2134, 4032: 4662]

ls = LightSource(azdeg=360, altdeg=30)

fig = plt.figure(figsize=(8*3, 8))
fig.tight_layout()

for i, mode in enumerate(['soft', 'overlay', 'hsv']):
    rgb = ls.shade(dem[::-1], cmap=plt.cm.gist_earth, blend_mode=mode,
                   vert_exag=0.5, dx=10, dy=10, fraction=1.5, vmin=-2300)

    
    ax = fig.add_subplot(1,3,i+1, projection=ccrs.PlateCarree())

    img = ax.imshow(rgb, extent=(lon.min(),lon.max(),lat.min(),lat.max()), transform=ccrs.PlateCarree())

    draw_map(get_map('河南'), color='w', linewidth=2)
    
    plt.title(mode)

    ax.set_extent(get_map('河南').get_extent(buffer=0))

plt.show()
```

![blend_mode](/assets/img/how-to-draw-pretty-hillshade-with-python/blend_mode.png)

看得出来，还是 `overlay` 看起来更均衡一些。

下面我们来看一下 `vert_exag` 参数，这个参数表征的是顶点的突出程度，这个点的值越大，立体感会越强，但是如果你把它设得太大，也会有一些副作用，来看示例：

```python
import netCDF4 as nc
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
from matplotlib.colors import LightSource
from cnmaps import get_map, draw_map

ds = nc.Dataset('/assets/img/how-to-draw-pretty-hillshade-with-python/data/cldasgrid_dem.nc')

_lon = ds.variables['LON'][:]
_lat = ds.variables['LAT'][:]
_dem = ds.variables['elevation'][:]

lon = _lon[4032: 4662]
lat = _lat[1635: 2134]

dem = _dem[1635: 2134, 4032: 4662]

ls = LightSource(azdeg=360, altdeg=30)

fig = plt.figure(figsize=(8*3, 8))
fig.tight_layout()

for i, ve in enumerate([0.5, 1, 10]):
    rgb = ls.shade(dem[::-1], cmap=plt.cm.gist_earth, blend_mode='overlay',
                   vert_exag=ve, dx=10, dy=10, fraction=1.5, vmin=-2300)

    
    ax = fig.add_subplot(1,3,i+1, projection=ccrs.PlateCarree())

    img = ax.imshow(rgb, extent=(lon.min(),lon.max(),lat.min(),lat.max()), transform=ccrs.PlateCarree())

    draw_map(get_map('河南'), color='w', linewidth=2)
    
    plt.title(ve)

    ax.set_extent(get_map('河南').get_extent(buffer=0))

plt.show()
```

![vert_exag](/assets/img/how-to-draw-pretty-hillshade-with-python/vert_exag.png)

上图从左到右 `vert_exag` 分别等于0.5、1和10，可以看出来，当 `vert_exag==10` 的时候，平原地图会有很强的噪点效果，这也是 `vert_exag` 值设置过大的一个副作用，在实际绘图的时候，需要综合考虑，选一个合适的值。当然，对于 `vert_exag` 参数，还有另外两个参数会与之配合（或者说制衡），那就是 `dx` 和 `dy`，这两个参数的含义是在平面空间上单个顶点的重采样间隔，`dx` 和 `dy` 的值越小，图像越能展现原始的数据细节，`dx` 和 `dy` 的值越大，那么最终出来的图越平滑，一些原始数据的细节就会被平滑掉了。下面我们来看一下不同的`dx`，`dy` 取值，对图像效果有什么影响。

```python
import netCDF4 as nc
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
from matplotlib.colors import LightSource
from cnmaps import get_map, draw_map

ds = nc.Dataset('/assets/img/how-to-draw-pretty-hillshade-with-python/data/cldasgrid_dem.nc')

_lon = ds.variables['LON'][:]
_lat = ds.variables['LAT'][:]
_dem = ds.variables['elevation'][:]

lon = _lon[4032: 4662]
lat = _lat[1635: 2134]

dem = _dem[1635: 2134, 4032: 4662]

ls = LightSource(azdeg=360, altdeg=30)

fig = plt.figure(figsize=(8*3, 8))
fig.tight_layout()

for i, dd in enumerate([1, 10, 100]):
    rgb = ls.shade(dem[::-1], cmap=plt.cm.gist_earth, blend_mode='overlay',
                   vert_exag=0.5, dx=dd, dy=dd, fraction=1.5, vmin=-2300)

    
    ax = fig.add_subplot(1,3,i+1, projection=ccrs.PlateCarree())

    img = ax.imshow(rgb, extent=(lon.min(),lon.max(),lat.min(),lat.max()), transform=ccrs.PlateCarree())

    draw_map(get_map('河南'), color='w', linewidth=2)
    
    plt.title(f'dx={dd}, dy={dd}')

    ax.set_extent(get_map('河南').get_extent(buffer=0))

plt.show()
```

![dx_dy](/assets/img/how-to-draw-pretty-hillshade-with-python/dx_dy.png)

可以看到，当 `dx` 和 `dy` 偏小的时候，同样出现了噪点问题。

最后还有一个很重要的参数就是 `fraction`，它是一个控制光影效果强度的参数，这个值越大，明暗的对比度就越大，我们来看一下对比效果：

```python
import netCDF4 as nc
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
from matplotlib.colors import LightSource
from cnmaps import get_map, draw_map

ds = nc.Dataset('/assets/img/how-to-draw-pretty-hillshade-with-python/data/cldasgrid_dem.nc')

_lon = ds.variables['LON'][:]
_lat = ds.variables['LAT'][:]
_dem = ds.variables['elevation'][:]

lon = _lon[4032: 4662]
lat = _lat[1635: 2134]

dem = _dem[1635: 2134, 4032: 4662]

ls = LightSource(azdeg=360, altdeg=30)

fig = plt.figure(figsize=(8*3, 8))
fig.tight_layout()

for i, fraction in enumerate([0.1, 1, 2]):
    rgb = ls.shade(dem[::-1], cmap=plt.cm.gist_earth, blend_mode='overlay',
                   vert_exag=0.5, dx=10, dy=10, fraction=fraction, vmin=-2300)

    
    ax = fig.add_subplot(1,3,i+1, projection=ccrs.PlateCarree())

    img = ax.imshow(rgb, extent=(lon.min(),lon.max(),lat.min(),lat.max()), transform=ccrs.PlateCarree())

    draw_map(get_map('河南'), color='w', linewidth=2)
    
    plt.title(f'fraction={fraction}')

    ax.set_extent(get_map('河南').get_extent(buffer=0))

plt.show()
```

![fraction](/assets/img/how-to-draw-pretty-hillshade-with-python/fraction.png)

可以看到当 `fraction=0.1` 时，几乎没有阴影效果，而 `fraction=2` 的时候，阴影效果很强烈，甚至感觉有点耀眼。

最后，让我们在一个动图效果下结束我们今天的讲解：

![precip](/assets/img/how-to-draw-pretty-hillshade-with-python/precip.gif)

上图展示了2021年7月20日郑州特大暴雨的逐小时降水量在一天中的分布变化，降水数据源是中国气象局的 CMPAS 格点降水产品，由于该数据非公开数据集，在此不提供数据下载。
