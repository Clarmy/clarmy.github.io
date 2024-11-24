---
title: Cartopy 是如何对 Matplotlib 做移花接木的
date: 2022-02-13 00:00:00 +08:00
modified: 2022-02-13 00:00:00 +08:00
tags: [python, cartopy]
description: 你有没有发现，当你用 Python 画图时使用 Cartopy 的投影参数以后，你的 `ax` 对象会莫名其妙多出好多新的方法，比如用于绘制岸线的 `ax.coastlines()` ，或者缩放到全球视角的 `ax.set_global()`。这些都是 Matplotlib 原生 `Axes` 对象所没有的方法，而你创建子图的时候，明明用的是 Matplotlib 原生的工厂函数创建的画轴啊，Cartopy 是怎么做到偷梁换柱，移花接木的呢？下面我们就来解密一下。
---

你有没有发现，当你用 Python 画图时使用 Cartopy 的投影参数以后，你的 `ax` 对象会莫名其妙多出好多新的方法，比如用于绘制岸线的 `ax.coastlines()` ，或者缩放到全球视角的 `ax.set_global()`。这些都是 Matplotlib 原生 `Axes` 对象所没有的方法，而你创建子图的时候，明明用的是 Matplotlib 原生的工厂函数创建的画轴啊，Cartopy 是怎么做到偷梁换柱，移花接木的呢？下面我们就来解密一下。

在之前写 [cnmaps](https://github.com/cnmetlab/cnmaps) 包的时候，我就大致读了一下 Cartopy 对 `GeoAxes` 对象的实现源码，其实 `GeoAxes` 就是 `Axes` 的一个增强子类，但是我很好奇，它是怎么实现通过传递一个参数而直接替换掉 Matplotlib 原生的 `Axes` 对象的，我们看下面的例子：

```python
$ ipython
Python 3.6.13 |Anaconda, Inc.| (default, Feb 23 2021, 12:58:59) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.16.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import matplotlib.pyplot as plt

In [2]: import cartopy.crs as ccrs

In [3]: fig1 = plt.figure()

In [4]: type(fig1.add_subplot())
Out[4]: matplotlib.axes._subplots.AxesSubplot

In [5]: fig2 = plt.figure()

In [6]: type(fig2.add_subplot(projection=ccrs.PlateCarree()))
Out[6]: cartopy.mpl.geoaxes.GeoAxesSubplot
```

从上面的例子可以看出，若我们调用 `add_subplot` 时不传入 `projection` 参数，那么返回结果的类型就是 Matplotlib 原生的 `AxesSubplot` 对象，而如果我们传入了 `projection` 对象，那么返回结果的类型就变成了 Cartopy 自己定义的 `GeoAxesSubplot` 类型。用一个参数就可以控制返回值类型，这么神奇的吗？

后来查阅了 Matplotlib 的官方文档，在[一个角落里](https://matplotlib.org/stable/api/projections_api.html)发现了这么一段描述：
> For more complex, parameterisable projections, a generic "projection" object may be defined which includes the method `_as_mpl_axes`. `_as_mpl_axes` should take no arguments and return the projection's Axes subclass and a dictionary of additional arguments to pass to the subclass' `__init__` method. Subsequently a parameterised projection can be initialised with:
> ```python
> fig.add_subplot(projection=MyProjection(param1=param1_value))
> ```
> where MyProjection is an object which implements a `_as_mpl_axes` method.

上面的大概意思是，对于复杂投影，你可以自定义一个投影对象，然后在类里实现一个无参数的 `_as_mpl_axes` 方法，这个方法的返回值应包括投影画轴的子类和一个准备继续传递给子类的参数字典，之后就可以用类似于 
```python
fig.add_subplot(projection=MyProjection(param1=param1_value))
``` 
的方式调用了。而且 `MyProjection` 应该是一个内置了 `_as_mpl_axes` 方法的对象。

原来是 Matplotlib 给大家留了后门，以扩展它的投影支持，其实 Matplotlib 原生支持一些投影，例如[这里](https://matplotlib.org/stable/gallery/subplots_axes_and_figures/geo_demo.html)，但是它显然还是想让专业的人做专业的事，所以它在页面里推荐了 Cartopy。

我们现在知道了 Matplotlib 留了后门，那具体 Cartopy 是怎么实现的呢？我们来看一下 Cartopy 在写投影的时候都写了写什么，以 0.19.0 版本为例：

```python
class Projection(CRS, metaclass=ABCMeta):
    ...
    def _as_mpl_axes(self):
        import cartopy.mpl.geoaxes as geoaxes
        return geoaxes.GeoAxes, {'map_projection': self}
    ...
```

由于篇幅问题，我仅截取了部分内容，完整代码请点击这里。Cartopy 在实现 `Projection` 类的时候，内置了 `_as_mpl_axes` 方法，且返回了它自定义的 `GeoAxes` 类，这个 `GeoAxes` 就是 Cartopy 的主角，它是 `matplotlib.axes.Axes` 的子类，它继承了 `Axes` 的属性和方法（当然也重写了一些方法），同时也新添加了一些特有的方法，例如 Cartopy 特有的 `coastlines()` ，`set_global()` 等方法就是在 `GeoAxes` 中定义的，想要查看 `GeoAxes` 的具体实现代码可以点击这里。

当 `Projection` 类内置了 `_as_mpl_axes` 方法以后，它就具有了一种移花接木的能力， `Axes` 对象在实例化的时候会根据 `_as_mpl_axes` 函数的返回值来决定自己的返回值。

说了这么多，不实操一下吗？当然要，下面我就用一个简单的例子，来创建一个自定义画轴。

```python
$ ipython
Python 3.6.13 |Anaconda, Inc.| (default, Feb 23 2021, 12:58:59) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.16.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import matplotlib.pyplot as plt

In [2]: import matplotlib

In [3]: class ClarmyAxes(matplotlib.axes.Axes):
   ...:     def __init__(self, *args, **kwargs):
   ...:         super().__init__(*args, **kwargs)
   ...: 

In [4]: class ClarmyProjection(object):
   ...:     def _as_mpl_axes(self):
   ...:         return ClarmyAxes, {}
   ...: 
```

上面就是仿照 Cartopy 的方式，我们自定义了一个 `ClarmyAxes` 和 `ClarmyProjection` ， `ClarmyProjection` 类内置 `_as_mpl_axes` 方法然后返回 `ClarmyAxes` 对象和空的参数，下面我们来实验一下它有没有生效。

```python
In [5]: fig = plt.figure()

In [6]: type(fig.add_subplot(111, projection=ClarmyProjection()))
Out[6]: matplotlib.axes._subplots.ClarmyAxesSubplot
```

我们发现它返回的是一个 `matplotlib.axes._subplots.ClarmyAxesSubplot` 对象，看起来是生效了，因为 Matplotlib 原先不可能存在一个 `ClarmyAxesSubplot` 子轴对象，但是它还是隶属于 `matplotlib.axes` 模块里，其实这是 Matplotlib 根据我们的 `ClarmyAxes` 临时创建起来的子轴，如果想要再定制化一些，可以像 Cartopy 一样加入下面这段代码：

```python
In [7]: ClarmyAxesSubplot = matplotlib.axes.subplot_class_factory(ClarmyAxes)

In [8]: ClarmyAxesSubplot.__module__ = ClarmyAxes.__module__
```
这样再做实例化的时候，它就与 Matplotlib 划清界限了。

```python
In [9]: fig = plt.figure()

In [10]: type(fig.add_subplot(111, projection=ClarmyProjection()))
Out[10]: __main__.ClarmyAxesSubplot
```

是不是还挺好玩的。