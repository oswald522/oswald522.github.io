---
title: VPS环境部署常用实践及操作命令（以Debian为例）
date: 2025-03-15T16:50:41+08:00
lastmod: 2025-06-16T10:00:00+08:00
draft: false
description: 一份从零开始的VPS（Debian）环境配置指南，涵盖系统安装、安全加固、性能优化到Docker部署的全流程，帮助你打造一个稳定、高效、安全的服务器环境。
featureimage: https://picsum.photos/seed/674acfad/1600/900.webp
showComments: true
tags:
  - 经验技术
  - Linux
  - VPS
  - Debian
  - 服务器部署
# slug: "series"
series: ["Linux部署教程"]
series_order: 1
---

## 👋 前言

VPS (Virtual Private Server)，我们可以亲切地称它为一台永不关机的“云端电脑” ☁️。如何从零开始，一步步将它配置成一个稳定、高效且安全的工作环境？这份笔记总结了我过往的实践经验，既是分享，也是个人备忘。

{{< alert icon="circle-info" >}}
本教程所有命令均在 **Debian 11/12** 系统下测试通过，理论上兼容 Ubuntu。建议在全新的、纯净的操作系统上操作。

约定说明：`{}` 及其中的内容代表您需要根据实际情况替换的变量。例如 `ssh {user}@{server_ip}`，若用户名为 `root`，IP为 `198.56.25.1`，则替换为 `ssh root@198.56.25.1`。
{{< /alert >}}

---

## 🗺️ 部署路线图

我们将按照以下步骤，由浅入深地完成服务器配置：

1. **新机开荒**：重装一个纯净的操作系统。
2. **基础配置**：更新系统、安装必备工具、校准时间。
3. **安全加固**：配置 SSH 密钥登录，修改端口，提高安全性。
4. **性能优化**：开启 BBR 加速，添加 SWAP 虚拟内存。
5. **应用环境**：安装 Docker 和 Docker-compose。
6. **最佳实践**：分享一些重要的安全好习惯。

---

## 🚀 第一步：新机开荒 (可选)

对于部分厂商（尤其国内）提供的系统镜像，可能包含一些不必要的监控或预装软件。通过 DD (Disk Dump) 的方式重装一个纯净的官方系统，是保证服务器干净的第一步。

> 💡 **何时需要DD？**
> 厂商不提供你想要的操作系统版本 (如 Debian 12)。
> 想彻底清除厂商的预装程序。

推荐脚本: {{< github repo="bin456789/reinstall" >}} (脚本内容已做基本审查)。

```shell
# 必须使用 root 账户执行
# 1. 更新源并安装基础工具
apt update && apt install -y curl wget

# 2. 下载DD脚本
# 国外VPS
curl -O https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh || wget -O reinstall.sh $_ 
# 国内VPS (使用镜像加速)
curl -O https://cnb.cool/bin456789/reinstall/-/git/raw/main/reinstall.sh || wget -O reinstall.sh $_

# 3. 运行脚本，例如安装 Debian 12
chmod a+x reinstall.sh && ./reinstall.sh -debian 12
```

**备选脚本**：{{< github repo="leitbogioro/Tools" >}}，可根据需要选用。需要注意的是系统重启安装过程可能需要一定的时间，因此执行命令脚本重启后需要等待时间才能使用ssh连接（具体时间根据服务器性能配置而定）。

---

## 🛠️ 第二步：基础系统配置

系统装好后，我们来做一些基础的初始化设置。

### 1. 更新系统与软件

```shell
# 养成备份好习惯，备份默认软件源
cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 全面更新系统
apt update && apt upgrade -y && apt dist-upgrade -y && apt full-upgrade -y && apt autoremove -y

# 安装常用必备工具
apt install -y sudo curl wget nano zsh git axel vim htop bat
```

### 2. 校准时区与主机名（可选）

