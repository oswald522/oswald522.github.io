---
title: "安卓系统刷机教程系列-03镜像修改篇"
description: ""
date: 2025-12-14T11:10:16+08:00
Lastmod: 2025-12-14T11:10:16+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/ca32ab/1600/900.webp"
tags: ["安卓刷机","系统修改","基础教程"]
series: ["安卓ROM系列"]
series_order: 3
---

## 镜像类型及主要作用

## 镜像修改步骤流程

获取到Root权限之后，很容易对文件进行修改。主要有两种方式：基于文件直接修改和基于Magisk Module的替代修改，两者各有优缺点，不再进行详细描述。以下分别对boot、recovery、system等镜像进行修改。通常大多数镜像是不可读，需要在root权限下进行重新挂载，命令行为`mount -o rw,remount $1`,其中`$1`通常为挂载路径，常见内容为`/system,/vendor`等等。

### boot镜像

### recovery镜像

### system镜像

### tvconfig等镜像

## 附加功能描述

### 1. 安卓4.4获取Root权限步骤

主要思想就是文件替换，使用superUser提供的二进制文件放置到相应的位置，然后修改启动脚本比如/system/bin/install-recovery.sh等等，添加一行：`/system/bin/su --auto-daemon &`, 注意`&`符合必须要存在，意味着su的守护进程在后台运行。

### 2. 安卓8.0及以上获取Root权限

主要原理较为复杂，建议直接使用Magisk进行管理和控制，具体安装过程参照官网教程进行。

注意的点有：

1. 注意设备平台架构体系。建议目标设备上面安装Magisk.apk，然后进行boot或者recovery镜像的patch。
2. 如果没有解开设备锁等（通常所说的dm-verify或这其他），提示保留vbmeata分区，然后进行修补。

### 3. 修改开机动画

### 4.开机服务（增加SSH服务）

### 5.修复遥控菜单

通常使用非官方的固件或者软件，发现主页键可能失效，出现无法打开桌面系统的情况。
参考资料比较少，
