---
title: "Git常见的开发工作流"
description: ""
date: 2026-01-04T10:32:23+08:00
lastmod: 2026-01-04T10:32:23+08:00
summary: "示例摘要"
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/cf5002/1600/900.webp"
tags: ["自我折腾"]
series: ["建站技术"]
series_order: 1
---

## 序言

很多入职开发前的伙伴, 可能或多或少接触过 git,实际用的最多的命令也就是 `git clone`,`git push`,
以为 git 的流程也就这样, 实则不然, 实际上企业中的 git 流程, 由五大分支构成

## 理论

[![fI6dAXt_8](https://linux.do/uploads/default/optimized/4X/c/2/3/c2318e85908126eddbac17a2025568881fe71465_2_689x420.webp)
fI6dAXt_82424×1476 111 KB
](<https://linux.do/uploads/default/original/4X/c/2/3/c2318e85908126eddbac17a2025568881fe71465.webp> "fI6dAXt_8")
> 提示
>
> 只有 `main` 主分支和 `develop` 开发分支是贯穿项目生命周期本身, 其他分支都会中途离场  
> 所以也说 `main` 和 `develop` 是上二分支, 其他分支是下三分支

- **主分支 (main/master)**: 项目的生产发布分支, 始终保持稳定可发布状态
- **热修复分支 (hotfix/*)**: 从 `main` 分支创建, 用于紧急修复线上 BUG, 修复后同时合并回 `main` 和 `develop`
- **开发分支 (develop)**: 日常开发的主分支, 所有功能开发的集成分支
- **特性分支 (feature/*)**: 从 `develop` 创建, 开发新功能, 完成后合并回 `develop`
- **预发布分支 (release/*)**: 从 `develop` 创建, 用于发布前的测试和最终调整, 测试通过后合并回 `main`(打版本标签) 和 `develop`

## [](https://linux.do/t/topic/1402698#p-12088756-h-3)分支合并关系

`feature/* → develop  develop → release/*  release/* → main + develop  hotfix/* → main + develop`

## [](https://linux.do/t/topic/1402698#p-12088756-h-4)主分支

作用: 存放 **稳定**, 可**随时部署到生产环境**的代码
特点:

- 分支上每一个提交对都应一个正式发布的版本
- 不允许在此分支直接开发
- 通常被打上版本标签 (如: v1.0.0 或 v20260101)

## [](https://linux.do/t/topic/1402698#p-12088756-h-5)开发分支

作用: 存放 **最新开发成果** 的集成分支, 是功能开发的集线器
特点:

- 当 `develop` 分支上的代码到达稳定状态并准备发布时, 会合并到 `main` 分支 (如果有 `release` 分支, 则合并到此分支)
- 所有功能的分支、发布分支都从 `develop` 分支拉取

## [](https://linux.do/t/topic/1402698#p-12088756-h-6)功能分支

来源: `develop`
合并目标: `develop`
命令惯例:`feature/user-authentication`,`feature/payment-integration`
作用: **开发新功能**
生命周期:

- 从 `develop` 分支拉取
- 开发完成后, 合并回 `develop`
- 合并后, 该功能分支通常被删除

## [](https://linux.do/t/topic/1402698#p-12088756-h-7)发布分支

来源: `develop`
合并目标:`develop` 和 `main`
命名惯例:`release/1.2.0`,`release/2026-spring`
作用: 为发布新版本做准备. 在此分支上, 只做 **BUG 修复**,**生成版本号**,**整理文档** 等发布准备工作, **不再添加新功能**
生命周期:

- 当 `develop` 分支的功能足够进行一次发布时, 从 `develop` 拉出 `release` 分支
- 在此分支上进行最后的测试和修复
- 准备就绪后, 将 `release` 分支合并到 `main` 分支 **并打上版本标签**
- 同时, 必须合并回 `develop` 分支, 因为 `release` 分支上修复的 BUG 可能 `develop` 分支上还没修复

## [](https://linux.do/t/topic/1402698#p-12088756-h-8)热修复分支

来源:`main`
合并目标: `main` 和 `develop`
命名惯例:`hotfix/critical-security-patch`,`hotfix/1.2.1`
作用:快速修复生产环境(`main`分支)上的紧急BUG
生命周期:

- 从 `main` 分支上出现BUG的提交点(通常是最近的标签)拉取
- 修复完成后,合并回`main`分支**并打上新的版本标签**(如:v1.0.1)
- 同时,必须合并回 `develop` 分支,确保修复在后续开发中也生效

## [](https://linux.do/t/topic/1402698#p-12088756-h-9)人话
>
> 恭喜你能看到这里, 理论看起来确实是枯燥的, 感谢你自己的坚持,
>
> 希望评论区就 gitflow 能展开相关讨论, 而不是一味的刷 感谢分享 感谢大佬,
>
> 我分享文章也是希望有思想和经验上的交流碰撞
实际上,普通的开发入职后,如果有开发需求,一般都是从`develop`分支拉取代码后,本地新建`feature/xxx 功能`分支,在`feature/xxx`分支上进行开发的。功能开发到一定阶段后,比如一阶段完成了核心需求(因为产品可能在开发内提出新需求,这很常见),
就可以合并到 `develop` 分支, `develop` 在积累了多个功能分支合并后,
就可以发布到 `release` 发布分支, 这一分支不再接受新功能合并, 只允许修补 BUG
这一过程就是提测, 让测试工程师核验功能完整性, 如果有 BUG 要及时修复
修复的时候拉取的是 `release` 分支,然后提交的时候也是推送到这个分支
在`release` 分支准备完善 (修复了测试工程师检测到的 BUG),就可以准备往主分支 (main) 发布了,
这就是一次完整的 git flow 发布流程, 当然还有线上有 BUG, 需要做紧急修复
这个时候, 就是直接拉取 main 分支并在本地创建热修复分支 hotfix,
在 BUG 被修复后, 就可以合并回 main 分支 和 develop 分支了
**是的, 这里要注意, hotfix 要合并到 2 个分支里,**
而不是 hotfix->main->develop 这是**错误的做法**, 因为造成 develop 被 main 污染 (额外的标签记录以及其他)
