+++
title = 'Python 中的浅拷贝和深拷贝'
date = 2022-04-30T17:19:27+08:00
categories = ['Python']
tags = ['python', 'bilibili', '在线工具']
summary = '学习 Python 的时候，关于浅拷贝和深拷贝不太懂，查阅资料时发现一个神奇的网站，对浅拷贝和深拷贝的理解帮助很大，非常直观。'
+++

* 工具地址：[https://pythontutor.com/live.html#mode=edit](https://pythontutor.com/live.html#mode=edit)
* B 站地址：[https://www.bilibili.com/video/BV1jT4y1G7AN](https://www.bilibili.com/video/BV1jT4y1G7AN)

```python
import copy

list1 = [1,2,3,[4,5,[6,7,8]]]

# 浅拷贝
list2 = list1
list3 = list1[:]
list4 = list1.copy()

# 深拷贝
list5 = copy.deepcopy(list1)
```

![202210101727181](https://xwls-oss.netlify.app/images/202210101727181.png)