```shell
# 将时区设置为上海
sudo timedatectl set-timezone Asia/Shanghai
# 修改主机名 (例如: my-awesome-vps)
hostnamectl set-hostname {my-awesome-vps}
echo "127.0.0.1 {my-awesome-vps}" >> /etc/hosts
# 查看当前时区和时间
timedatectl
```

> 💡 **提示：**
>
> * 查看所有可用时区: `timedatectl list-timezones`
> * 选择一个易于辨识的主机名，方便多台服务器管理。

---

## 🔐 第三步：安全加固 - SSH 配置

这是保障服务器安全**最重要**的一步！强烈推荐使用密钥登录，彻底告别弱密码风险。

### 1. 🔑 生成并配置 SSH 密钥

**本地电脑操作**：打开你的终端（不是VPS），生成密钥对。

```shell
# -t 指定算法, ed25519 是目前推荐的算法
# -C 添加注释，通常用邮箱或用户名，方便识别
ssh-keygen -t ed25519 -C "{your_email@example.com}"
```

命令执行后，会在你的本地电脑 `~/.ssh/` 目录下生成 `id_ed25519` (私钥) 和 `id_ed25519.pub` (公钥) 两个文件。

**服务器操作**：将**公钥**内容添加到服务器的信任列表。

```shell
# 1. 登录你的VPS
# 2. 备份现有的 authorized_keys 文件 (如果有的话)
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak

# 3. 将你的公钥内容完整地粘贴进去
# 方法一：使用 echo (推荐)
# 将 cat ~/.ssh/id_ed25519.pub 在本地电脑查看到的公钥内容替换下面的一整行
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMz2IFBt+XSSXX+FX5FlX your_email@example.com" >> ~/.ssh/authorized_keys

# 方法二：使用编辑器
# nano ~/.ssh/authorized_keys
# (将公钥内容粘贴进去，保存并退出)
```

### 2. 🚪 更改默认 SSH 端口

修改 SSH 端口通常有以下几个主要原因：

* 增强安全性：SSH 服务默认使用的 22 端口是攻击者经常扫描和尝试攻击的目标。通过将端口修改为一个不常见的数值，可以减少自动攻击和暴力破解的风险，因为攻击者通常会首先针对常见的默认端口进行攻击。例如，如果攻击者使用自动化工具扫描大量服务器，这些工具可能主要集中在 22 端口。而修改了端口后，就降低了被这类工具轻易发现和攻击的可能性。

* 减少误连接和非法访问尝试：一些网络环境中可能存在大量的随机连接请求或非法访问尝试，针对默认的 22 端口。更改端口可以减少这类无意义的连接请求。假设您的服务器处于一个公共网络环境中，经常会收到大量的随机连接尝试，其中很多是针对常见端口的。修改 SSH 端口可以减少这类不必要的干扰。默认的 22 端口是自动化扫描和攻击的首要目标。修改为一个不常用的高位端口（如 10000-65535 之间）能有效减少恶意尝试。

```shell
# 将默认的 22 端口修改为你选择的端口，例如 55520
sudo sed -i 's/^#\?Port 22.*/Port {55520}/g' /etc/ssh/sshd_config
```

### 3. 🚫 禁用密码和 Root 登录 (强烈建议)

```shell
# 编辑 SSH 配置文件
sudo nano /etc/ssh/sshd_config
```

找到并修改以下几项，确保它们是如下状态：

```ini
PermitRootLogin no          # 禁止 root 用户直接登录
PasswordAuthentication no   # 禁止使用密码登录
PubkeyAuthentication yes    # 确保公钥认证是开启的
```

### 4. ✅ 重启并验证

以下的命令和操作系统相关，有一些操作系统是 ssh 而不是sshd。

```shell
# 重启 SSH 服务使其生效
sudo systemctl restart sshd
```

{{< alert icon="triangle-exclamation" type="warning" >}}
**🔥 关键操作！**
在断开当前连接之前，**必须**打开一个**新的终端窗口**，使用你的**新端口**和**密钥**尝试登录：
`ssh -p {55520} {user}@{server_ip}`
**确认可以成功登录后**，再关闭旧的终端窗口。否则配置错误会导致你再也无法连接服务器！
{{< /alert >}}

