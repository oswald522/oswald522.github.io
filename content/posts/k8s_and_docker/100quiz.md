---
title: "K8S&Docker 100问题"
description: ""
date: 2025-07-02T08:51:41+08:00
Lastmod: 2025-07-02T08:51:41+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/8b3cf9/1600/900.webp"
tags: ""
series: "K8s教程"
series_order: 2
---

## Docker基础篇

基础篇主要包含

* Docker常用的命令集合

```shell
curl -sSL https://get.docker.com | bash 
usermod -aG docker $USER
newgrp docker
```

* `docker attach` 和 `docker exec`的区别

* Dockerfile编写技巧及镜像优化方法
   1. `COPY` 和 `ADD`区别
   > 使用COPY复制指定目录下的所有文件到容器，不包括本级目录。
   > ADD
   > 使用COPY和ADD时，可以使用`--chown=<user>:<group>`直接进行文件权限的更改，这样就不用使用RUN在容器里面执行chown，不仅方便，还可以缩小镜像的体积。
   2. `CMD` 和 `ENTRYPOINT`的区别
   > CMD在使用docker run时可以被直接覆盖，比如我们使用docker run启动centos:env-cmd镜像，并在后面指定启动命令，此时CMD就会被覆盖；在ENTRYPOINT指定的不能被直接覆盖，后置的命令会被当作ENTRYPOINT的参数。ENTRYPOINT也是可以被覆盖的，需要指定--entrypoint参数，比如我们指定entrypoint为ls，后置命令为/tmp，就相当于ENTRYPOINT是ls，CMD是/tmp.
* 多阶段构建的含义，方法和具体操作
  > 通俗来讲，多阶段构建就是在Dockerfile中定义多个FROM，每个FROM下有多个不同的指令，一般可简单分为构建步骤和生成业务应用镜像步骤，也就是说前面一个或多个阶段用于构建，产生业务应用的包或其他产物，之后的阶段将上述阶段产生的包或其他产物再次构建成镜像。这样一来，最后一步就没有了构建时产生的缓存文件，也起到了优化镜像体积的作用。
* 对于Docker多阶段构建的例子，提供配置文件并进行详细说明

## K8S 基础篇

* K8s适用的场景，解决的主要问题，优缺点
* K8s架构解析、主要组成及各部分功能特性
* 涉及的基础概念：
  * Pod：Pod可被建模为一组具有共享命名空间、卷、IP地址和端口的容器。
  * Service
* Pod镜像拉取策略和重启策略
* 给出一个创建Pod的标准格式（YAML格式）

## K8S服务部署调度篇

该部分章节主要讲述

* Replication Controller和ReplicaSet 简单总结和回顾
  * Replication Controller可以确保Pod副本数达到期望值，也就是RC定义的数量。换句话说，Replication Controller可以确保一个Pod或一组同类Pod总是可用的。
  * ReplicaSet是支持基于集合的标签选择器的下一代ReplicationController，它主要用作Deployment协调创建、删除和更新Pod，和ReplicationController唯一的区别是，ReplicaSet支持标签选择器。
* 无状态应用管理Deployment
* 有状态应用管理StatefulSet
* 守护进程集DaemonSet
* CronJob

## K8S服务发布+配置管理篇

### 服务发布Service和Ingress

### 配置管理ConfigMap和Secret

* 两者的区别
  * 相对于Secret，ConfigMap更倾向于存储和共享非敏感、未加密的配置信息，假如是在集群中使用敏感信息，最好使用Secret。
