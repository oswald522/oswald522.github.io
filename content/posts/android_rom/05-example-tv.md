---
title: "安卓刷机教程-05创维电视刷机实战"
description: ""
date: 2025-12-14T12:51:56+08:00
lastmod: 2025-12-14T12:51:56+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/bc701f/1600/900.webp"
tags: ["安卓刷机","系统修改","基础教程"]
series: ["安卓ROM系列"]
series_order: 5
---

## 前言

本文将系统介绍电视刷机包的完整刷机流程，以家中创维电视为例，演示如何对电视系统进行精简和升级。通过DIY固件，可以实现以下功能：

- 🚀 **优化系统性能**：去除冗余功能，提升系统响应速度
- 🔓 **解锁隐藏功能**：支持更多视频格式，优化画质参数
- 🎨 **深度定制体验**：自定义开机动画、预装应用
- 💪 **刷入第三方ROM**：彻底改变使用体验

虽然本文以特定的创维型号为例，但只要是基于 **Mstar（晨星）芯片** 的电视，其底层原理基本通用。如果你也厌倦了系统的种种限制，不妨跟着教程一起探索！😎

> [!warning]
> 风险提示：刷机涉及修改系统底层文件，存在导致设备无法开机（变砖）的可能风险。请在操作前确保已备份重要数据，并具备一定的动手能力。本教程仅供学习交流使用，刷机可能导致保修失效，操作不当可能造成设备变砖，作者不对任何损失负责。

## 实战流程

### 零、系统配置说明及前期准备

#### 设备信息

- **品牌型号**：创维（SkyWorth）7S77/43G31XX-S010339
- **生产日期**：2021年04月19日
- **主控芯片**：Mstar
- **系统架构**：ARMv7（32位），出厂预制安卓8.0系统。

#### 提前检查

开启电视ADB调试后使用开心助手进行连接，`adb shell`的不具有root用户权限，因此只能采取从刷机包提取boot.img的方法，如果具有root权限，可以直接提取所有镜像文件，不需要解包打包的过程。

#### 预先准备:刷机包和U盘

**1. 刷机包获取**:建议在淘宝搜索"[品牌型号]+刷机包"，向客服提供电视铭牌照片（型号、SN等信息），价格通常5-20元。*目标文件*：本例获取的刷机包名为 `MstarUpgrade.bin`，大小约 1.66GB。

**2. U盘准备**：容量：建议8GB以下（部分老设备不识别大容量U盘）。格式：**FAT32格式**，刷机前务必格式化，避免文件碎片导致刷写失败。

### 一、刷机包解包

