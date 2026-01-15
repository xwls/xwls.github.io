---
title: Python 从 CSV 下载文件
date: 2022-07-02 15:56:31+08:00
categories:
- Python
tags:
- util
- python
summary: 使用 python 读取 csv 文件，解析文件内的 url 并下载到本地。
---

## CSV 文件

```text
img01,https://img.xxx.com/a/b/c/1.jpg
img02,https://img.xxx.com/a/b/c/2.jpg
img03,https://img.xxx.com/a/b/c/3.jpg
img04,https://img.xxx.com/a/b/c/4.jpg
```

## Python 代码

```python
import csv
import requests

with open("urls.csv") as f:

    f_csv = csv.reader(f)

    for row in f_csv:
        img_name = row[0]
        img_url = row[1]
        file_path = "./imgs/"+img_name+".png"
        r = requests.get(img_url)
        with open(file_path, "wb") as save_file:
            save_file.write(r.content)
        print(img_name+" download complete")