---

## ⚡ 第四步：性能优化

这个功能通过调整各种系统和网络参数来优化服务器的性能。

### 0. 系统参数调优

这个功能通过调整各种系统和网络参数来优化服务器的性能。

实现方法：

1. 内核参数调整：例如，增加 TCP 缓冲区大小、修改系统队列长度等，这些改变有助于提高网络吞吐量和减少延迟。
2. 性能优化：安装和配置 Tuned 和其他系统性能优化工具来自动调整和优化服务器的运行状态。
3. 资源限制：例如，设置文件打开数量的限制，这可以防止某些类型的资源耗尽攻击。

通过这些功能，你的服务器不仅能够更有效地管理资源，还能提高对外部威胁的防护能力，保障系统稳定运行。

`bash <(wget -qO- https://raw.githubusercontent.com/jerry048/Tune/main/tune.sh) -t`

### 1. 🚄 BBR 网络加速

BBR 是 Google 提出的一种新型拥塞控制算法（Bottleneck Bandwidth and RTT），全称为瓶颈带宽和往返传播时间。

在 Linux 系统中，BBR 主要有以下特点和作用：

* 提高网络性能：它可以显著提高吞吐量和降低 TCP 连接的延迟，使数据传输更加高效。
* 适应不同网络环境：适合高延迟、高带宽的网络链路，以及慢速接入网络的用户，能在一定丢包率的网络链路上充分利用带宽，并降低网络链路上的缓冲区占用率从而降低延迟。
* 优化拥塞控制：BBR 改变了传统基于丢包反馈的拥塞控制机制，通过精确测量往返传播时间（RTT）和瓶颈带宽等参数来更有效地控制数据发送速率，避免了传统算法中因单纯丢包判断拥塞而导致的带宽利用率不高和端到端延迟大等问题。
* 提升网络稳定性：有助于减少网络拥塞和数据包丢失，提高网络的稳定性和可靠性。

> PT 玩家力荐的 **BBRx** 版本，在原版 BBR 基础上调整了关键参数，实测效果更佳。

```shell
# 一键安装 BBRx 并优化系统内核
bash <(wget -qO- https://raw.githubusercontent.com/jerry048/Tune/main/tune.sh) -x

# 安装完成后，重启 VPS 使新内核生效
sudo reboot
```

**验证 BBR 是否开启成功**：

```shell
# 重启后重新登录，执行命令
lsmod | grep bbr

# 理想情况下，应看到 tcp_bbrx 和 tcp_bbr
# 如果只看到 tcp_bbr，可以稍等几分钟再次重启 sudo reboot 试试
```

**备选 BBR 方案** (如果你不偏好 BBRx):

```shell
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
# 同样需要重启生效
```

### 2. 💾 添加 SWAP 虚拟内存

SWAP 就像是物理内存(RAM)的“备用空间”。当 RAM 不足时，系统会将不常用的数据临时存放到 SWAP（一块硬盘空间）中，避免因内存耗尽而导致程序崩溃。对于内存较小的 VPS 来说非常必要。

```shell
# 使用一键脚本添加 SWAP
wget -O swap.sh https://raw.githubusercontent.com/yuju520/Script/main/swap.sh && chmod +x swap.sh && ./swap.sh

# 查看当前内存和 SWAP 情况
free -m
```

---

## 📦 第五步：应用环境 - 安装 Docker

Docker 是现代应用部署的基石，它能将应用及其依赖打包成一个轻量、可移植的“容器”，实现环境隔离和快速部署。

### 1. 安装 Docker

Docker 从 18.06.0-ce 版本就开始自带 Docker Compose 工具，因此，安装docker会直接安装docker compose插件。以后看到的相关教程出现的命令`docker-compose`调整为`docker compose`，即中间去掉`-`符号。

