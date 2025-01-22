---
title: 下载 ERA5 数据时如何实现断点续传
date: 2025-01-22 16:14:00 +08:00
modified: 2025-01-22 16:14:00 +08:00
tags: [ERA5]
description: "教你如何使用 wget 实现下载 ERA5 数据时的断点续传功能"
comments: true
---
我们在下载 ERA5 数据的时候，大部分人应该都会遵守 cds 网站的指导，在配置完页面以后使用网站自动生成的 Python 示例代码进行下载（或者做一些外围修改，比如改为以日为循环或者改为多线程/进程，而核心的下载仍然使用 cdsapi 的方法）。

但是这种下载方法有一个问题，那就是它做不到断点续传。如果 cds 按照我们请求的表单生成的单个文件比较大的话，那么就很容易出现因为网络不稳定而导致的中断。一旦出现中断，如果直接在代码里增加简单的重试逻辑再次请求下载，那么 cds 就会重新生成一个新的链接给我们下载那么我们就需要从头再来（如果以几百 K 的速度辛辛苦苦下载到 99% 然后连接中断需要重新下载，是不是就很崩溃？）。如果反复出现网络波动，那么程序就会一次又一次地重新提交请求然后重新下载，这样不但浪费时间，也会给 cds 的服务器带来额外的压力。

所以我们需要用一个方法来实现断点续传。即使在下载的过程中链接中断了我们重试时，也能使用原有的链接从中断点继续下载，直到文件下载完成。

要实现断点续传功能，我们可以使用 Python 自己编写函数实现。但我觉得麻烦，其实可以使用现成的成熟工具 wget 来完成下载的断点续传。wget 是跨平台的工具，它既既支持 Linux 也支持 Windows 系统。具体安装细节可以自行查阅相关资料。

我们需要的工具包：
1. wget：不同平台安装方式不一样请自行搜索，注意不是 `pip install wget` 这种方式安装。
2. retrying: `pip install retrying`
3. cdsapi: `pip install cdsapi`

下面这个例子是下载 ERA5 高空某些变量（用于跑盘古）数据的脚本，单次请求下载的文件大约 2.7GB 左右。当然 `.cdsapirc` 文件的配置还是需要的，这里就不再赘述了。

```python
import os
import cdsapi
import zipfile
from multiprocessing import Pool
from subprocess import run
from retrying import retry

dataset = "reanalysis-era5-pressure-levels"
request_template = {
    "product_type": ["reanalysis"],
    "variable": [
        "geopotential",
        "specific_humidity",
        "temperature",
        "u_component_of_wind",
        "v_component_of_wind"
    ],
    "year": ["2024"],
    "month": [
        "01", "02", "03",
        "04", "05", "06",
        "07", "08", "09",
        "10", "11", "12"
    ],
    "day": [
        "01", "02", "03",
        "04", "05", "06",
        "07", "08", "09",
        "10", "11", "12",
        "13", "14", "15",
        "16", "17", "18",
        "19", "20", "21",
        "22", "23", "24",
        "25", "26", "27",
        "28", "29", "30",
        "31"
    ],
    "time": [
        "00:00", "01:00", "02:00",
        "03:00", "04:00", "05:00",
        "06:00", "07:00", "08:00",
        "09:00", "10:00", "11:00",
        "12:00", "13:00", "14:00",
        "15:00", "16:00", "17:00",
        "18:00", "19:00", "20:00",
        "21:00", "22:00", "23:00"
    ],
    "pressure_level": [
        "50", "100", "150",
        "200", "250", "300",
        "400", "500", "600",
        "700", "850", "925",
        "1000"
    ],
    "data_format": "grib",
    "download_format": "zip"
}

def check(zip_file):
    """通过 zip 解压的方式验证下载的文件是完整的"""
    directory = os.path.dirname(zip_file)
    try:
        with zipfile.ZipFile(zip_file, "r") as zip_ref:
            zip_ref.extractall(directory)
    except zipfile.BadZipFile:
        # 如果文件无效则删除
        os.remove(zip_file)
        return False
    else:
        return True

@retry  # 使用 retrying 库的 retry 装饰器，可以在函数执行失败时自动重试
def download_by_wget(url, savefp):
    """使用 wget 下载文件，支持断点续传"""
    cmd = f"wget -O {savefp} -c {url}"  # 这是一个简单的 wget 命令，-O 参数指定保存的文件名，-c 参数表示断点续传
    run(cmd, shell=True, check=True)   # 使用 run 函数执行命令，shell=True 表示使用 shell 执行，check=True 表示如果命令执行失败则抛出异常


def download_data(month, day):
    client = cdsapi.Client()

    try:
        request = request_template.copy()
        request["month"] = [month]
        request["day"] = [f"{day:02d}"]
        savedir = "/data/reanalysis/era5/upper/grib/zqtuv/2024/"
        os.makedirs(savedir, exist_ok=True)
        savefp = os.path.join(savedir, f"era5_{month}_{day:02d}.grib.zip")
        if os.path.exists(savefp):
            if os.path.getsize(savefp) < 2600 * 1024 * 1024:  # 如果文件小于 2.6GB 则删除，因为这个文件大小是正常的文件大小
                os.remove(savefp)
            else:
                print(f"File {savefp} already exists and is of sufficient size, skipping")
                return
        
        # client.retrieve(dataset, request).download(savefp)  # 使用官方示例脚本的下载方法，该方法无法完成断点续传的功能
        response = client.retrieve(dataset, request)
        url = response.location    # response 对象的 location 属性保存的其实是下载的 URL，也不知道为什么要这样取名
        print(f"URL: {url}")  # 打印下载链接
        download_by_wget(url, savefp)

        check_result = check(savefp)
        if not check_result:
            print(f"{savefp} abnormal, removed")

    except Exception as e:
        print(f"Failed to download data for {month}-{day:02d}: {e}")
    else:
        print(f"Downloaded data for {month}-{day:02d}")

if __name__ == "__main__":
    tasks = [(month, day) for month in request_template["month"] for day in range(1, 32)]
    with Pool(processes=4) as pool:    # 使用多进程下载，可以加快下载速度
        pool.starmap(download_data, tasks)
```

这个脚本主要就是在发送数据请求之后，不直接用 cdsapi 提供的下载方法，而是获取到下载链接（cds 给我们生成下载链接在一定时效内是可以反复下载的，cdsapi 的下载本质上也是从这个链接进行下载）后使用 `wget` 命令下载。`wget` 命令的 `-c` 参数即是启动断点续传，它能够自动计算本地已存在文件的数据位长度（断点位置），然后从服务端下载时自动寻址到断点位置并进行后续的下载，该命令具有幂等性，可以反复执行直到文件下载完整为止。

上述代码是在 Linux 环境下运行的，未在 Windows 环境下做过测试，但原理相通。
