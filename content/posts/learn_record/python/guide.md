---
title: "Python读书笔记"
description: ""
date: 2025-07-30T10:36:36+08:00
Lastmod: 2025-07-30T10:36:36+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/778c28/1600/900.webp"
tags: ["读书笔记","Python教程"]
series: "读书笔记"
series_order: 1
---


## 读《Python高手之路》笔记

涵盖的主题较为宽泛，主要是给出如何成为一名经验丰富的Python开发者提供的建议。可以算的上是Python开发的实践知识。

### 主题一

涉及主题：系统库开发与使用

### 主题二：开发自己的Python库

1. 良好的项目工程代码
2. API优化修改

```python
import warnings
class Car(object):
    def __init__(self,name):
        self.name = name
    def turn_left(self):
        ```
        Older API
        ```
        warnings.warn("the function `turn_left` has been ,Please use `self.turn(direction="left") instead.`")
        return self.turn(direction="left")
    def turn(self,direction:str):
        ```
        New APi
        ```
        pass

```

### 主题三

1. 推荐实践操作
2.