```shell
# 🌐 非大陆服务器 (官方脚本)
curl -fsSL https://get.docker.com | bash

# 🇨🇳 大陆服务器 (使用国内镜像加速)
curl https://install.1panel.live/docker-install | bash
```

### 2. 配置与验证

```shell
# 设置 Docker 开机自启动
sudo systemctl enable docker

# 查看 Docker 版本
docker -v

# 验证 Docker Compose (新版 Docker 已内置)
docker compose version
```

### 3. (可选) 卸载 Docker

如果需要卸载，可以使用以下命令：

```shell
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo apt-get remove docker docker-engine
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

---

## 🧑‍💻 第六步：安全与最佳实践 (重要!)

养成良好的使用习惯，能让你的服务器免受许多不必要的风险。

* ✅ **使用普通用户**：日常操作避免直接使用 `root`，创建一个普通用户，在需要管理员权限时使用 `sudo`。 (TODO: 待补充创建用户教程)
* 🐳 **隔离未知应用**：对于非自己编写或不熟悉的应用（如 WordPress），一律使用 Docker 部署，最大限度地隔离，减少攻击面。
* 🕵️ **审查第三方脚本**：执行任何来路不明的一键脚本前，务必先用 `cat` 或 `nano` 查看其内容，警惕埋雷。也可以让 AI 帮忙初步审查。
* 🛡️ **慎用管理面板**：尽量避免安装宝塔等闭源或权限过高的服务器管理面板，它们会显著扩大攻击面。
* 🔑 **保护私钥**：为你的 SSH 私钥文件设置一个强密码，即使私钥文件泄露，没有密码也无法使用。

---

## 🎉 结语

恭喜！到这里，你的新 VPS 已经完成了从“毛坯房”到“精装修”的全部过程。它现在是一个相对安全、高效且配置了现代化应用环境的平台。

关于如何使用 Docker 部署各类常用应用，篇幅所限，我们下期再见！

---

## 📚 附录：参考资料

* [Linux 部署及安全实践指南](https://linux.do/t/topic/468841)
* [【配置优化】我拿到VPS服务器必做的那些事](https://linux.do/t/topic/160305)

    原文链接：新到手的 Linux 服务器，我这样设置 | Dejavu's Blog
    昨天下午把几台服务器的自托管服务都迁移了一遍，花了几个小时，顺便把 Linux 服务器开荒教程也更新了一下。

概要

本文记录我初始化 Linux 服务器的 标准作业程序（SOP），这并非通用指南，仅作为个人快速部署环境的速查手册。
重装操作系统

重装有风险，本文不为任何脚本的安全性背书，请自行衡量并承担后果。如果条件允许，请尽可能使用自定义 ISO 镜像手动安装。
使用一键 DD 脚本

对于不支持自定义 ISO 镜像的服务商，使用一键 DD 脚本比较方便。开始前，务必花费几分钟读下作者的说明文档，这远比盲目粘贴命令重要。

    bin456789/reinstall 支持重装的系统类型更多
    bohanwood/debi 专注于纯净、精简的 Debian 系统环境

以 Oracle ARM 机器安装 Debian 为例，使用 debi 脚本的操作流程如下：

下载脚本

curl -fLO <https://raw.githubusercontent.com/bohanyang/debi/master/debi.sh> && chmod +x debi.sh

重装配置

sudo ./debi.sh \
  --version 13 \
  --architecture arm64 \
  --cloudflare \
  --user viamoe \
  --authorized-keys-url <https://github.com/githubUserName.keys> \
  --ssh-port 22122

参数说明：

    --version 13 指定 Debian 13（代号 Trixie）
    --architecture arm64 指定系统架构，这里的 arm64 匹配 ARM 架构机器
    --cloudflare 预设 Cloudflare 作为默认 DNS
    --user viamoe 创建具有 sudo 权限的普通用户 viamoe
    --authorized-keys-url https://github.com/githubUserName.keys 导入 SSH 公钥，实现无密码登录
    --ssh-port 22122 更改默认 SSH 端口，规避低级暴力扫描

开始重装

重启后，机器将自动联网重装系统

sudo reboot

期间可通过 VNC 观察安装进度，一般几分钟后新系统即可就绪。
使用自定义 ISO 镜像

若服务商支持上传自定义 ISO 镜像手动安装，这是最为推荐的方式，能让我们从源头掌控系统环境的纯净度。针对 Netcup 服务器的具体操作可参考：Netcup 服务器安装自定义 ISO 镜像。

完成重装后，使用预设的用户凭据登录，开始后续配置。
配置用户

赋予普通用户 sudo 权限，方便后续使用和管理。

# 切换到 root 用户

su

# sudo 免密码

# 替换 dejavu 为实际用户名

echo "dejavu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/dejavu

# 设置权限

chmod 440 /etc/sudoers.d/dejavu

SSH 安全加固
配置公钥登录

# 退出 root 用户

exit

# 创建公钥目录

mkdir -p ~/.ssh

# 粘贴 SSH 公钥

vim ~/.ssh/authorized_keys

# 或者上传 SSH 公钥

# ssh -i /path/to/your/ed25519_key username@<IP> -p <port>

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

优化 SSH 配置

sudo vim /etc/ssh/sshd_config

重点修改以下配置：

# 修改默认 SSH 端口

Port 22122

# 登录宽限时间

# 超过 1 分钟不输入密码/密钥自动断开

LoginGraceTime 1m

# 禁止 root 用户直接登录

PermitRootLogin no

# 严格模式，检查主目录权限

StrictModes yes

# 开启 SSH 公钥登录方式

PubkeyAuthentication yes

# 禁止传统密码登录方式

PasswordAuthentication no

# 禁止空密码登录

PermitEmptyPasswords no

# 禁用键盘交互式认证（防止通过 PAM 绕过密码限制）

KbdInteractiveAuthentication no

# Debian 默认需要 yes，否则可能会影响部分会话功能

UsePAM yes

# 关闭 X11 转发

X11Forwarding no

# 保持 SSH 会话连接

# 服务器每 60 秒发一次心跳包

ClientAliveInterval 60

# 如果客户端 5 次没回应才断开

ClientAliveCountMax 5

# 禁止 DNS 反向解析

UseDNS no

保持当前终端会话，新开一个 SSH 连接，确认正常，否则回到保持的 SSH 会话中检查配置。

# 检查 SSH 配置

sudo sshd -t

# 使新的 SSH 配置生效

sudo systemctl restart sshd

IPv6 静态路由

Netcup 服务器提供 /64 的 IPv6，建议设置静态路由

sudo vim /etc/network/interfaces

添加

iface ens3 inet6 static
    address <静态 IPv6 地址>
    netmask 64
    # Netcup network gateway
    gateway fe80::1
    # 接受路由公告
    accept_ra 2

然后执行

sudo ip addr flush dev ens3

更换 Linux 内核

推荐使用针对虚拟化优化、更加轻量和高效的 Cloud 内核。

sudo apt install -y linux-image-cloud-amd64

重启加载新内核

sudo reboot
uname -r

# 输出

6.12.63+deb13-cloud-amd64

清理旧内核

# 列出已安装的内核包

sudo dpkg --list | grep linux-image

# 输出示例

ii  linux-image-6.12.63+deb13-amd64       6.12.63-1                            amd64        Linux 6.12 for 64-bit PCs (signed)
ii  linux-image-6.12.63+deb13-cloud-amd64 6.12.63-1                            amd64        Linux 6.12 for x86-64 cloud (signed)
ii  linux-image-amd64                     6.12.63-1                            amd64        Linux for 64-bit PCs (meta-package)
ii  linux-image-cloud-amd64               6.12.63-1                            amd64        Linux for x86-64 cloud (meta-package)

# 移除常规内核元包及具体版本（请根据实际输出的版本号替换）

sudo apt purge -y linux-image-amd64 linux-image-6.12.63+deb13-amd64

# 更新引导

sudo update-grub

# 运行清理

sudo apt autoremove --purge -y

安装常用软件包

按需安装基础软件包及常用应用程序。
基本工具包

sudo apt update && apt upgrade
sudo apt install \
    apt-transport-https \
    build-essential \
    git \
    curl \
    wget \
    unzip \
    tmux \
    btop \
    bind9-dnsutils \
    tree \
    vim

安装 Docker

参考 Docker 官方文档 说明的步骤，下列命令不保证时效性

# 添加 Docker 官方的 GPG 密钥

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL <https://download.docker.com/linux/debian/gpg> -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 添加软件源

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl status docker

补充资料：给 Docker 启用 IPv6 支持
安装 Nginx

同样地，参考 Nginx 项目文档——在 Debian 上的 安装步骤

sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring

# 导入 GPG 密钥

curl <https://nginx.org/keys/nginx_signing.key> | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

# 验证 GPG 密钥

mkdir -m 700 ~/.gnupg
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
<http://nginx.org/packages/debian> `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

