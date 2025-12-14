---
title: "最简邮局搭建教程|使用poste.io部署自己的邮局"
date: 2025-09-15T17:41:40+08:00
draft: true
description: ""
showComments: true
featureimage: "https://picsum.photos/seed/907fd2/1600/900.webp"
tags: ["折腾日记","邮局部署","自部署项目"]
series: ["自建服务"]
series_order: 2
---

文章详细描述Poste.io邮局服务器的搭建过程及常见的问题解决处理方法。

## 📜 **一、引言：什么是 Poste.io？**

Poste.io 是一款设计精巧、功能强大的开源邮件服务器解决方案。它致力于简化邮件系统的部署与日常管理，将复杂的配置流程整合为一体化的简易操作。官方网站的口号sologon为`SMTP + IMAP + POP3 + Antispam + Antivirus
Web administration + Web email ...on your server in ~5 minutes`，可见其部署容易简单，功能强大。

Poste.io 的核心优势包括：

* **协议支持全面**：原生支持 SMTP、IMAP 等主流邮件协议。
* **内置安全防护**：集成了高效的反垃圾邮件与反病毒引擎，为您的邮件通信保驾护航。
* **部署轻便快捷**：官方支持 Docker 容器化部署，极大地降低了安装和迁移的复杂度。相比其他邮件软件，内置管理用户界面，不需要单独组件进行配置选择。
* **适用场景广泛**：无论是个人爱好者还是中小型企业，`Poste.io` 都能满足其核心邮件收发需求，并提供灵活的扩展能力。

