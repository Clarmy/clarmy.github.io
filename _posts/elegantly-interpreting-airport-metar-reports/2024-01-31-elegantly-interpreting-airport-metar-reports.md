---
title: 优雅地解译机场 METAR 报文
date: 2024-01-31 00:00:00 +08:00
modified: 2024-01-31 00:00:00 +08:00
tags: [Python, METAR]
description: 当我们想要获取气象观测数据的时候，除了抓取气象局的网站以外，还可以考虑抓取民航的航空例行天气报告（METAR），METAR 报告是隶属于民航空管局或机场的气象部门（非气象局序列）对机场天气状况的观测报告。这类报告不仅囊括了常规的天气要素，还会对影响飞机起降的特殊天气现象做额外的观测和报告。其观测质量并不逊色于气象局的站点观测，甚至在特殊天气现象的观测质量上要比气象局的观测结果更准确。
---
当我们想要获取气象观测数据的时候，除了抓取气象局的网站以外，还可以考虑抓取民航的航空例行天气报告（METAR），METAR 报告是隶属于民航空管局或机场的气象部门（非气象局序列）对机场天气状况的观测报告。这类报告不仅囊括了常规的天气要素，还会对影响飞机起降的特殊天气现象做额外的观测和报告。其观测质量并不逊色于气象局的站点观测，甚至在特殊天气现象的观测质量上要比气象局的观测结果更准确。