# 设置仓库锁定，优先使用 Nginx 官方提供的软件包

echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx

# 安装 Nginx

sudo apt update && sudo apt install nginx

# 启动 Nginx

sudo systemctl start nginx

使用自签 SSL/TLS 证书，设置 Nginx 默认回退 (Fallback) 虚拟主机。

# 准备目录

sudo mkdir -p /etc/nginx/cert
sudo chmod 700 /etc/nginx/cert

# 生成证书

sudo openssl req -x509 -new -nodes -newkey rsa:2048 \
  -sha256 \
  -days 3650 \
  -keyout /etc/nginx/cert/deny.key \
  -out /etc/nginx/cert/deny.pem \
  -subj "/C=XX/ST=Denied/L=Denied/O=Denied/CN=invalid.local" \
  -addext "subjectAltName=DNS:invalid.local"
  
# 设置权限

sudo chmod 600 /etc/nginx/cert/deny.key
sudo chmod 644 /etc/nginx/cert/deny.pem

新建虚拟主机配置文件

sudo vim /etc/nginx/conf.d/00-default.conf

内容如下：

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name_;
    return 444;
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name_;

    ssl_certificate /etc/nginx/cert/deny.pem; 
    ssl_certificate_key /etc/nginx/cert/deny.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    error_page 497 =444 /dev/null;

    return 444;
}

