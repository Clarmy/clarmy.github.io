---
title: 一份 GRIB 与 NetCDF 的体积对比测试报告
date: 2022-03-12 00:00:00 +08:00
modified: 2022-03-12 00:00:00 +08:00
tags: [GRIB, NetCDF]
description: 由于前面写的一篇关于 GRIB 与 NetCDF 的文章《浅析GRIB与NetCDF数据格式的特点及性能对比》在GRIB2与NetCDF4文件体积问题上有一些错误结论，经朋友指正，因此专门做此文予以更正。
---
由于前面写的一篇关于 GRIB 与 NetCDF 的文章[「GRIB 与 NetCDF 数据格式的特点及性能对比」]({% post_url 2022-03-06-analysis-grib-vs-netcdf %}) 在 GRIB2 与 NetCDF4 文件体积问题上有一些错误结论，经朋友指正，因此专门做此文予以更正。

## 摘要
本文利用工具将一个原始 ERA5 的 GRIB1 格式文件转换为不同压缩算法下的 GRIB2、NetCDF3 以及不同压缩等级下的 NetCDF4 格式文件，同时对他们进行 bz2 压缩，最后对比它们的文件体积大小。最终的结论是，不论是否使用 bz2 压缩，体积最小的都是 GRIB2 家族的文件格式。

## 数据简介
本次测试使用的原始数据为通过ERA5再分析数据下载的 GRIB 文件，该原始数据为 GRIB1 格式，而为了对比不同格式下的体积，我会用一些工具将该数据进行格式转换。

