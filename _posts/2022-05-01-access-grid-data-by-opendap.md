---
title: 利用 OPeNDAP 快速获取格点数据
date: 2022-05-01 00:00:00 +08:00
modified: 2022-05-01 00:00:00 +08:00
tags: [Python, OPeNDAP, Panoply]
description: 国内的气象圈子对于 OPeNDAP 这个单词应该是既熟悉又陌生，熟悉就熟悉在它出现频率很高，感觉好像哪哪儿都提到了它；而陌生就陌生在平时实际工作中好像又很少真正用过它。事实上 OPeNDAP 是一个可以极大提高格点数据传输和使用效率的“工具”，当初我第一次体验这个东西的时候就发出了“卧槽还可以这样”的感慨。
---
国内的气象圈子对于 OPeNDAP 这个单词应该是既熟悉又陌生，熟悉就熟悉在它出现频率很高，感觉好像哪哪儿都提到了它；而陌生就陌生在平时实际工作中好像又很少真正用过它。事实上 OPeNDAP 是一个可以极大提高格点数据传输和使用效率的“工具”，当初我第一次体验这个东西的时候就发出了“卧槽还可以这样”的感慨。

当你解锁了 OPeNDAP 以后你会突然发现世界如此美好，格点数据的获取如此方便，并且 OPeNDAP 的内涵非常丰富，服务端有众多友好的实现方案，现成的 Docker 镜像，客户端的 Panoply、多语言生态下的库包甚至 GDAL 都原生支持了 OPeNDAP 协议，使用起来不要太方便。有些服务端实现方案还支持各种花里胡哨的玩法，比如可以在URL中传参数实现服务端直接切片。

今天这一篇是这个系列的第一篇，也算是入门篇。

## 什么是OPeNDAP？
简单介绍一下 OPeNDAP 的概念，OPeNDAP 是 Open-source Project for a Network Data Access Protocol 的缩写，直译过来就是 **开源网格数据获取协议**，所以说本质上它其实只是是一个纸面上的协议，或者可以理解为一种标准。那么任何基于该标准而实现的服务器或客户端程序，都可以享受 OPeNDAP 所带来的便利，它能带来什么便利呢？简单来说就是让你像使用本地文件一样使用远程的格点数据。也就省去了数据文件下载的过程，且基于 OPeNDAP 服务的数据加载模式属于“懒加载”，它只会读取你真正需要那一部分数据，从而避免你为了加载一个单一的要素而需要下载整个文件的行为，节约时间和带宽。

## GFS的OPeNDAP入口
国外的很多气象格点数据的下载网站都提供了 OPeNDAP 的接口，比如 NCEP 的 GFS 就给不同产品分别建立了 OPeNDAP 的获取入口，下面我就带大家实操一下如何获取数据的 URL 链接及其元信息。

进入GFS数据下载主页：[https://nomads.ncep.noaa.gov/](https://nomads.ncep.noaa.gov/)

![01](/assets/img/access-grid-data-by-opendap/01.png)

我们点开 *GFS 0.50 Degree* 对应的 OPeNDAP 链接。

![02](/assets/img/access-grid-data-by-opendap/02.png)

选择日期，比如我们选择2022年5月1日的。

![03](/assets/img/access-grid-data-by-opendap/03.png)

在时次选择时进入 info 链接，进入以后可以看到 OPeNDAP/DODS Data URL，这个 URL 就是可以用于直接打开的数据 URL。

![04](/assets/img/access-grid-data-by-opendap/04.png)

这一页内容很长，它是该时次数据的完整的元信息，你可以从这个页面查找到该数据以 OPeNDAP 协议暴露出去的数据变量名、维度及数组尺寸等信息。

## 如何用 Panoply 打开 OPeNDAP 数据
我们知道，如果想要快速查看 nc 数据，Panoply 是一个很好的选择，Panoply 是一个用 Java 写的气象数据可视化软件，支持多平台，非常好用。而且 Panoply 原生支持基于 OPeNDAP 的远程数据加载。

![05](/assets/img/access-grid-data-by-opendap/05.png)

我们打开Panoply以后，点击 **File** -> **Open Remote Dataset**

![06](/assets/img/access-grid-data-by-opendap/06.png)

然后把刚才的数据URL粘贴进去，点击 **Load**

![07](/assets/img/access-grid-data-by-opendap/07.png)

这样我们就在 Panoply 通过 OPeNDAP 远程的方式”加载“了这个数据集。但这个时候的加载其实只是加载了数据的元信息，并没有把整个数据集全部下载下来（也就是“懒加载”），所以这一步速度很快。

后面我们可以根据自己的需要，仅加载某一个要素，比如我们选择2米气温，在这里它的变量名为 **tmp2m**。

**注意，这里的变量名既不是GRIB标准变量名也不是ecCodes的变量名，更不是CF-convention的标准变量名，而是他们自己重新定义的，变量名的含义可以在前面提到的 info 页面里查到**

![09](/assets/img/access-grid-data-by-opendap/09.png)

根据我们请求，Panoply 可以给我们生成预览图，同时可以看到该变量已将预报时次集成在图层里，我们可以在这里方便地选择不同时次生成预览图。而且每次切换的过程都是惰性的，只会加载所选时次而非全部时次的数据。

## 在 Python 中使用 OPeNDAP
Python 对 OPeNDAP 的使用是基于相应的第三方库实现的，可供使用的库包括：netcdf4、pydap 和 xarray，其中 xarray 是基于 netcdf4/pydap 作为底层引擎实现的更高级封装。

三种方法加载 OPeNDAP 的操作其实就是把本地路径地址替换成 URL，其他都跟平时使用没啥区别。

```python
import netCDF4 as nc  # pip install netCDF4
import xarray as xr  # pip install array
from pydap.client import open_url  # pip install requests pydap

url = 'http://nomads.ncep.noaa.gov:80/dods/gfs_0p25_1hr/gfs20220501/gfs_0p25_1hr_06z'

ds0 = nc.Dataset(url)
ds1 = xr.open_dataset(url, engine='netcdf4', decode_times=False)
ds2 = xr.open_dataset(url, engine='pydap', decode_times=False)
ds3 = open_url(url)
```

这三种方法我个人还是更倾向于使用 netCDF4 这个库来读取数据，当然 xarray 肯定更流行一些，用得人也更多，而 pydap 用的人相对会比较少。

据我所知，其他语言对 OPeNDAP 的支持也很完善，包括但不限于 NCL、MATLAB、Java、Fortran，感兴趣的可以自己去查一下相关资料，我这里就不再赘述。

## OPeNDAP 能给我们带来的便利
在没有解锁 OPeNDAP 之前，我们在查看一个体积很大的在线 nc 文件，可能就需要把整个文件都下载下来，然后用 Panoply 或者程序解析查看，如果我们原本只是想要查看其中的一个或很少几个要素，我们也还是需要把整个文件都下载下来，这种做法一方面会浪费很多时间和带宽资源，另一方面，我们还需要考虑本地存储全量要素是否能够存得下的问题，当然 GFS 的 grib 可以提供 filter 的方法（底层是 cgi 程序，一种服务端古老的土豪方案）挑选变量，但是如果是其他的数据集不提供这种功能怎么办呢？

OPeNDAP 的懒加载可以让我们最大限度降低等待时间，数据根据需要去最小可用部分，提高了使用效率，降低了客户端和服务端的带宽需求，一举多得，实现双赢。因此国外的很多气象格点数据的下载网站都会提供一个 OPeNDAP 的入口，大家以后再下载格点数据的时候可以留意一下是否有这样的入口，尽量使用 OPeNDAP 来读取数据。