重载 Nginx 配置

sudo nginx -t
sudo nginx -s reload

设置 ZRAM 和 Swap

合理配置 ZRAM 和 Swap，平衡硬盘 I/O 性能和 CPU 开销。
配置 ZRAM

ZRAM 通过在 RAM 中划分一块区域压缩，速度远快于硬盘上的 Swap 空间。

# 安装 zram-tools 管理工具

sudo apt update && sudo apt install -y zram-tools

编辑 ZRAM 配置

sudo vim /etc/default/zramswap

参数说明：

# 选择 zstd 算法，最佳平衡压缩比和速度

ALGO=zstd

# 分配 ZRAM 比例

# 1GB RAM 建议 100%

# 2~4GB RAM 建议 60%

# 8GB RAM 以上建议 25%

PERCENT=70

# 优先级 100，确保系统优先使用 ZRAM

PRIORITY=100

配置 Swap

Swapfile 比 Swap 分区更为灵活

# 创建 2GB 的 Swapfile

# 1024 * 2 = 2048

sudo dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress

# 设置权限

sudo chmod 600 /swapfile

# 格式化

sudo mkswap /swapfile

# Swap 启用优先级低于 ZRAM

sudo swapon --priority -2 /swapfile

# 启动时自动挂载

echo '/swapfile none swap sw,pri=-2 0 0' | sudo tee -a /etc/fstab

# 检查挂载

sudo mount -a

系统内核调整

根据物理 RAM 大小，调整设置系统使用 Swap 的「积极性」。

# 1GB RAM（激进）

