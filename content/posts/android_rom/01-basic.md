---
title: "安卓刷机教程-01基础篇"
description: ""
date: 2025-12-08T10:52:17+08:00
Lastmod: 2025-12-14T10:52:17+08:00
draft: true
showComments: true
featureimage: "https://picsum.photos/seed/2d8df5/1600/900.webp"
tags: ["安卓刷机","系统修改","基础教程"]
series: ["安卓ROM系列"]
series_order: 1
---

介绍安卓系统刷机的基础知识和一些常用的工具

## 安卓系统刷机教程01-基础篇

## 基本概念

Root权限、安卓刷机包镜像，刷机包，刷机流程（解包、打包、刷入），ADB，系统校验

## 使用到工具分类

工欲善其事必先利其器，安卓系统刷机修改需要使用到一些工具软件，大多数软件通常在window操作系统下运行，

### 0. 基础环境配置

1. Python环境配置
   如果涉及到使用python环境，建议使用uv管理器直接安装运行。以下给出建议的代码，代码通常在windows系统的powershell运行，linux系统可自行查询对应的配置代码。

   ```powershell
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   cd XXX #对应的源码文件包
   uv python install
   uv run xxxx.py xxx  # 运行代码
   ```

2. ADB及fastboot工具
   Android SDK Platform-Tools 是 Android SDK 的一个组件。它包含与 Android 平台进行交互的工具，主要是 adb 和 fastboot。虽然 adb 和 fastboot 中的某些新功能仅适用于较新的 Android 版本，但它们是向后兼容的，因此您只需使用最新版本的 SDK 平台工具即可。下载地址为<https://developer.android.google.cn/tools/releases/platform-tools?hl=zh-cn>。

### 1. 固件打包解包工具

| 序号  | 名称                 | 开发语言   | 运行平台             | 适用SOC   | 功能描述                                         |                            来源地址                            |
| --- | ------------------ | ------ | ---------------- | ------- | -------------------------------------------- | :--------------------------------------------------------: |
| 1   | Mstar-bin-tool[^1] | Python | Win或Linux，Python | 晨星Mstar | 刷机固件包解包合成解密等功能                               | [mstar-bin-tool](https://github.com/xhuboy/mstar-bin-tool) |
| 2   | TIK                |        | Win或Linux        | 通用      | 通用类型的解包打包软件                                  |                                                            |
| 3   | payload-dumper-go  | Go     | 全平台支持            | 通用      | An android OTA payload dumper written in Go. |         <https://github.com/ssut/payload-dumper-go>          |

[^1]: 这是一个脚注的内容。

### 2. 系统修改工具

大多数是基础的镜像修改，主要使用的工具软件有HxD Hex Editor（二进制编辑调整）、WinMergeU（二进制或其他格式文件对比）、Beyond Compare（文件对比）等等

### 3. 连接工具

使用列表格式，说明清楚内容和下载地址
主要工具有 谷歌官方的platform-tools adb，开心连接助手，

### 4. 其他可能需要软件工具

1. 系统软件优化类
   - Magisk
   - KernelSu
2. 增强管理工具
   - 小白文件管理器

3. 影视小软件
   - 星火电视海外版，直播观看电视频道，
   - Uvvidio影视
   - 各式类型的TVBox，优点是开源空壳，只需要视频源即可进行观看视频内容。

4. 网络调试类
   - 系统抓包软件，如PorxyPin、Reqable等等。
