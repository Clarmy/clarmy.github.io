---
title: 一键解决 matplotlib 中文显示问题
date: 2022-04-10 00:00:00 +08:00
modified: 2022-04-10 00:00:00 +08:00
tags: [python, matplotlib]
description: matplotlib 的中文显示问题是一个老生常谈的话题了，我们在网上也可以查到非常多的解决方案，但是在我看来这些解决方案都过于“手动”不够优雅，而且我身边的业务运行环境基本全部都基于 Docker 容器化技术实现，对于 matplotlib 的中文字体设置也必然需要有自动化配置流程，于是大概半年前我写了 mplfonts 这个包，可以轻松+愉快地一键配置 matplotlib 中文字体环境。
---
matplotlib 的中文显示问题是一个老生常谈的话题了，我们在网上也可以查到非常多的解决方案，但是在我看来这些解决方案都过于“手动”不够优雅，而且我身边的业务运行环境基本全部都基于 Docker 容器化技术实现，对于 matplotlib 的中文字体设置也必然需要有自动化配置流程，于是大概半年前我写了 [mplfonts](https://github.com/Clarmy/mplfonts) 这个包，可以轻松+愉快地一键配置 matplotlib 中文字体环境。

这个包之前一直是我自己在用，并未考虑兼容性问题，以至于我给身边的朋友说这个包以后，他在自己的电脑上用不了。很长一段时间我也没有管这个事情，最近心血来潮想把它捡起来，于是在 GitHub Actions 里增加了多系统多版本的 CI，修补了一些 bug，然后重新更新了版本。目前来看，从我自己测试和 CI 的测试结果来看应该是 OK 的，当然还是需要有更多人在实际使用中的反馈才是更真实的。

## 安装
mplfonts 的安装方法是 pip: `$ pip install -U mplfonts`

## 使用
安装好 mplfonts 之后，需要有一个初始化的配置过程。有两种方法：
1. 在终端执行 `$ mplfonts init` 即可。
2. 嵌入初始化代码，在脚本中加入：
```python
from mplfonts.bin.cli import init
init()
```
以上两种方式的结果是等效的，并且只需要执行一次即可。

完成以上步骤，你就可以享受丝滑的中文字体了。而且默认是 Google 和 Adobe 合作设计的开源字体 Noto，可以随意使用，无需顾虑版权争议。

我们来看一下例子：
```python
import matplotlib.pyplot as plt
from mplfonts import use_font

FONT_NAMES = {
    'Noto Sans Mono CJK SC': 'Noto等宽',
    'Noto Serif CJK SC': 'Noto宋体',
    'Noto Sans CJK SC': 'Noto黑体',
    'Source Han Serif SC': '思源宋体',
    'Source Han Mono SC': '思源等宽',
    'SimHei': '微软雅黑'
}

def test_chinese():
    for font_name, desc in FONT_NAMES.items():
        use_font(font_name)
        fig = plt.figure(figsize=(4, 1))
        
        ax = plt.Axes(fig, [0., 0., 1., 1.]) 
        ax.set_axis_off()  
        fig.add_axes(ax)
        
        plt.text(.1, .6, font_name, fontsize=20)
        plt.text(.1, .2, desc, fontsize=20)

        plt.show()

test_chinese()
```
![sample](/assets/img/how-to-solve-tofu-in-matplotlib/sample.png)

## 漫谈
其实饱受 matplotlib 字体显示之苦的并非只有中文，邻国韩国和日本的网友在使用 matplotlib 的时候也会遇到相同的问题，社区里一般会把这个问题称为“豆腐块(tofu)”，就是说写出来的字是豆腐。

关于豆腐问题，在 GitHub 上有一个台湾的哥们写了一个包：[mpl-tc-fonts](https://github.com/Hsins/mpl-tc-fonts)，这是一种比较优雅的解决方案，他先下载了 Google 的 Noto CJK 开源字体，然后在程序入口自动化地执行命令安装字体，再修改 matplotlibrc 配置文件以使其生效，最终实现繁体中文的正确显示。但是他的这个包不支持简体中文，而且程序里很多硬编码，不利于更多非英文字体的扩展，于是我就参考他的一些实现方式，写了 mplfonts，初期的设计理念是在解决中文字体渲染问题的同时，为其他非拉丁字母语系问题的字体扩展留下余地。

关于 Google 的开源字体 Noto，其名称很有意思，据说是“No tofu”的简写，也就是“不要豆腐块”，是 Google 与 Adobe 联合设计的开源字体，由于免费开源且字体优美，目前在网页设计领域已经得到了非常广泛的使用，关于这款字体的更多信息可以去 Google 搜索。目前我在 mplfonts 包里集成了 Noto CJK 字族，衬线体和无衬线体都包含：

* Noto Sans Mono CJK SC：Noto等宽黑体（无衬线体）
* Noto Serif CJK SC：Noto宋体（衬线体）
* Noto Sans CJK SC：Noto黑体（无衬线体）
* Source Han Serif SC：思源宋体（衬线体）
* Source Han Mono SC：思源等宽宋体（无衬线体）

其中等宽无衬线字体比较适合于与代码并存的文本环境中，具体选择的看个人喜好。

其实我还偷偷加入了出镜率很高的微软雅黑（SimHei），但是**请勿商用以免被方正集团碰瓷**。

其实 mplfonts 还有管理字体的功能，更多功能介绍建议查看 GitHub 上该仓库的README。