---
title: "信创电脑修改Root密码教程"
description: ""
date: 2025-07-30T10:53:37+08:00
lastmod: 2025-07-30T10:53:37+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/b33554/1600/900.webp"
tags: ["Linux教程"]
series: "Linux教程"
series_order: 1
---

## 信创电脑修改Root密码教程

如果在使用银河麒麟操作系统V10时不慎忘记了root密码，可以按照以下步骤重置：

1. 重启系统并进入GRUB菜单：首先，需要重启银河麒麟V10操作系统。在系统启动过程中，屏幕会显示GRUB菜单。请注意时机，在GRUB菜单出现后迅速按下e键。
![进入GRUB菜单](/img/grub_e.png)
2. 输入GRUB账户密码：银河麒麟V10服务器版操作系统在GRUB模式下可能需要输入账户密码才能进一步操作。默认情况下，账户名为`root`，密码为`Kylin123123`。

3. 修改启动参数：在GRUB编辑界面中，使用上下箭头键将光标移至以linux开头的行。在该行末尾添加以下参数：`rw init=/bin/bash console=tty0`。这些参数会将系统引导至单用户模式，允许你无需密码即可执行系统命令。修改完成后，按下Ctrl+X或根据屏幕提示的快捷键启动系统。
![修改启动参数](/img/linux_string.png)
4. 修改root密码
   系统启动进入单用户模式后，你会看到一个命令行界面。此时，输入passwd root命令并按回车键。系统会提示你输入并确认新的root密码。直接输入两遍你想要设置的新密码即可，无需输入原密码。

5. 重启系统：
    密码修改完成后，需要重启系统以使更改生效。请注意，在单用户模式下，直接使用reboot命令可能无效。建议使用完整路径/usr/sbin/reboot来重启系统，并可加上-f参数以强制重启。
![修改启动参数](/img/change_reboot.png)