**安装Python 3.x环境**,建议使用[uv](https://github.com/astral-sh/uv)或者[Anaconda](https://www.anaconda.com/)。

**下载解包工具**[mstar-bin-tool](https://github.com/sha-man-4pda/mstar-bin-tool),并解压到本地目录（如`E:\mstar-bin-tool-master`），具体用法可见`doc`目录。

解包流程为：在`mstar-bin-tool`当前目录下运行`uv run unpack.py`或者`python unpack.py MstarUpgrade.bin`(需配置后python环境变量)，如果运行解包命令没有提供解压路径参数`[extarct_path]`，默认解压路径为当前目录下的`unpacked`文件夹。

```shell
PS E:\mstar-bin-tool-master> uv run .\unpack.py E:\MstarUpgrade.bin
mstar-bin-tool unpack.py v.1.2_sha-man
[i] Analizing header ...
[i] Saving header script to unpacked\~header_script.sh ...
[i] Parsing script ...
[i] Partition: sboot    Offset: 0x4000  Size 0x24000 (144.0 KB) -> unpacked\sboot.img
[i] Partition: MBOOT    Offset: 0x28000 Size 0x263200 (2.39 MB) -> unpacked\MBOOT.img
[i] Partition: recovery Offset: 0x28c000        Size 0x13cc800 (19.8 MB) -> unpacked\recovery.img
[i] Parsing setenv recoverycmd -> mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 recovery 0x02000000\; bootm 0x25000000
[i] Partition: boot     Offset: 0x1659000       Size 0x10c4800 (16.77 MB) -> unpacked\boot.img
[i] Parsing setenv bootcmd -> mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 boot 0x01400000\; bootm 0x25000000
[i] Partition: optee    Offset: 0x271e000       Size 0x3d0ac0 (3.82 MB) -> unpacked\optee.bin
[i] Partition: armfw    Offset: 0x2aef000       Size 0x1c900 (114.25 KB) -> unpacked\armfw.bin
[i] Partition: RTPM     Offset: 0x2b0c000       Size 0x10000 (64.0 KB) -> unpacked\RTPM.img
[i] Partition: dtb      Offset: 0x2b1c000       Size 0x39e8 (14.48 KB) -> unpacked\dtb.img
[i] Partition: frc      Offset: 0x2b20000       Size 0x81010 (516.02 KB) -> unpacked\frc.img
[i] Partition: cm4      Offset: 0x2ba2000       Size 0x20d18 (131.27 KB) -> unpacked\cm4.img
[i] Partition: system   Offset: 0x2bc3000       Size 0x95c7d18 (149.78 MB) -> unpacked\system_sparse.0
[i] Partition: system   Offset: 0xc18b000       Size 0x8e88668 (142.53 MB) -> unpacked\system_sparse.1
[i] Partition: system   Offset: 0x15014000      Size 0x95af0a8 (149.68 MB) -> unpacked\system_sparse.2
[i] Partition: system   Offset: 0x1e5c4000      Size 0x95ffe58 (150.0 MB) -> unpacked\system_sparse.3
[i] Partition: system   Offset: 0x27bc4000      Size 0x1de400 (1.87 MB) -> unpacked\system_sparse.4
[i] Partition: cache    Offset: 0x27da3000      Size 0x1050190 (16.31 MB) -> unpacked\cache_sparse.0
[i] Partition: vendor   Offset: 0x28df4000      Size 0x95ec88c (149.92 MB) -> unpacked\vendor_sparse.0
[i] Partition: vendor   Offset: 0x323e1000      Size 0x8fd24a8 (143.82 MB) -> unpacked\vendor_sparse.1
[i] Partition: vendor   Offset: 0x3b3b4000      Size 0x7b803f8 (123.5 MB) -> unpacked\vendor_sparse.2
[i] Partition: tvservice        Offset: 0x42f35000      Size 0xa000000 (160.0 MB) -> unpacked\tvservice.img
[i] Partition: tvconfig Offset: 0x4cf35000      Size 0xb00000 (11.0 MB) -> unpacked\tvconfig.img
[i] Partition: tvdatabase       Offset: 0x4da35000      Size 0x800000 (8.0 MB) -> unpacked\tvdatabase.img
[i] Partition: tvcustomer       Offset: 0x4e235000      Size 0x1000000 (16.0 MB) -> unpacked\tvcustomer.img
[i] Partition: tvcertificate    Offset: 0x4f235000      Size 0x800000 (8.0 MB) -> unpacked\tvcertificate.img
[i] Partition: atv      Offset: 0x4fa35000      Size 0x1800000 (24.0 MB) -> unpacked\atv.img
[i] Partition: factory  Offset: 0x51235000      Size 0x6c00000 (108.0 MB) -> unpacked\factory.img
[i] Partition: skyworth Offset: 0x57e35000      Size 0x9600000 (150.0 MB) -> unpacked\skyworth.img
[i] Partition: skyworth Offset: 0x61435000      Size 0x3a00000 (58.0 MB) append to unpacked\skyworth.img
[i] Partition: userdata Offset: 0x64e35000      Size 0x56ec568 (86.92 MB) -> unpacked\userdata_sparse.0
[i] Parsing setenv bootargs -> console
[i] Parsing setenv bootargs -> console=ttyS0,115200 androidboot.console=ttyS0 root=/dev/ram rw rootwait init=/init CORE_DUMP_PATH=/data/core_dump.%e_%p.gz KDebug=1 delaylogo=true androidboot.selinux=permissive security=selinux platform=mi tee_mode=optee str_ignore_wakelock pm_path=/tvconfig/config/PM.bin loglevel=0
[i] Parsing setenv bootlogo_gopidx -> 2
[i] Parsing setenv bootlogo_buffer -> E_MMAP_ID_BOOTLOGO_BUFFER
[i] Parsing setenv OSD_BufferAddr -> E_MMAP_ID_JPD_WRITE_ADR
[i] Parsing setenv upgrade_mode -> null
[i] Parsing setenv factory_poweron_mode -> memory
[i] Parsing setenv Customer_Rest -> true
[i] Parsing setenv Customer_Rest_AIStandby -> null
[i] Parsing setenv str_crc -> 2
[i] Parsing setenv db_table -> 0
[i] Parsing setenv verify -> n
[i] Parsing setenv sync_mmap -> 1
[i] Parsing setenv CONFIG_PATH -> /tvconfig/config
[i] Parsing setenv mboot_default_env -> 0
[i] Parsing setenv music -> 1
[i] Parsing setenv music_vol -> 35
[i] Parsing setenv BootVideoPlayBySrv -> 1
[i] Parsing setenv BootVideoPlaySourceConfig -> file:///system/bootvideo.mp4?looptime=1&playtime=10
[i] Parsing setenv close_log -> yes
[i] Parsing setenv MstarUpgrade_complete -> 1
[i] Parsing setenv sync_mmap -> 1
[i] Parsing setenv db_table -> 0
[i] Sparse: converting system_sparse.*to system.img
[i] Sparse: remain cache_sparse.* to cache_sparse.*
[i] Sparse: converting vendor_sparse.* to vendor.img
[i] Sparse: remain userdata_sparse.*to userdata_sparse.*
[i] Done.
```

### 二、刷机镜像解析

MstarUpgrade.bin通常被成为固件（Firmware），即电视操作系统的软件，包含了所有控制电视硬件的基本代码，以及电视中运行的各种功能和服务。驱动程序：为电视的各种硬件组件编写的软件，如显示驱动、声音驱动等，确保硬件能正常工作。配置文件：可能包括用于设置电视特定功能的配置文件，如语言、地区设置、网络配置等。解包后会得到多个`.img` 或 `.bin` 文件，以下是关键文件的说明：

```ini
PS E:\mstar-bin-tool-master\unpacked> ls
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        2025/12/28     10:43         116992 armfw.bin   
-a----        2025/12/28     10:44       25165824 atv.img
-a----        2025/12/28     10:43       17582080 boot.img   #重要文件，Magisk Patch对象
-a----        2025/12/28     10:43       17105296 cache_sparse.0
-a----        2025/12/28     10:43         134424 cm4.img
-a----        2025/12/28     10:43          14824 dtb.img
-a----        2025/12/28     10:44      113246208 factory.img
-a----        2025/12/28     10:43         528400 frc.img
-a----        2025/12/28     10:43        2503168 MBOOT.img  #重要文件，负责电视启动操作
-a----        2025/12/28     10:43        4000448 optee.bin
-a----        2025/12/28     10:43       20760576 recovery.img  #重要文件
-a----        2025/12/28     10:43          65536 RTPM.img
-a----        2025/12/28     10:43         147456 sboot.img
-a----        2025/12/28     10:44      218103808 skyworth.img #厂商自定义文件
-a----        2025/12/28     10:44     1006632960 system.img    #安卓主系统，调整变更主要内容
-a----        2025/12/28     10:44        8388608 tvcertificate.img
-a----        2025/12/28     10:44       11534336 tvconfig.img  #厂商自定义文件
-a----        2025/12/28     10:44       16777216 tvcustomer.img  #厂商自定义文件
-a----        2025/12/28     10:44        8388608 tvdatabase.img  #厂商自定义文件，数据库
-a----        2025/12/28     10:44      167772160 tvservice.img
-a----        2025/12/28     10:44       91145576 userdata_sparse.0
-a----        2025/12/28     10:44      704643072 vendor.img   #厂商专有代码文件，数据库
-a----        2025/12/28     10:43          16384 ~header   # 刷机包文件头，记录刷机包的刷写代码，容量为16K
-a----        2025/12/28     10:43           6349 ~header_script.sh  # 刷机包文件头的提取内容，记录刷机包的刷写代码
```

| 文件名 | 大小 | 功能说明 | 修改难度 |
|:--------:|:------:|:----------:|:----------:|
| **~header** | 16KB | 刷机包文件头，包含分区表和刷写脚本 | ⚠️ 谨慎修改，自动生成 |
| **~header_script.sh** | 6KB | 文件头提取内容（可读） | - |
| **sboot.img** | 147KB | 安全启动镜像 | 🔒 不建议修改 |
| **MBOOT.img** | 2.5MB | 主引导程序（类似电脑BIOS） | ⚠️ 重要文件 |
| **boot.img** ⭐ | 17MB | Android内核及根文件系统 | ✅ Magisk修改对象 |
| **recovery.img** | 20MB | 恢复模式镜像 | ⚠️ 重要文件 |
| **system.img** ⭐ | 1006MB | Android主系统（应用、框架层） | ✅ 主要修改对象 |
| **vendor.img** | 704MB | 厂商专有代码和驱动 | ⚠️ 谨慎修改 |
| **tvservice.img** | 167MB | 电视专用服务 | - |
| **tvconfig.img** | 11MB | 厂商配置文件 | ✅ 可精简 |
| **skyworth.img** | 218MB | 创维专属分区 | ✅ 可精简 |
| **userdata_sparse.0** | 91MB | 用户数据（稀疏镜像） | - |

> **Tips**：带⭐标记的是主要修改对象，其他文件建议保持原样。

### 三、刷机镜像修改

经搜索归纳可知，解包通常有以下两种方法:**ROM助手**和**Linux挂载**。两者均不推荐，ROM助手没有找到特别合适的软件，修改方式固定单一；Linux挂载小白还需要一定Linux使用基础，且实战尝试无法挂载安装稀疏镜像，提示超级块损坏。

推荐使用的方法是[MIK镜像工具](https://github.com/CryptoNickSoft/MIK)，**优势**为Windows下图形化操作，支持拖拽解压/封装，无需Linux环境。

#### system镜像

修改system镜像可以实现去除广告、预制安装应用、调整设置等功能，更多涉及安卓系统的优化和调试。
建议直接使用上面的软件进行解压和封装镜像，此处仍然提供网络其他解压方法进行参考。

##### 镜像挂载修改流程

1. 建立目录镜像挂载
    挂载的目录可以随意进行选取，通常`/mnt/system`

    ```shell
    test@ubuntu:/mnt$ mkdir -p system
    test@ubuntu:/mnt$ sudo mount -rw -t ext4 system.img system/
    ```

2. 查看并修改system.img内容

    ```shell
    test@ubuntu:/mnt/system/$  ls -l # ll 如果存在alias命令
    total 60
    drwxr-xr-x 13 root  root  4096 Jan  1  1970 ./
    drwxrwxr-x  4 biren biren 4096 Jun  9 20:41 ../
    drwxr-xr-x  2 root  root  4096 Dec 16  2012 app/
    drwxr-xr-x  2 root   2000 4096 Dec 16  2012 bin/
    -rw-r--r--  1 root  root  1979 Dec 16  2012 build.prop
    drwxr-xr-x  9 root  root  4096 Dec 16  2012 etc/
    drwxr-xr-x  2 root  root  4096 Dec 16  2012 fonts/
    drwxr-xr-x  2 root  root  4096 Dec 16  2012 framework/
    drwxr-xr-x  8 root  root  8192 Dec 16  2012 lib/
    drwxr-xr-x  3 root  root  4096 Dec 16  2012 media/
    drwxr-xr-x  3 root  root  4096 Dec 16  2012 tts/
    drwxr-xr-x  8 root  root  4096 Dec 16  2012 usr/
    drwxr-xr-x  3 root   2000 4096 Dec 16  2012 vendor/
    drwxr-xr-x  2 root   2000 4096 Dec 16  2012 xbin/
   ```

3. 退出挂载的命令为`test@ubuntu:/mnt$ sudo umount system`，注意当前命令的执行目录。

##### 内置APK删除修改

apk通常放置在`/app`或`/priv-app`目录：等目录当中，直接替换或者添加其他需要的应用即可。预装的应用可能在`preload`文件夹，所有APKS可以使用**Apk-Info工具**查看apks的具体信息。

##### Host文件实现去广告

编辑`/etc/hosts`文件，添加广告域名拦截规则：

```bash
# 创维广告域名示例
127.0.0.1 ads.skyworth.com
127.0.0.1 upgrade.skyworth.com
```

#### boot镜像（主要用于获取Root权限）

主要操作为使用magisk app进行patch，网络上面参考资料已经足够多。
此处记录一个小问题，提取到boot.img镜像后直接使用手机上的Magisk进行Ptach刷入后电视无法启动，主要原因可能有两点：一是**CPU系统架构不匹配**，手机CPU架构为armv8,电视CPU架构为armv7，两者就是64位和32位的区别；二是**疑似DM-verification问题**，涉及到系统镜像的检验等等问题暂不明确，与dm-verity、AVB等有关。
上述问题的解决方案是直接在电视上面安装MagiskAPP，然后进行Patch操作重新刷入后即可成功开机。

#### 其他镜像（可选操作）

修改其他相应的镜像可以调整开机动画、首屏、开机logo内容，此处不再进行详述，方法和`system.img`大同小异。

| 镜像文件 | 可修改内容 | 备注 |
|---------|-----------|------|
| **tvconfig.img** | 开机动画、系统音效 | - |
| **recovery.img** | Recovery界面美化 | - |
| **vendor.img** | 驱动优化（高级） | ⚠️ 需专业知识 |

### 四、刷机包封装

这个过程是解包的逆过程，即将所有镜像重新打包成 `MstarUpgrade.bin`,需要正确配置 `header_script.sh` 和 `config.ini`。简单来说刷机包的文件结构布局如下图，通常将涉及的镜像进行二进制合并，在MBOOT模式下对内置存储芯片进行格式化和内容刷写。文件头主要是实现镜像容量尺寸定位和刷写，以及部分启动变量的写入。

#### 理解文件头 (Header)

解包时生成的 `~header_script.sh` 极其重要，它记录了分区的内存布局和刷写命令（如 `mmc create`, `mmc write`）。其中的内容直接询问大模型可以得到更多的信息。**提示：无需手动进行编写，直接复用解包时生成的内容，或在此基础上修改。**就当前获取的`header_script.sh`，内容为:

```bash
#-------------USB Upgrade Bin Info----------------
# Device : sky848_9s60
# Build PATH : /work_ssd3/workspace/Release/build_system/Mstar/7S77_G51/Android
# Build TIME : 2020-11-23 20:32:01

# File Partition: set_partition
mmc slc 0 1
mmc rmgpt
mmc create misc 0x00080000
mmc create param 0x100000
mmc create recovery 0x02000000
mmc create boot 0x01400000
mmc create optee 0x00600000
mmc create armfw 0x00010000
mmc create RTPM 0x00040000
mmc create dtb 0x00100000
mmc create frc 0x00100000
mmc create cm4 0x00080000
mmc create system 0x3C000000
mmc create cache 0x38000000
mmc create vendor 0x2A000000
mmc create tvservice 0x0A000000
mmc create tvconfig 0x01400000
mmc create tvdatabase 0x00800000
mmc create tvcustomer 0x01000000
mmc create tvcertificate 0x00800000
mmc create atv 0x1800000
mmc create factory 0x6C00000
mmc create skyworth 0xD000000
mmc create demura 0x300000
mmc create userdata 0x106800000

# File Partition: mboot
filepartload 0x28A00000 $(UpgradeImage) 0x4000 0x24000
mmc write.boot 1 0x28A00000 0 0x24000
filepartload 0x28A00000 $(UpgradeImage) 0x28000 0x263200
mmc write.p 0x28A00000 MBOOT 0x263200

# File Partition: recovery
filepartload 0x28A00000 $(UpgradeImage) 0x28c000 0x13cc800
mmc erase.p misc
mmc erase.p recovery
mmc write.p 0x28A00000 recovery 0x13CC800 1
setenv recoverycmd mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 recovery 0x02000000\; bootm 0x25000000
saveenv

# File Partition: boot
filepartload 0x28A00000 $(UpgradeImage) 0x1659000 0x10c4800
mmc erase.p boot
mmc write.p 0x28A00000 boot 0x10C4800 1
setenv bootcmd mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 boot 0x01400000\; bootm 0x25000000
saveenv

# File Partition: optee
filepartload 0x28A00000 $(UpgradeImage) 0x271e000 0x3d0ac0
multi2optee 0x28A00000 optee $(filesize)

# File Partition: armfw
filepartload 0x28A00000 $(UpgradeImage) 0x2aef000 0x1c900
multi2optee 0x28A00000 armfw $(filesize)

# File Partition: RT_PM
filepartload 0x28A00000 $(UpgradeImage) 0x2b0c000 0x10000
mmc erase.p RTPM
mmc write.p 0x28A00000 RTPM 0x10000 1

# File Partition: dtb
filepartload 0x28A00000 $(UpgradeImage) 0x2b1c000 0x39e8
mmc erase.p dtb
mmc write.p 0x28A00000 dtb 0x39E8 1

# File Partition: frc
filepartload 0x28A00000 $(UpgradeImage) 0x2b20000 0x81010
mmc erase.p frc
mmc write.p 0x28A00000 frc 0x81010

# File Partition: cm4
filepartload 0x28A00000 $(UpgradeImage) 0x2ba2000 0x20d18
mmc erase.p cm4
mmc write.p 0x28A00000 cm4 0x20D18

# File Partition: system
mmc erase.p system
filepartload 0x28A00000 $(UpgradeImage) 0x2bc3000 0x95c7d18
sparse_write mmc 0x28A00000 system $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0xc18b000 0x8e88668
sparse_write mmc 0x28A00000 system $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0x15014000 0x95af0a8
sparse_write mmc 0x28A00000 system $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0x1e5c4000 0x95ffe58
sparse_write mmc 0x28A00000 system $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0x27bc4000 0x1de400
sparse_write mmc 0x28A00000 system $(filesize)

# File Partition: cache
filepartload 0x28A00000 $(UpgradeImage) 0x27da3000 0x1050190
mmc erase.p cache
sparse_write mmc 0x28A00000 cache $(filesize)

# File Partition: vendor
mmc erase.p vendor
filepartload 0x28A00000 $(UpgradeImage) 0x28df4000 0x95ec88c
sparse_write mmc 0x28A00000 vendor $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0x323e1000 0x8fd24a8
sparse_write mmc 0x28A00000 vendor $(filesize)
filepartload 0x28A00000 $(UpgradeImage) 0x3b3b4000 0x7b803f8
sparse_write mmc 0x28A00000 vendor $(filesize)

# File Partition: tvservice
filepartload 0x28A00000 $(UpgradeImage) 0x42f35000 0xa000000
mmc erase.p tvservice
mmc write.p 0x28A00000 tvservice 0xA000000 1

# File Partition: tvconfig
filepartload 0x28A00000 $(UpgradeImage) 0x4cf35000 0xb00000
mmc erase.p tvconfig
mmc write.p 0x28A00000 tvconfig 0xB00000 1

# File Partition: tvdatabase
filepartload 0x28A00000 $(UpgradeImage) 0x4da35000 0x800000
mmc erase.p tvdatabase
mmc write.p 0x28A00000 tvdatabase 0x800000 1

# File Partition: tvcustomer
filepartload 0x28A00000 $(UpgradeImage) 0x4e235000 0x1000000
mmc erase.p tvcustomer
mmc write.p 0x28A00000 tvcustomer 0x1000000 1

# File Partition: tvcertificate
filepartload 0x28A00000 $(UpgradeImage) 0x4f235000 0x800000
mmc erase.p tvcertificate
mmc write.p 0x28A00000 tvcertificate 0x800000 1

# File Partition: atv
filepartload 0x28A00000 $(UpgradeImage) 0x4fa35000 0x1800000
mmc erase.p atv
mmc write.p 0x28A00000 atv 0x1800000 1

# File Partition: factory
filepartload 0x28A00000 $(UpgradeImage) 0x51235000 0x6c00000
mmc erase.p factory
mmc write.p 0x28A00000 factory 0x6C00000 1

# File Partition: skyworth
mmc erase.p skyworth
filepartload 0x28A00000 $(UpgradeImage) 0x57e35000 0x9600000
mmc write.p 0x28A00000 skyworth 0x9600000 1
filepartload 0x28A00000 $(UpgradeImage) 0x61435000 0x3a00000
mmc write.p.continue 0x28A00000 skyworth 0x4B000 0x3A00000 1

# File Partition: demura
mmc erase.p demura

# File Partition: userdata
filepartload 0x28A00000 $(UpgradeImage) 0x64e35000 0x56ec568
mmc erase.p userdata
sparse_write mmc 0x28A00000 userdata $(filesize)

# File Partition: set_config
setenv bootargs console
saveenv
setenv bootargs console=ttyS0,115200 androidboot.console=ttyS0 root=/dev/ram rw rootwait init=/init CORE_DUMP_PATH=/data/core_dump.%e_%p.gz KDebug=1 delaylogo=true androidboot.selinux=permissive security=selinux platform=mi tee_mode=optee str_ignore_wakelock pm_path=/tvconfig/config/PM.bin loglevel=0
setenv bootlogo_gopidx 2
setenv bootlogo_buffer E_MMAP_ID_BOOTLOGO_BUFFER
setenv OSD_BufferAddr E_MMAP_ID_JPD_WRITE_ADR
setenv upgrade_mode null
setenv factory_poweron_mode memory
setenv Customer_Rest true
setenv Customer_Rest_AIStandby null
setenv str_crc 2
setenv db_table 0
setenv verify n
setenv sync_mmap 1
setenv CONFIG_PATH /tvconfig/config
setenv mboot_default_env 0
setenv music 1
setenv music_vol 35
setenv BootVideoPlayBySrv 1
setenv BootVideoPlaySourceConfig file:///system/bootvideo.mp4?looptime=1&playtime=10
setenv close_log yes
saveenv
setenv MstarUpgrade_complete 1
setenv sync_mmap 1
setenv db_table 0
saveenv
printenv
```

#### 编写config.ini文件并调试

`mstar-bin-tool`源代码目录`configs`当中已经包含较多的示例代码，其过程也比较简单，该部分需要不间断进行调试或者修改，该部分内容可以直接联系我或者期待下一期分享。**扩展内容：只刷写`boot.img`、`recovery.img`、`system.img`单一或者组合镜像也是可以的。**

在此提供测试的config文件

```ini
# 
# Pack Configuration File for sky848_9s60 (MStar 7S77_G51 Build)
# Build Time: 2020-11-23 20:32:01
# 

[Main]

# Output file name derived from general usage, adjust if needed
FirmwareFileName=MstarUpgrade.bin
ProjectFolder=./unpacked
useHexValuesPrefix=true
SCRIPT_FIRMWARE_FILE_NAME=$$(UpgradeImage)
DRAM_BUF_ADDR=28A00000
MAGIC_FOOTER=12345678
HEADER_SIZE=16KB

[HeaderScript]
Label: \#-------------USB Upgrade Bin Info----------------
 \# Device : sky848_9s60
 \# Build PATH : /work_ssd3/workspace/Release/build_system/Mstar/7S77_G51/Android
 \# Build TIME : 2020-11-23 20:32:01

Prefix:
 mmc slc 0 1
 mmc rmgpt
 mmc create misc 0x00080000
 mmc create param 0x100000
 mmc create recovery 0x02000000
 mmc create boot 0x01400000
 mmc create optee 0x00600000
 mmc create armfw 0x00010000
 mmc create RTPM 0x00040000
 mmc create dtb 0x00100000
 mmc create frc 0x00100000
 mmc create cm4 0x00080000
 mmc create system 0x3C000000
 mmc create cache 0x38000000
 mmc create vendor 0x2A000000
 mmc create tvservice 0x0A000000
 mmc create tvconfig 0x01400000
 mmc create tvdatabase 0x00800000
 mmc create tvcustomer 0x01000000
 mmc create tvcertificate 0x00800000
 mmc create atv 0x1800000
 mmc create factory 0x6C00000
 mmc create skyworth 0xD000000
 mmc create demura 0x300000
 mmc create userdata 0x106800000
    
# Custom directives at the end of the script (Derived from "File Partition: set_config" and partition commands)
Suffix:
 setenv bootargs console
 saveenv
 setenv bootargs console=ttyS0,115200 androidboot.console=ttyS0 root=/dev/ram rw rootwait init=/init CORE_DUMP_PATH=/data/core_dump.%e_%p.gz KDebug=1 delaylogo=true androidboot.selinux=permissive security=selinux platform=mi tee_mode=optee str_ignore_wakelock pm_path=/tvconfig/config/PM.bin loglevel=0
    setenv bootlogo_gopidx 2
    setenv bootlogo_buffer E_MMAP_ID_BOOTLOGO_BUFFER
    setenv OSD_BufferAddr E_MMAP_ID_JPD_WRITE_ADR
    setenv upgrade_mode null
    setenv factory_poweron_mode memory
    setenv Customer_Rest true
    setenv Customer_Rest_AIStandby null
    setenv str_crc 2
    setenv db_table 0
    setenv verify n
    setenv sync_mmap 1
    setenv CONFIG_PATH /tvconfig/config
    setenv mboot_default_env 0
    setenv music 1
    setenv music_vol 35
    setenv BootVideoPlayBySrv 1
    setenv BootVideoPlaySourceConfig file:///system/bootvideo.mp4?looptime=1&playtime=10
    setenv close_log yes
 # avbab set_device_state 0
 setenv devicestate unlock
 saveenv
 # avbab disable-verity
 setenv MstarUpgrade_complete 1
    setenv sync_mmap 1
    setenv db_table 0
    saveenv
    printenv

# List of partitions to pack
# Sizes are derived from "mmc create" or explicit sizes in the source script.
# List of partitions to pack
# [partition_name] - Name of partition. Shold begin with "part/"
# create - flag to generate "mmc create" directive. It requires "size" parameter
# size - Required parameter, if create flag sets to True. Partition size to create [hex]
# erase - flag to generate "mmc erase.p" directive.
# imageFile - Path to image file to pack
# type - partition type:
#  partitionImage - Plain partition image. It generates "filepartload" and "mmc write.p" directives
#   secureInfo - signature file. Uses "store_secure_info" directive
#   nuttxConfig - Nuttx config file. Uses "store_nuttx_config" directive
# lzo - pack partition/chunk to lzo. Uses "mmc unlzo" directive
# chunkSize - chunk size to split partition. A single chunk uses, if chunkSize is not set. Units: B, KB, MB, GB

[part/sboot]
imageFile=${Main:ProjectFolder}/sboot.img
type=sboot
emptySkip=True

[part/MBOOT]
imageFile=${Main:ProjectFolder}/MBOOT.img
type=partitionImage
emptySkip=False

# File Partition: misc
[part/misc]
erase=True

# File Partition: param
[part/param]
erase=True
imageFile=${Main:ProjectFolder}/param.img

# File Partition: recovery
[part/recovery]
erase=True
imageFile=${Main:ProjectFolder}/recovery.img
type=partitionImage
command=setenv recoverycmd mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 recovery 0x02000000\; bootm 0x25000000
 saveenv

# File Partition: boot
[part/boot]
erase=True
imageFile=${Main:ProjectFolder}/boot.img
command=setenv bootcmd mmc read.p 0x23000000 dtb 0x00100000\; mmc read.p 0x25000000 boot 0x01400000\; bootm 0x25000000
 saveenv
type=partitionImage


[part/optee]
imageFile=${Main:ProjectFolder}/optee.bin
type=multi2optee
emptySkip=False

[part/armfw]
imageFile=${Main:ProjectFolder}/armfw.bin
type=multi2optee
emptySkip=False

# File Partition: RT_PM
[part/RTPM]
erase=True
imageFile=${Main:ProjectFolder}/RTPM.img
type=partitionImage

[part/dtb]
erase=True
imageFile=${Main:ProjectFolder}/dtb.img
type=partitionImage
emptySkip=True

[part/frc]
erase=True
imageFile=${Main:ProjectFolder}/frc.img
type=partitionImage
emptySkip=False

[part/cm4]
erase=True
imageFile=${Main:ProjectFolder}/cm4.img
type=partitionImage
emptySkip=False

# File Partition: system
[part/system]
erase=True
imageFile=${Main:ProjectFolder}/system.img
type=partitionImage
sparse=True
chunkSize=150MB

# File Partition: cache
[part/cache]
erase=True
imageFile=${Main:ProjectFolder}/cache.img
type=partitionImage
sparse=True

# File Partition: vendor
[part/vendor]
erase=True
imageFile=${Main:ProjectFolder}/vendor.img
type=partitionImage
sparse=True
chunkSize=200MB

# File Partition: tvservice
[part/tvservice]
erase=True
imageFile=${Main:ProjectFolder}/tvservice.img
type=partitionImage

# File Partition: tvconfig
[part/tvconfig]
erase=True
imageFile=${Main:ProjectFolder}/tvconfig.img
type=partitionImage

# File Partition: tvdatabase
[part/tvdatabase]
erase=True
imageFile=${Main:ProjectFolder}/tvdatabase.img
type=partitionImage

# File Partition: tvcustomer
[part/tvcustomer]
erase=True
imageFile=${Main:ProjectFolder}/tvcustomer.img
type=partitionImage

# File Partition: tvcertificate
[part/tvcertificate]
erase=True
imageFile=${Main:ProjectFolder}/tvcertificate.img
type=partitionImage

# File Partition: atv
[part/atv]
erase=True
imageFile=${Main:ProjectFolder}/atv.img
type=partitionImage

[part/factory]
erase=True
imageFile=${Main:ProjectFolder}/factory.img
type=partitionImage

# File Partition: skyworth
[part/skyworth]
erase=True
imageFile=${Main:ProjectFolder}/skyworth.img
type=partitionImage

# File Partition: demura
[part/demura]
erase=True

# File Partition: userdata
[part/userdata]
erase=True
imageFile=${Main:ProjectFolder}/userdata.img
type=partitionImage
sparse=True
```

#### 打包刷机镜像

在`mstart-bin-tool`目录下启动终端，运行命令为`uv run .\pack.py .\configs\XXXXX.ini`，生成的 `MstarUpgrade.bin` 即为最终的刷机包。

### 五、U盘烧录与刷机

以下内容仅限于前述特定电视。

1. **拷贝文件**：将新生成的 `MstarUpgrade.bin` 复制到 FAT32 格式 U 盘的**根目录**。*注意*：部分海信或老款设备可能要求放入 `/Target/His$xxx$Upgrade.bin`其中$xxx$为特定型号，请根据具体机型确认。
2. **进入强刷模式**：
    - 断开电视电源。
    - 插入 U 盘到 USB 2.0 接口。
    - **按住电视机身（非遥控器）的“待机/确定（开关机）”键不松手**。
    - 接通电源，直到屏幕出现蓝色/绿色的升级进度条再松手。
3. **等待重启**：刷机过程约 5-10 分钟，完成后电视会自动重启。**期间严禁断电！断电可能会导致刷写过程失败，直接影响底层芯片存储，导致系统无法开机。**

```ini
界面提示示例：
┌─────────────────────────┐
│   System Upgrading...   │
│   ████████░░░░ 65%      │
│   Do NOT Power Off!     │
└─────────────────────────┘
```

## 问题集锦

> [!TIP]- 刷机后系统后台更新怎么办？
> 如果电视厂商下发的OTA刷机包包含新的boot镜像，重新刷写升级过程会导致Maigisk Patch后的boot镜像被覆盖，进而导致root权限消失。主要有以下方法解决：
>
> 1. 禁用系统更新，一劳永逸。~~~有一些系统更新就是为了增加广告，越更新越卡~~~
> 1. 找到OTA包提取boot镜像进行按照步骤重新刷入。简易流程为**ADB系统登录**，使用命令`su`获取root权限，运行`du -h`获取分区存储情况，手动更新电视系统，期间不断运行`du -h`观察系统分区占用情况，最后找到不断增多的分区目录，找到`*.zip`之类的OTA升级包。将OTA升级包当作zip或者rar格式的刷机包对待即可。

> [!TIP]- 电视无法识别 U 盘？
>
> 1. 确认 U 盘文件系统为 **FAT32**（不支持 NTFS/ExFAT）。
> 1. 分区表需为 **MBR** 格式（不支持 GPT）。
> 1. 尝试更换 USB 2.0 接口或更小容量（<8GB）的 U 盘。

> [!TIP]- 系统无法正常启动或者刷机错误
>
> 1. 可能是刷机失败或固件不兼容，使用官方固件或联系售后。
> 1. **Boot 校验失败**：Root 后的 boot 镜像未通过校验。尝试刷回原厂 `boot.img` 确认是否恢复。
> 1. **System 权限错误**：Linux 下修改文件后未修正文件权限（通常需 0644 或 0755）。
> 1. **救砖**：准备好官方原版固件，按上述“强刷模式”重新刷入即可修复。

## 参考资料

1. [精简system.img和tvconfig.img让电视迅捷如风](https://post.smzdm.com/p/alpoqmq8/)
1. [康佳LED37R5200PDF电视精简升级](https://www.znds.com/tv-1245812-1-1.html)
1. [Magisk官方文档](https://topjohnwu.github.io/Magisk/)
1. [mstar-bin-tool GitHub](https://github.com/sha-man-4pda/mstar-bin-tool)
