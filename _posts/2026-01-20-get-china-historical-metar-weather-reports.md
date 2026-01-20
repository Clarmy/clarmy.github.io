---
title: 如何获取中国机场长周期高质量的历史 METAR 报文数据
date: 2026-01-20 17:15 +08:00
modified: 2026-01-20 17:15 +08:00
tags: [Metar, China, Historical Data]
description: "详细介绍如何获取中国机场2000-2025年长周期高质量历史METAR报文数据，对比分析IEM、ogimet和EOL数据源优劣，推荐使用观天者数据站免费查询下载40+中国机场历史气象观测数据"
comments: true
---
之前我在 [ *「推荐几个民航机场 METAR 报文的数据源」* ]({% post_url 2024-02-03-recommend-metar-data-sources %}) 一文中介绍了几个获取民航机场 METAR 报文数据源的网站，其中大部分只能获取距离当前时间较近的 METAR 报文数据。如果想要获取更长时间的历史 METAR 报文数据，只有爱荷华州立大学 IEM 的网站和 ogimet.com 可以获取。

然而当我真的去尝试使用这两个网站获取长周期历史 METAR 数据的时候却遇到几个问题：

1. 爱荷华州立大学 IEM 的数据和 ogimet 的数据有时候会出现互相矛盾的情况。
2. 爱荷华州立大学 IEM 的数据在比较久远的历史数据（比如2010年以前）里会出现很多看起来很奇怪的数据（有 AUTO 标识，用英制单位，感觉像是基于某种算法填充的，可靠性存疑）。
3. ogimet 数据对于数据查询有频率限制，对于长周期数据量的需求来说，获取起来很不方便。
4. ogimet 也存在很多脏数据。

后来，经过多方查找，我又找到一个存储了历史数据的数据源：[NSF NCAR Earth Observing Laboratory (EOL)](https://doi.org/10.26023/YHA4-K6FJ-W005)（以下简称 EOL，国内访问须科学上网），它提供了更完整的全球长周期历史 METAR 观测的原始报文数据。

EOL 数据是简单粗暴地直接存储从 2000 年 3 月至 2025 年 11 月各个气象中心数据交换过程中收发的最原始报文。需要提交申请，经对方人工审核通过之后才发送下载链接，全部数据文件一共约 240 多 GB（压缩后）。原始数据没有任何预处理，格式并不整齐，需要花费一定的时间做数据清洗、整理和质控才能放心使用。

示例：
```
210 

SAIE32 EIDB 110700

METAR EIDL 110700Z NIL=
METAR EIKY 110700Z 20002KT 9999 BKN036 04/03 Q1007=
METAR EIME 110700Z 09003KT 040V140 5000 BR BKN004 SCT017 BKN044 06/06
     Q1006=
METAR EISG 110700Z NIL=
METAR EIWF 110700Z NIL=




306 

SAMZ40 FQMA 110700

METAR FQXA 110700Z 15007KT 120V180 9999 BKN020 28/25 Q1019=
```
在拿到数据文件之后，我们决定将这份长周期历史数据整理出来，并和 ogimet/IEM 数据进行比对分析和取舍，最终输出一份高质量的中国机场长周期历史 METAR 报文数据集。

同时，为了方便用户使用，我们开发了界面友好使用简单的数据查询网站 [观天者数据站](https://datastation.skyviewor.com/)，目前该数据集已在网站上线，并成为该平台网站的第一个上线的数据集。该数据集除了可以提供历史的 METAR 数据，同时还从全网收集并实时更新提供最新的 METAR 报文数据（平均延迟约15分钟），只需要在网站里提交对应时间的请求即可。

现在用户可以免费注册观天者数据站，在首页进入「中国机场例行天气报告」数据集进行查询下载，最多支持国内40多个大型国际机场，免费用户每天可以查询最多 5000 条历史 METAR 报文数据（每日签到可是获得更多额度）。另外，为了方便用户使用，我们提供了详细的文档说明，更多详细信息和使用方法可以参考以下文档：

* [中国机场例行天气报告](https://docs.datastation.skyviewor.com/data-query/china-airports/metar-dataset/)
* [中国机场例行天气报告操作指南](https://docs.datastation.skyviewor.com/data-query/china-airports/how-to-query-china-metar/)
