---
title: "安卓刷机教程-05海信安卓4.4实战"
description: ""
date: 2025-12-14T12:51:56+08:00
lastmod: 2025-12-14T12:51:56+08:00
draft: true
showComments: true
featureimage: "https://picsum.photos/seed/bc701f/1600/900.webp"
tags: ["安卓刷机","系统修改","基础教程"]
series: ["安卓ROM系列"]
series_order: 5
---

以下内容为转载。

在我的上一篇帖子中，我写了精简固件并刷机之后的流畅手感，发现有值友和我境况相同，因为电视广告太多都想怒砸一波电视了。所以这一期，我也不想藏着掖着了，不如把我折腾的全过程写出来让值友们少走弯路。

该教程理论上也支持其他所有Mstar系列芯片的安卓智能电视，比如乐视、小米、LG、TCL、长虹、康佳等。

1.从百度云网盘按实际机型下载官方固件

风行电视原厂固件可以在风行电视微信公众号-技术支持-《风行产品UI系统强制升级注意事项》一文里找到，这里也贴出官方固件下载链接：

    638机芯
    提取码：tciq
    938机芯
    提取码：hymi
    648机芯
    提取码：9ady
    338机芯
    提取码：gpn7

2.从我的Github上下载该项目所有源码并解压到桌面

3.下载一个好用的HEX编辑器

我用的是 HxD Hex Editor（源码压缩包里有）
4.用HxD Hex Editor打开固件文件（BIN）

拉到最下面，记录下最后几位，比如下图中我打开的是938系列芯片的固件，这个数字是

    12346938

HEX编辑器打开BIN固件HEX编辑器打开BIN固件
5.安装python，最好是安装最新版

6.WIN+R打开cmd命令提示符，输入

    CD XX

(XX指代你存放下载的源码的目录)，例如： CD C:UsersLemonDesktopFunTV-Mstar-series-Core-Root
7.解包固件

输入

    unpack.py Mstarupgrade.bin

8.解包完成后会得到unpacked文件夹，打开，会看到system.img
9.使用ROM精灵等软件编辑或精简img格式固件

删除system.img中的

```
AdPlayer.apk、AdPlayer.odex、TVLauncher.apk、
TVLauncher.odex、TVUpgrade.apk、TVUpgrade.odex、
funtvRCOTA.apk
```

其他组件看个人需求增删，这时候也可以加入supersu权限管理了，做好后对img重新打包

（具体教程百度很多，顺带加入ROOT的教程也有，这里我就不啰嗦了，实在不懂的话下一期我再水一篇教程自己动手，解包打包精简风行电视Mstar系列官方固件 ）

用ROM精灵打开tvconfig.img，可以修改开机画面和开机铃声，不多叙述。
10.编辑config.ini

打开FunTV-Mstar-series-Core-RootconfigsD58Y-system-tvconfig.ini这个文件，

 将

     MAGIC_FOOTER=12346648

 这一行的 12346648

改为你用HEX编辑器看到的那几个数字， 保存更改。

这一步非常关键，我搞了半个多月主要就是卡在MAGIC_FOOTER如何计算上了自己动手，解包打包精简风行电视Mstar系列官方固件

把unpacked文件夹改名为pack。
11.打包做好的精简固件

回到CMD命令行窗口，输入

    pack.py C:UsersLemonDesktopFunTV-Mstar-series-Core-RootconfigsD58Y-system-tvconfig.ini 

（ini文件位置自己注意更换,或者直接把pack.py拖进去，空格，再把ini文件拖进去）
12.刷原厂包

先把百度云下载的原厂固件更名为“Mstarupgrade.bin”，放入U盘，电视关机并拔掉电源线。插入u盘并按住电源键，插入电源线，等到蓝色刷机界面松开电源键等待刷机进度条走完。
13.刷精简包前的准备工作

刷完官方固件开机，不要联网，先连接遥控器，所有设置按自己习惯设置好，关机。
14.刷精简包

把自己动手做好的固件包放在U盘里，也重命名为Mstarupgrade.bin。

电视关机并拔掉电源线。插入u盘并按住电源键，插入电源线，等到蓝色刷机界面松开电源键等待刷机进度条走完。
15.Done！

每次冷开机大概持续15秒，开机画面后直接进入替换后的桌面。

## 参考资料

1. [自己动手，解包打包精简风行电视Mstar系列官方固件](https://post.smzdm.com/p/alpoqmdp/)