而且 METAR 报是全球公开共享的公共数据，在网络上很容易获取。比如我们可以从美国的航空天气网 [https://aviationweather.gov](https://aviationweather.gov) 实时获取全球任何国家的国际机场（4F级）的最新 METAR 报文。而民航机场的地理坐标也是公开的，很方便对观测数据的地理坐标做校准。因此 METAR 报文是气象实况观测数据的一个很好的补充，甚至美国的气象预报测评网站 ForecastWatch 也主要使用机场 METAR 报文解析后的数据作为测评的真值。

METAR 报文有着全球统一的格式标准，任何熟悉该格式规则的人，都可以很容易地将原始报文解译成真实的观测数据。当然，为了能更简单和优雅地解译报文，有一些工具可以帮助我们做这一类工作。比如 `metpy.io.parse_metar_to_dataframe`，可以把原始报文解译并构建成 DataFrame 对象，有兴趣的朋友可以自己去了解这个函数。

今天我想要介绍的还是自己写的一个解译 METAR/TAF 报文的 Python 扩展包：pymetaf。这个包是很多年前由于工作需要而临时开发的，当时 metpy 还没有开发相关功能的函数。现在虽然 metpy 有解析 METAR 的功能，但是由于 metpy 的 METAR 解译函数在单位等方面并不适合我们国内的使用习惯，因此近期我对 pymetaf 做了一些通用化的调整，让它的解译结果更适合我们中国宝宝体质。

我们可以使用 pip 来安装 pymetaf:
```bash
$ pip install pymetaf
```

解译的方法也很简单，我们来看一个简单的例子：

```python
from pymetaf import parse_text

# 一条首都机场在2021年5月发出的真实 METAR 报文
metar_string = "METAR ZBAA 311400Z 01002MPS CAVOK 14/12 Q1009 NOSIG="

# 由于 METAR 原始报文不包含年和月的信息，需要手动传入年月信息
data = parse_text(metar_string, 2021, 5)
```

解译后的结果：
```json

{
    "kind": "METAR",
    "icao": "ZBAA",
    "datetime": "2021-05-31T14:00:00+00:00",
    "wind_direction": 10,
    "wind_direction_units": "degree",
    "wind_speed": 2,
    "wind_speed_units": "m/s",
    "gust": null,
    "wind_direction_range": null,
    "cavok": true,
    "visibility": 99999,
    "visibility_units": "m",
    "temperature": 14,
    "dew_temperature": 12,
    "temperature_units": "degree C",
    "qnh": 1009,
    "qnh_units": "hPa",
    "cloud": null,
    "weather": [
        "Clear Sky"
    ],
    "auto": false
}
```

结果是以字典的形式存储的，对于一些未观测或者省略的字段，用空值来填充了。由于该报文发出了 CAVOK 的标志，意思是好天气，因此 visibility 的值会被处理为 99999，它代表的不是缺省，而是能见度无限好。

除了这种很常规的报文解译，它还可以解译一些相对比较特殊的报文结果，例如下面这条报文：
>METAR RCKH 160600Z VRB03KT 2500 SHRA BR SCT003 BKN006 OVC020 26/26 Q1008 TEMPO 5000 -SHRA RMK A2977 RA AMT 5.4MM=

这条来自我国台湾地区的高雄国际机场（2021年5月16日），可以看到它与首都机场有一个明显不同的地方是风观测组：VRB03KT，而首都机场的风观测组是 01002MPS。这里第一个不同是单位，高雄机场使用的单位是 KT，即“节”(knot)。在我们的使用习惯里，作为公制单位的 MPS(m/s) 才是主流，因此 pymetaf 会对其做一次单位转换。第二个不同，VRB03KT 中的 VRB 表示不定风向，也就是风速太小了风向不确定或者风向变化过快没有主导风向，对于这种情况 pymetaf 会把风向的结果输出为 None。
```json
{
    "kind": "METAR",
    "icao": "RCKH",
    "auto": false,
    "datetime": "2021-05-16T06:00:00+00:00",
    "wind_direction": null,
    "wind_direction_units": "degree",
    "wind_speed": 1,
    "wind_speed_units": "m/s",
    "gust": null,
    "wind_direction_range": null,
    "visibility": 2500,
    "visibility_units": "m",
    "cavok": false,
    "temperature": 26,
    "dew_temperature": 26,
    "temperature_units": "degree C",
    "qnh": 1008,
    "qnh_units": "hPa",
    "cloud": [
        {
            "cloud_height": 120,
            "cloud_height_units": "m",
            "cloud_mask": 0.75,
            "cloud_type": null
        },
        {
            "cloud_height": 400,
            "cloud_height_units": "m",
            "cloud_mask": 1,
            "cloud_type": null
        },
        {
            "cloud_height": 60,
            "cloud_height_units": "m",
            "cloud_mask": 0.5,
            "cloud_type": null
        }
    ],
    "weather": [
        "Showers of Rain",
        "Mist"
    ],
}
```

这一条解译的结果多了一些，由于我们对 METAR 解析的首要目标是观测而非预测，所以 pymetaf 会把趋势报（TEMPO后面的部分）剪掉，然后解译前面的部分，天气现象部分，SHRA 对应 Showers of Rain（阵雨），BR 对应 Mist（轻雾）；云组部分，根据 METAR 的标准规范反算出以米为单位的云高 cloud_height，及云覆盖度 cloud_mask，由于原始的云覆盖度是以 FEW/SCT/BKN/OVC 等字符编码表示，它们各自对应一个8分位的覆盖度范围，pymetaf 会对其进行一个标准化处理，输出为 0-1 的小数，以适应常规气象数据的使用习惯。如果报文中出现带有云类型表示的云组，例如 SCT030CB，则它可以解译出 `"cloud_type": "cumulonimbus"`。

另外，对于一些自动观测的报文，或者使用英里作为能见度单位的报文，pymetaf 也能轻松处理：

> METAR ZYQQ 081700Z AUTO /////MPS //// // ////// M05/M07 Q1006
> 
> METAR KSUA 300715Z AUTO 10SM CLR 12/08 A3013 RMK AO2 T01170076

当然，虽然我为该项目编写了一些测试用例，但是依然不能保证遇到一些更特殊的报文会触发某些 BUG。也希望读者朋友们如果使用的话，遇到 BUG 可以给我提一些 Issue。

项目地址：[https://github.com/cnmetlab/pymetaf](https://github.com/cnmetlab/pymetaf)

