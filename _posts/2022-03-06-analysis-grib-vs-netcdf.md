---
title: GRIB 与 NetCDF 数据格式的特点及性能对比
date: 2022-03-06 11:58:47 +08:00
modified: 2022-03-06 11:58:47 +08:00
tags: [GRIB, NetCDF]
description: 在气象行业里，GRIB 和 NetCDF 文件格式可谓家喻户晓，但是这两种文件格式到底有哪些优势或劣势？在工程实践上两种格式各适合于什么样的应用场景以及性能如何？这些问题似乎在国内的气象技术圈子里很少见到有人讨论，今天我就结合我这些年来和这两种格式打交道的经验，跟大家简单分享一下这两种数据格式的特点及性能对比，供各位参考。
---
在气象行业里，GRIB 和 NetCDF 文件格式可谓家喻户晓，但是这两种文件格式到底有哪些优势或劣势？在工程实践上两种格式各适合于什么样的应用场景以及性能如何？这些问题似乎在国内的气象技术圈子里很少见到有人讨论，今天我就结合我这些年来和这两种格式打交道的经验，跟大家简单分享一下这两种数据格式的特点及性能对比，供各位参考。

## GRIB 的基本特点
GRIB 文件是由一种名为 Message 的自描述数据单元组成的二进制文件集合，而且它是由多个 Message 直接拼接而成的，是一种散装的数据格式。甚至一个GRIB 文件内可以同时包含 GRIB1 格式和 GRIB2 格式的 Message，这也就是为什么我们通常看到的 ECMWF 和 GFS 的 GRIB 文件没有像常规的数据文件一样带有后缀名的原因。

由于 GRIB 格式自描述的基本单位是 Message，所以它可以以 Message 作为基本单元进行排列组合而不会影响使用，比如说，只要你恰如其分地从两个相邻的Message 中间把一个 GRIB 文件切开，那么它就会直接变成两个 GRIB 文件，就像蚯蚓一样。同样地，你把两个 GRIB 文件直接拼接到一起，它又会直接形成了一个大的 GRIB 文件。

这种特征给它带来的最大好处是：灵活，非常有利于网络传输。

举一个 GFS 的例子，众所周知，GFS 数据集非常的庞大，单个时次的文件能达到好几百 GB，如此庞大的数据量的下载是一个挑战。而且 GFS 输出的要素太多，对于一些只需要几个要素的需求，如果我们把它整个数据集全部下载下来对磁盘和带宽资源就太浪费了。所以我们可能需要一种方法可以切出其中一部分要素要素来下载，这时候 GRIB 文件格式的散装特征就发挥作用了，NCEP 官方在发布 GFS 数据文件时，都会附带一个 .idx 的索引文件，它会记录每一条 Message 的字节范围，而 HTTP 协议天然支持截断请求，那么你只需要根据 .idx 文件计算出自己想要的变量，然后在用 HTTP 请求的时候传入请求的字节范围即可。

然而，GRIB 的散装结构也不都是优点，它也有它的问题，那就是文件校验会比较麻烦。通常情况下，我们在使用其他常见的二进制文件的时候，如果文件损坏，那么它就无法正常打开，例如一个损坏的 .docx 文件，你是无法用 Word 正常打开的，但 GRIB 与众不同，如果你在下载 GRIB 文件的时候，有几个字节没有下载完整，它仅会影响它所在的那个 Message，其他 Message 可以正常使用，当你没有读到残缺的 Message 时，它看起来就是个正常的文件，所以你就无法通过文件是否正常打开来判断文件是否完整，还需要结合字节数的对比以及重建索引等操作才能更有把握地判断文件的完整性。


GRIB 在官方介绍的时候，自诩为一种紧凑且体积小的数据格式，但是事实上这一所谓的“优势”早就已经被 NetCDF4 格式按在地上摩擦了，而且它的文件读取效率也远低于 NetCDF4，基本上已经算是被 NetCDF4 全方位超越和碾压了，这一点后面我们会讲到。

## NetCDF 的基本特点
说到 NetCDF 格式，不得不把 NetCDF3 和 NetCDF4 分开来说。虽然它们都是 NetCDF 家族的成员，但是它们基本上算是两个物种。你知道为什么 GRIB 会自诩为紧凑而小巧的数据格式吗？那因为它在和 NetCDF3 做对比，跟 GRIB 的文件大小相比，同要素的 NetCDF3 格式的文件体积确实很大，不吹不黑基本2倍起步。

但是后来，NetCDF4 出来了，NC4 完全舍弃了 NC3 的底层架构，而采用了 HDF5 的底层，或者换句话说，其实NC4就是一种特殊的 HDF5，它俩的关系就类似于 geojson 与 json 之间的关系，你甚至可以直接用 h5py 去打开 nc4 的文件，或者用 NetCDF4 的包去读 .h5 文件。

NC4 对于 NC3 是全方位的提升，NC4 文件支持压缩，其压缩比相当的优秀，并且压缩后的文件可以直接读取也不影响速度，至于为什么HDF5的性能会这么好，其实我也没仔细研究过。

上面讲了这么多，都是空口白牙，没有证据，下面我就用几个实际案例来证实上面的这些结论。

## 文件体积对比
下面演示的例子所使用的数据，一部分来自 ERA5，一部分来自 GFS。ERA5 的数据我是下载了地面层13个要素的单一时次的 GRIB 和 NetCDF 文件，为了公平起见，两种格式就是在官网下载文件时分别选 GRIB 和 NetCDF 输出格式下载的，不是我自己转换的。而 GFS 数据仅用于演示 pygrib 的两种读取效率，并没有参与到与 NetCDF 格式的性能比较，这些数据都上传到和鲸社区了，可以去那里下载。

我们现在有两个文件，分别是 era5-sample.grib 和 era5-sample.nc3。


```bash
$ ls -lh era5-sample*
-rw-r--r--  1 clarmylee  staff    80M  3  5 19:26 era5-sample.grib
-rw-r--r--@ 1 clarmylee  staff   161M  3  5 19:29 era5-sample.nc3
```

我们把 GRIB 文件转为 GRIB2 存一份：
```bash
$ grib_set -s edition=2 era5-sample.grib era5-sample.grib2
$ ls -lh era5-sample.grib*
-rw-r--r--  1 clarmylee  staff    80M  3  5 19:26 era5-sample.grib
-rw-r--r--  1 clarmylee  staff    80M  3  6 02:14 era5-sample.grib2
```

可以看到 GRIB 和 GRIB2 的体积几乎没有变化。而在初始的状态下，NC3 的文件是 GRIB 文件的2倍。这也是 GRIB 曾经引以为豪的资本，但如果我们用 bzip2 对二者进行压缩的话：
```bash
$ bzip2 -k era5-sample.nc3
$ bzip2 -k era5-sample.grib
$ ls -lh era5-sample*
-rw-r--r--  1 clarmylee  staff    80M  3  5 19:26 era5-sample.grib
-rw-r--r--  1 clarmylee  staff    52M  3  5 19:26 era5-sample.grib.bz2
-rw-r--r--@ 1 clarmylee  staff   161M  3  5 19:29 era5-sample.nc3
-rw-r--r--  1 clarmylee  staff    32M  3  5 19:29 era5-sample.nc3.bz2
```