echo "vm.swappiness=100" | sudo tee -a /etc/sysctl.conf

# 2GB/4GB RAM（积极）

echo "vm.swappiness=60" | sudo tee -a /etc/sysctl.conf

# 8GB RAM 以上（保守）

echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

对于 2GB RAM 以下的机器，应当更倾向于释放文件缓存：

echo "vm.vfs_cache_pressure=50" | sudo tee -a /etc/sysctl.conf

应用配置

sudo sysctl -p

验证状态

sudo swapon --show
sudo zramctl

SSD Trim 优化

如果 VPS/VDS 使用的是 NVMe SSD，建议启用 Trim 自动优化。

# 定时任务

sudo systemctl enable --now fstrim.timer

并非所有服务商提供的 VPS/VDS 都支持 Trim，可以验证下：

# DISC-MAX 列不为 0

lsblk -D
NAME   DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
sr0           0        0B       0B         0
zram0         0        4K       2T         0
vda           0      512B       2G         0
├─vda1        0      512B       2G         0
└─vda2        0      512B       2G         0

# 尝试手动触发

sudo fstrim -v /
/: 434.4 GiB (466435428352 bytes) trimmed

NTP 时区同步
设置 UTC 时区

无论服务器位于何处，我都习惯将时区统一设置为 UTC（世界协调时），避免跨时区运维时产生的日志混乱和时间差纠纷，是一个非常省心的「隐性福利」。

sudo timedatectl set-timezone UTC

使用 Chrony 同步时间

与默认的 systemd-timesyncd 比，Chrony 的精度和性能更佳。

# 安装 Chrony 客户端

sudo apt update
sudo apt install chrony

添加 NTP 服务器

sudo vim /etc/chrony/chrony.conf

注释默认的 pool 及 server 行，添加 Cloudflare NTP 节点：

server time.cloudflare.com iburst
server time.cloudflare.com iburst
server time.cloudflare.com iburst
server time.cloudflare.com iburst

启动 Chrony 服务

sudo systemctl mask systemd-timesyncd.service
sudo systemctl enable --now chrony

验证同步状态

chronyc sources -v

配置 nftables 防火墙

nftables 是 Linux 内核现代化的包过滤框架。在 Debian 13 中，我们应该摒弃过时的 iptables 和 UFW，直接拥抱原生且高效的 nftables。
检查环境

从 Debian 10 开始，nftables 是默认安装的，但通常处于未启用状态，检查：

# 确保 nftables 已安装

sudo apt update && sudo apt install nftables -y

# 默认 inactive 状态是正常的

sudo systemctl status nftables

若此前安装过 UFW，建议卸载以免冲突

sudo ufw disable && sudo apt purge ufw -y

理解配置

在编写规则前，只需掌握三个层级：

    表（Table）：规则的顶级容器

    链（Chain）：绑定在网络钩子（INPUT/FORWARD/OUTPUT）上的规则集，定义默认策略（Drop 或 Accept）

    规则（Rule）：具体的匹配条件与行动

开始使用

以下配置是典型的「安全」服务器方案：仅允许 Cloudflare CDN 回源访问 80/443 端口，除 SSH 端口外全部切断。

# 编辑配置文件

sudo vim /etc/nftables.conf

语法很简单，可以看注释

# !/usr/sbin/nft -f

# 刷新规则

flush ruleset