原始数据下载地址：[https://doi.org/10.5281/zenodo.6348679](https://doi.org/10.5281/zenodo.6348679)

## 使用工具及方法
本文使用的命令行工具包括：

1. ecCodes
2. wgrib2
3. netCDF4
4. bzip2
   
其中 ecCodes 和 netCDF4 都可以直接使用 conda 进行安装，而 bzip2 和 wgrib2 需要根据自己的系统查询对应的安装方法在这里不再赘述，本文使用 wgrib2 是通过 docker 启动对应镜像的方式进行的: `$ docker pull agilesrc/wgrib2`

## 使用wgrib2设置grib2压缩格式的方法
wgirb2可以使用的压缩方法包括：
* ieee : data is ieee format (4 bytes per data point)
* simple : no compression, packed scaled integers
* complex1 : complex packing
* complex1-bitmap : complex and using bitmap for undefined grid points
* complex2 : complex packing, pack increments (deltas)
* complex2-bitmap : complex packing, pack increments (deltas) and using bitmap for undefined
* complex3 : complex packing, pack increments after linear extrapolation
* complex3-bitmap : complex packing, pack increments after linear extrapolation and using bitmap for undefined
* jpeg : jpeg2000 compression
* aec : aec/CCSDS compression
* same : try to keep same packing type as input, if input is in an unsupported output packing, complex1 is used

示例：
```
$ wgrib2 in.grb -set_grib_type complex3 -grib_out out.grb 
```
相关文档链接：[https://www.cpc.ncep.noaa.gov/products/wesley/wgrib2/set_grib_type.html](https://www.cpc.ncep.noaa.gov/products/wesley/wgrib2/set_grib_type.html)

## 使用wgrib2将grib2转化为netcdf格式的方法
示例:
```
$ wgrib2 /assets/img/a-size-comparison-of-grib-and-netcdf/in.grb2 -netcdf out.nc
```
相关文档链接：[https://www.cpc.ncep.noaa.gov/products/wesley/wgrib2/netcdf.html](https://www.cpc.ncep.noaa.gov/products/wesley/wgrib2/netcdf.html)

## 使用grib_set对grib1和grib2格式进行切换的方法
`grib_set` 是 ecCodes 命令行工具包中的一个命令行工具，用于修改 grib 文件的属性，也可以对 grib1 和 grib2 格式进行切换，然而该命令行并不能转换所有的 grib 文件，例如 GFS 的 grib2 文件有时就无法转换。

ecCodes的安装方法：
```
$ conda install -c conda-forge -y eccodes
```
示例：
```bash
# 将girb1转换为grib2
$ grib_set -s edition=2 in.grib out.grib2

# 将grib2转换为grib1
$ grib_set -s edition=1 in.grib2 out.grib1
```
## 使用nccopy转换netcdf文件格式及压缩格式的方法
`nccopy` 是 netCDF4 命令行工具包里的一个命令行工具，它可以实现nc文件的各种格式转换。

netcdf4 的安装方法:

```bash
$ conda install -c conda-forge -y netCDF4
```

根据 nccopy 的文档，与转换格式和压缩相关的参数包括：
> [-k kind] specify kind of netCDF format for output file, default same as input kind strings: ‘classic’, ’64-bit offset’, ‘cdf5’, ‘netCDF-4’, ‘netCDF-4 classic model’
> 
> [-d n] set output deflation compression level, default same as input (0=none 9=max)

示例：
```bash
# 将netcdf3文件转换为`netCDF-4 classic model`模式的netcdf4格式，并使用1级压缩
$ nccopy -k 'netCDF-4 classic model' -d 1 era5-sample.nc3 era5-sample-c0.nc4
```
## GRIB1 与 GRIB2 的体积对比
GRIB1 格式是不支持压缩的，而 GRIB2 格式支持压缩，因此我们对比 GIRB1 和 GRIB2 格式文件的体积，本质上是对比 GRIB1 与各种压缩方式下 GRIB2 文件的体积。

由于我们的原始文件 `era5-sample.grib` 是 GRIB1 格式，我们先把它转换为 GRIB2，在终端执行:
```bash
$ grib_set -s edition=2 era5-sample.grib era5-sample.grib2
```
查看新的 GRIB2 文件:
```bash
$ grib_ls era5-sample.grib2
era5-sample.grib2
edition      centre       date         dataType     gridType     stepRange    typeOfLevel  level        shortName    packingType
2            ecmf         20211028     fc           regular_ll   8            heightAboveGround  10           10u          grid_simple
2            ecmf         20211028     fc           regular_ll   8            heightAboveGround  10           10v          grid_simple
2            ecmf         20211028     fc           regular_ll   8            heightAboveGround  2            2d           grid_simple
2            ecmf         20211028     fc           regular_ll   8            heightAboveGround  2            2t           grid_simple
2            ecmf         20211028     fc           regular_ll   8            surface      0            fal          grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            slhf         grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            ssr          grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            str          grid_simple
2            ecmf         20211028     fc           regular_ll   8            surface      0            sp           grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            sshf         grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            ssrd         grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            strd         grid_simple
2            ecmf         20211028     fc           regular_ll   7-8          surface      0            tp           grid_simple
13 of 13 messages in era5-sample.grib2

13 of 13 total messages in 1 files
```
可以看到 GRIB 的 edition 已经变成2了，下面我们要对转换后的 GRIB2 文件进行压缩，在预装了 wgrib2 并支持 bash 命令的环境下，在数据目录运行以下脚本：
```shell
#!/bin/sh  

ctypes=( "ieee" "simple" "complex1" "complex2" "complex3" "jpeg" "aec" "same" )  


for ctype in "${ctypes[@]}"
do
    wgrib2 -set_grib_type $ctype era5-sample.grib2 -grib_out era5-sample-$ctype.grib2
done
```
从而获得以下文件:
```bash
$ ls -lh era5-sample-*.grib2
-rw-r--r--  1 clarmylee  staff    36M  3 12 14:02 era5-sample-aec.grib2
-rw-r--r--  1 clarmylee  staff    34M  3 12 14:01 era5-sample-complex1.grib2
-rw-r--r--  1 clarmylee  staff    27M  3 12 14:01 era5-sample-complex2.grib2
-rw-r--r--  1 clarmylee  staff    27M  3 12 14:02 era5-sample-complex3.grib2
-rw-r--r--  1 clarmylee  staff   322M  3 12 14:01 era5-sample-ieee.grib2
-rw-r--r--  1 clarmylee  staff    36M  3 12 14:02 era5-sample-jpeg.grib2
-rw-r--r--  1 clarmylee  staff    65M  3 12 14:02 era5-sample-same.grib2
-rw-r--r--  1 clarmylee  staff    65M  3 12 14:01 era5-sample-simple.grib2
```
下面我们用代码统计一下这些文件的大小并绘制图形

![1](/assets/img/a-size-comparison-of-grib-and-netcdf/1.png)

可以看出来体积最大的压缩格式为 ieee，体积最小的为 complex3，而原始的 GRIB1 格式的文件是除了 ieee 以外体积最大的，使用 `grib_set` 直接转换后的 GRIB2 文件比原始文件略小，而 complex3 压缩格式的文件大约是原始文件格式的1/3左右。

以上 GRIB 文件是在不丧失直接读取能力的存储格式，下面我们再来测试一下，将他们压缩为 .bz2 格式以后的体积，在终端执行 `$ bzip2 -k *grib*` ，可以得到以下文件:
```bash
$ ls -lh *.bz2
-rw-r--r--  1 clarmylee  staff    25M  3 12 14:02 era5-sample-aec.grib2.bz2
-rw-r--r--  1 clarmylee  staff    33M  3 12 14:01 era5-sample-complex1.grib2.bz2
-rw-r--r--  1 clarmylee  staff    26M  3 12 14:01 era5-sample-complex2.grib2.bz2
-rw-r--r--  1 clarmylee  staff    26M  3 12 14:02 era5-sample-complex3.grib2.bz2
-rw-r--r--  1 clarmylee  staff    55M  3 12 14:01 era5-sample-ieee.grib2.bz2
-rw-r--r--  1 clarmylee  staff    26M  3 12 14:02 era5-sample-jpeg.grib2.bz2
-rw-r--r--  1 clarmylee  staff    31M  3 12 14:02 era5-sample-same.grib2.bz2
-rw-r--r--  1 clarmylee  staff    31M  3 12 14:01 era5-sample-simple.grib2.bz2
-rw-r--r--  1 clarmylee  staff    52M  3 12 13:58 era5-sample.grib.bz2
-rw-r--r--  1 clarmylee  staff    52M  3 12 13:58 era5-sample.grib2.bz2
```
![2](/assets/img/a-size-comparison-of-grib-and-netcdf/2.png)
可以看出来，经过 bz2 压缩以后文件体积最小的是基于 aec 方法压缩的文件，bz2 压缩效果最明显的是 ieee，而原先体积较小的文件经 bz2 算法压缩后的效果甚微。

综上可以看出，在 GRIB 的生态下，单纯从降低文件体积的角度考虑，在不丧失读取能力的情况下，使用 complex3 压缩算法的 GRIB2 格式进行存储是最优的方案，若可以接受读取能力丧失的情况，那么 aec 压缩算法也可以考虑。

## NetCDF3 与 NetCDF4 的体积对比
下面我们来讨论一下 NetCDF3 和 NetCDF4 之间的文件体积对比，跟 GRIB 类似的是，旧版本的 NetCDF3 不支持原生压缩，如果要压缩需要借助类似于 bz2 的工具进行，而新版本的 NetCDF4 支持原生压缩，因此对两种格式的对比实际上就是用 NetCDF3 与 NetCDF4 不同压缩等级之间的对比。

grib_to_netcdf 命令支持将 GRIB 转换为下列四种 NetCDF 存储格式：

* netCDF classic file format
* netCDF 64 bit classic file format (Default)
* netCDF-4 file format
* netCDF-4 classic model file format

以上四种数据格式的底层差异我们在这里不做赘述，仅比较它们的体积，我们先分别将 GRIB 文件转为这4种 nc 格式。
```bash
$ grib_to_netcdf -k 1 -o era5-sample-class.nc3 era5-sample.grib
$ grib_to_netcdf -k 2 -o era5-sample-64class.nc3 era5-sample.grib
$ grib_to_netcdf -k 3 -o era5-sample.nc4 era5-sample.grib
$ grib_to_netcdf -k 4 -o era5-sample-class.nc4 era5-sample.grib
$ ls -lh *nc*
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:26 era5-sample-64class.nc3
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:26 era5-sample-class.nc3
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:27 era5-sample-class.nc4
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:26 era5-sample.nc4
```
可以看到，直接使用 grib_to_netcdf 命令转换的各种 NetCDF 格式的体积没有显著差异，下面我们先使用 nccopy 对其中 nc4 进行原生压缩，执行以下脚本:

```bash
#!/bin/sh  

clevels=( 0 1 2 3 4 5 6 7 8 9 )  


for level in "${clevels[@]}"
do
    nccopy -k 'netCDF-4' -d $level era5-sample.nc4 era5-sample-c$level.nc4
done
```
检查结果:
```bash
$ ls -lh era5-sample-*nc*
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:26 era5-sample-64class.nc3
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:44 era5-sample-c0.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c1.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c2.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c3.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c4.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c5.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c6.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c7.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c8.nc4
-rw-r--r--  1 clarmylee  staff    44M  3 12 15:44 era5-sample-c9.nc4
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:26 era5-sample-class.nc3
-rw-r--r--  1 clarmylee  staff   161M  3 12 15:27 era5-sample-class.nc4
```
可以看到，在无压缩的状态下，nc4 的体积为161M，而使用1-9级压缩后的结果都是44M，也就是说 NetCDF4 格式下，压缩与非压缩的结果差异很大，而不同等级之间的压缩差异很小，并且非压缩的 NetCDF4 与 NetCDF3 的体积一样大。

我们再对其进行 bz2 压缩，执行 `$ bzip2 -k era5-sample-*nc*`
然后画出图来看一下：
![3](/assets/img/a-size-comparison-of-grib-and-netcdf/3.png)
从上图可以看出来，按照非 bz2 算法压缩的形式排序，体积最大的是未经任何压缩的 NetCDF4 格式的文件，而体积最小的是9级压缩的 NetCDF4 格式。
![4](/assets/img/a-size-comparison-of-grib-and-netcdf/4.png)
若按照 bz2 压缩之后的文件体积来看的话，NetCDF3 经 bz2 压缩以后会比 NetCDF4 的体积更小。
## 4种格式体积的交叉对比
下面综合上面所有的压缩或不压缩以及不同压缩级别的格式，汇总在一起来看一下它们的体积对比
![5](/assets/img/a-size-comparison-of-grib-and-netcdf/5.png)
上图是按照非 bz2 压缩体积从大到小排序的，可以看出来的是，按照不丧失可读性的原始文件排序，依然是 complex3 压缩算法下的 GRIB2 格式的文件体积最小。
![6](/assets/img/a-size-comparison-of-grib-and-netcdf/6.png)
而使用 bz2 压缩后的格式排序，体积最小的依然是 aec 压缩算法下的 GRIB2 格式。

## 结论
从上述的对比中，我们可以得出结论，不论是否使用 bz2 压缩，体积最小的都是 GRIB2 家族的文件格式，在非 bz2 模式下体积最小的是基于 complex3 压缩算法的 GRIB2 文件，在 bz2 模式下体积最小的是基于 aec 压缩算法的 GRIB2 文件，当然这仅仅是从体积的角度来考虑。若要考察读取速度，则还需要额外的实验和测试。

若要引用本文，请使用以下引用格式：

> Wentao Li. (2022). 一份GRIB与NetCDF的体积对比报告 (Version v1). Zenodo. https://doi.org/10.5281/zenodo.6348695