> 🔗 **官方链接**
>
> * **官网**: <https://poste.io/>
> * **官方文档**: [Poste.io documentation](https://poste.io/doc/)

---

## 🛠️ **二、部署前的准备工作**

在正式开始部署之前，请确保您已具备以下基础环境和资源。一个周全的准备是项目成功的关键。

* **服务器要求**
  * **公网IP**: 一台拥有独立公网 IP 的服务器。
  * **端口开放**: 确保服务器的 `25` 端口未被服务商封禁。
  * **反向DNS (rDNS)**: （可选配置项）强烈建议使用支持配置 PTR (rDNS) 记录的服务器，这对提升邮件送达率至关重要。

* **软件环境**
  * **Docker & Docker Compose**: 请在服务器上预先安装 Docker 和 Docker Compose。

* **域名**
  * 准备一个您拥有管理权限的域名，用于创建域名邮箱。

* **系统优化 (推荐)**
  * **Swap 交换分区**: Poste.io 在启用全部功能（特别是反病毒和反垃圾邮件模块，模块可以进行关闭以减少内存占用）时会占用较多内存。为保证系统稳定运行，建议为服务器配置 2-4GB 的 Swap 交换分区。

---

## 🌐 **三、域名 DNS 解析配置**

部署前，预先配置好域名解析是高效部署的关键步骤。假设您计划使用的邮箱后缀为 `@example.com`，邮件服务器的主机名为 `mail.example.com`。

> 请参考下表进行配置，**不要照抄下面的字段设置**一定要将将示例中的域名`example.com`和 IP 地址替换为您自己的信息。

### 3.1 基础解析

核心 DNS 记录一览表:

| 名称 (Host) | 记录类型 (Type) | 记录值 (Value) | 优先级 (Priority) | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `@` | `MX` | `mail.example.com` | `10` | 指定邮件交换服务器 |
| `mail` | `A` | `你的服务器 IP 地址` | - | 将邮件服务器主机名指向服务器 IP |
| `@` | `TXT` | `v=spf1 ip4:你的服务器 IP 地址 ~all` | - | SPF 记录，声明合法的发件服务器 |
| `xxxxxx._domainkey` | `TXT` | `v=DKIM1; k=rsa; p=...` | - | **DKIM 记录，公钥值需在部署后获取** |
| `_dmarc` | `TXT` | `v=DMARC1; p=none; rua=mailto:dmarc@example.com`| - | DMARC 策略，用于邮件认证 |

**记录详解：**

* 📨 **MX (Mail Exchange) 记录**: 告诉其他邮件服务器，所有发往 `example.com` 的邮件应该由 `mail.example.com` 这台服务器来接收。

* 📍 **A (Address) 记录**: 将 `mail.example.com` 这个主机名解析到您服务器的公网 IP 地址。

* 🛡️ **TXT 记录 (SPF, DKIM, DMARC)**: 这是确保邮件不被主流邮箱服务商（如 Gmail, Outlook）判断为垃圾邮件的关键三剑客。
  * **SPF (Sender Policy Framework)**: 用于防止他人伪造您的域名发信。
  * **DKIM (DomainKeys Identified Mail)**: 通过数字签名技术验证邮件的真实性和完整性。**请注意**：此记录的公钥值需要在 Poste.io 部署并生成密钥后才能获得（详见 `5.3` 节）。
  * **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**: 整合 SPF 和 DKIM 的验证结果，并告知收件方如何处理验证失败的邮件。

### 3.2 协议解析（可选）

为了方便客户端使用 IMAP、SMTP、POP 等协议连接邮件服务器，并确保邮件的正常发送和接收，建议您在 DNS 中添加以下记录。这些记录通常会指向您的邮件服务器主机名，例如 `mail.example.com`。

#### 3.2.1 A 记录配置

这些记录将各自的服务子域名解析到您的邮件服务器 IP 地址。

| 名称 (Host) | 记录类型 (Type) | 记录值 (Value) |
| :--- | :--- | :--- |
| `imap` | `A` | `你的服务器 IP 地址` |
| `smtp` | `A` | `你的服务器 IP 地址` |
| `pop` | `A` | `你的服务器 IP 地址` |

#### 3.2.2 CNAME 记录配置 (替代 A 记录)

您也可以选择使用 CNAME 记录，将这些子域名指向 `mail.example.com`，这样可以避免为每个服务单独设置 IP 地址。

| 名称 (Host) | 记录类型 (Type) | 记录值 (Value) |
| :--- | :--- | :--- |
| `imap` | `CNAME` | `mail.example.com` |
| `smtp` | `CNAME` | `mail.example.com` |
| `pop` | `CNAME` | `mail.example.com` |

#### 3.2.3 服务端口配置 (客户端)

邮件客户端连接服务器时需配置相应的服务地址和端口：

* **IMAP (Internet Message Access Protocol)**：用于收取邮件，通常地址为 `imap.example.com`，端口 `143` (非加密) 或 `993` (SSL/TLS 加密)。
* **SMTP (Simple Mail Transfer Protocol)**：用于发送邮件，通常地址为 `smtp.example.com`，端口 `25` (非加密)、`465` (SSL/TLS 加密) 或 `587` (STARTTLS 加密)。
* **POP (Post Office Protocol)**：用于收取邮件，通常地址为 `pop.example.com`，端口 `110` (非加密) 或 `995` (SSL/TLS 加密)。

---

## 🚀 **四、开始部署 Poste.io**

### **最简命令（快速但不推荐）**

```shell
docker run \
    --net=host \
    -e TZ=Asia/Shanghai \
    -v /your-data-dir/data:/data \
    --name "mailserver" \
    -h "mail.example.com" \
    -t analogic/poste.io
```

> 上面的命令使用`net=host`模式，具体可查看<https://docs.docker.com/network/host/>，可自动配置相应的端口。
>
### 基于docker-compose方法

1. **创建项目目录并进入**
  以下内容使用root用户，但是使用具有docker权限的用户也可进行。

  ```bash
  mkdir -p /root/data/docker_data/posteio
  cd /root/data/docker_data/posteio
  ```

2. **创建 `docker-compose.yml` 配置文件**

    ```bash
    vim docker-compose.yml
    ```

    将以下配置内容粘贴到文件中。请务必根据您的实际情况修改 `hostname`、`LETSENCRYPT_EMAIL` 和 `LETSENCRYPT_HOST` 的值。

    ```yaml
    services:
      mailserver:
        image: analogic/poste.io
        container_name: posteio
        restart: always
        hostname: mail.example.com  # ❗️替换为你的邮件服务器主机名
        ports:
          - "25:25"
          - "110:110"
          - "143:143"
          - "465:465"
          - "587:587"
          - "993:993"
          - "995:995"
          - "4190:4190"
          - "80:80"        # 用于 Let's Encrypt HTTP-01 验证
          - "443:443"      # 用于 HTTPS 访问
        environment:
          - HTTPS=ON
          - TZ=Asia/Shanghai
          - LETSENCRYPT_EMAIL=admin@example.com  # ❗️替换为你的管理员邮箱
          - LETSENCRYPT_HOST=mail.example.com     # ❗️替换为你的邮件服务器主机名
          # - DISABLE_CLAMAV=TRUE                 # 性能优化：取消注释可禁用反病毒，显著降低资源占用
          # - DISABLE_RSPAMD=TRUE                 # 性能优化：取消注释可禁用反垃圾邮件，显著降低资源占用
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - $PWD/mail-data:/data
    ```

    > **💡 性能提示**：对于资源有限的服务器，可以通过取消注释 `DISABLE_CLAMAV` 和 `DISABLE_RSPAMD` 来禁用反病毒和反垃圾邮件功能，这将大幅降低内存和 CPU 的占用。

3. **启动服务**
    编辑完成后，保存并退出文件 (`:wq`)，然后运行以下命令以后台模式启动容器：

    ```bash
    docker-compose up -d
    ```

---

## ⚙️ **五、初始化与配置**

部署成功后，通过浏览器访问 `https://mail.example.com` (请替换为您的主机名) 即可进入 Web 管理界面。![no_https](/img/no_https.png)

### 5.1 注册管理员账户

首次访问时，浏览器可能会提示“隐私错误”或“证书无效”，这是因为初始证书是自签名的，请选择“继续前往”即可。进入页面后，创建您的第一个管理员账户和密码。![注册管理用户](/img/reg_admin.png)。

> 需要注意的是**邮件管理员**的登录地址为`https://mail.example.com/admin/login`,普通用户登录地址<https://mail.example.com/webmail/>。 两者区别为管理员用户可添加普通用户、设置服务器、查看服务器状态等等高权限操作，普通用户不具有相应的权限。

### 5.2 配置 Let's Encrypt SSL 证书

登录后，导航至 **System Settings -> TLS Certificate**。在 `Let's Encrypt domain` 字段中输入您的邮件服务器主机名（例如 `mail.example.com`），然后点击 **Save Changes**。系统将自动申请并配置免费的 SSL 证书。
![配置证书](/img/lets-cert.png)
![设置证书域名](/img/lets-domain.png)

### 5.3 生成并配置 DKIM 密钥

这是提升邮件信誉度的关键一步。

1. 导航至 **Virtual domains**，点击您的域名（例如 `example.com`）。
   ![设置域名](/img/set_domain.png)
2. 在域名配置页面中，找到 **DKIM** 部分，点击 **Create DKIM key**。
   ![alt text](/img/create-key.png)
3. 系统会生成一段 TXT 记录。**请将这段记录完整地添加到您域名的 DNS 解析中**。这就是我们在 `3.1` 节中预留的 DKIM 记录。解析生效通常需要几分钟到几小时不等。
  ![alt text](/img/get-key.png)

### 5.4 Webmail 日常使用

在管理界面的右上角，点击 **Webmail** 按钮，即可跳转到邮件收发界面。使用您刚才创建的账户登录，即可开始体验一个功能齐全、界面简洁的 Web 邮箱。

![alt text](/img/admin-webmail.png)
![alt text](/img/email.png)

---

## ❓ **六、常见问题记录**

1. 邮件测试教程
推荐测试网站<http://www.mail-tester.com/>检测你的邮件分数， 满分就基本确定你的自建邮箱可用且发送的邮件不会被识别成垃圾邮件！

  > 测试步骤为：
  > **1. 向测试邮箱发信**：打开网站后会给你一个测试邮箱，点击复制后，在你的邮箱里发一份邮件，建议内容随便复制一段话(20个字左右即可) ❗不要留空,会减低评分
  >
  > **2.点击查看你的邮件得分**。
  ![alt text](/img/email-grade.png)
  >
  > **3.等待结果**。如果是10分，恭喜你不用再折腾，如果不是，去下面没打勾的测试点击下拉查看详情并修改你的自建邮箱配置。一般来说自建邮箱开启DKIM认证非常重要，它使邮件可信且识别为重要。

2. WebEmail故障排除

  登录管理员用户，选择左侧用户栏`Server status`可以查看占用登录情况，`Connection diagnostics`可以查看端口开放情况。大多数情况下：如果发现25端口显示为红色，则说明25端口被服务商禁用，当前邮局可接收邮件但无法发送。解决方法有：使用中继服务（resend等）、向服务商申请等。
  ![查看端口情况](/img/check-port.png)