table inet filter {
    # ============================================================
    # IP 集合 (Sets)
    # ============================================================
    set cloudflare_v4 {
        type ipv4_addr; flags interval;
        elements = {
            173.245.48.0/20,
            103.21.244.0/22,
            103.22.200.0/22,
            103.31.4.0/22,
            141.101.64.0/18,
            108.162.192.0/18,
            190.93.240.0/20,
            188.114.96.0/20,
            197.234.240.0/22,
            198.41.128.0/17,
            162.158.0.0/15,
            104.16.0.0/13,
            104.24.0.0/14,
            172.64.0.0/13,
            131.0.72.0/22
        }
    }

    set cloudflare_v6 {
        type ipv6_addr; flags interval;
        elements = {
            2400:cb00::/32,
            2606:4700::/32,
            2803:f800::/32,
            2405:b500::/32,
            2405:8100::/32,
            2a06:98c0::/29,
            2c0f:f248::/32
        }
    }

    # ============================================================
    # INPUT 链 (入站)
    # ============================================================
    chain input {
       # 规则外流量拒绝
        type filter hook input priority filter; policy drop;

        # 放行已建立连接
        ct state established, related accept
        # 丢弃无效包
        ct state invalid drop

        # 允许回环接口
        iif "lo" accept
        # 允许 IPv6 NDP
        ip6 nexthdr icmpv6 icmpv6 type { nd-neighbor-solicit, nd-neighbor-advert, nd-router-advert } accept

        # 允许 ICMP (速率限速)
        ip protocol icmp limit rate 4/second accept
        ip6 nexthdr icmpv6 icmpv6 type echo-request limit rate 4/second accept

        # 允许 SSH 端口
        tcp dport 22122 accept

        # 仅限 Cloudflare IP 段访问 80/443
        ip saddr @cloudflare_v4 tcp dport { 80, 443 } accept
        ip6 saddr @cloudflare_v6 tcp dport { 80, 443 } accept
    }

    # ============================================================
    # FORWARD 链 (转发)
    # ============================================================
    chain forward {
        # 规则外流量拒绝
        type filter hook forward priority filter; policy drop;

        # 放行已建立连接
        ct state established, related accept
        # 丢弃无效包
        ct state invalid drop

        # 允许 Docker 出站 (覆盖默认网桥和自定义网桥)
        iifname "docker0" accept
        iifname "br-*" accept

        # 允许 Docker 容器间通信
        iifname "docker0" oifname "docker0" accept
        iifname "br-*" oifname "br-*" accept
    }

    # ============================================================
    # OUTPUT 链 (出站)
    # ============================================================
    chain output {
        # 默认允许所有出站
        type filter hook output priority filter; policy accept;
    }
}

应用规则

# 检查语法

sudo nft -c -f /etc/nftables.conf

# 启动服务

sudo systemctl enable --now nftables

# Docker 重启后会自动注入规则

sudo systemctl restart nftables && sudo systemctl restart docker

# 测试 Docker 出站

sudo docker run --rm busybox ping -c 4 dejavu.moe

安装配置 Fail2ban

# 安装 Fail2ban

sudo apt update && sudo apt install fail2ban

# 创建一个最小的 SSH 服务保护配置

sudo vim /etc/fail2ban/jail.local

配置内容如下：

[DEFAULT]

# 本机地址自动忽略，防止误封自己

ignoreip = 127.0.0.1/8 ::1

# 封禁 1 天

bantime  = 1d

# 在 10 分钟内累计失败即触发

findtime = 10m

# 触发封禁的失败次数阈值

maxretry = 3

# 自动 nftables 规则插入

banaction = nftables-multiport
banaction_allports = nftables-allports
[sshd]

# 启用 SSH 保护

enabled = true

# SSH 服务的监听端口（务必正确匹配）

port    = 22122

# 使用 systemd 日志后端

backend = systemd

# 严格地匹配失败日志

mode    = aggressive

启动 Fail2ban 服务

sudo systemctl enable --now fail2ban
sudo systemctl restart nftables && sudo systemctl restart fail2ban

测试一下封禁

# 封禁一个「黑户」

sudo fail2ban-client set sshd banip 2400:6180:0:d2:0:2:9699:d000

# 检查 nftables 规则中是否存在 f2b 动态表

sudo nft list ruleset | grep f2b

检查 Fail2ban 配置

# 巡查「监狱」

sudo fail2ban-client status

# 查封禁 IP

sudo fail2ban-client status sshd

至此，一台纯净、安全的服务器已经准备就绪。Happy Hosting!

另请参阅：
