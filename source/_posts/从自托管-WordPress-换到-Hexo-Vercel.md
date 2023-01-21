---
title: 从自托管 WordPress 换到 Hexo + Vercel
date: 2023-01-21 14:43:45
tags:
---

WordPress 真的好, 插件众多, 主题好看, 发布方便, 易于部署, 简直就是博客系统神器.

but, 是真的吗?

## WordPress 为什么不适合小型博客

1. 使用 PHP 开发. 虽然说用 PHP 没什么不好, 但是这注定了 WordPress 加载速度比使用 NodeJS 等技术开发的网站慢.
2. 过于臃肿. WordPress 功能完全, 插件众多是好的. 但是很多小型博客是用不到这些功能的(例如用户注册).
3. 吃服务器性能. 以上缺点在服务器性能好, 优化正确的情况下完全可以忽略. 但是一台好的服务器动辄每月上百块, 还要购买 CDN 服务等, 作为学生无法负担.

## Hexo 为什么适合小型博客

1. 使用 NodeJS 开发, 静态页面, 从根源上解决了页面渲染慢的问题.
2. 可以自己添加所需功能, 十分轻量.
3. 可以部署在 Serverless 服务中, 例如 Vercel, 无需关注服务器费用.

> 我更看中的是原生 Markdown 支持, WordPress 上的插件多多少少都有点问题

## Hexo 的缺点

1. 其优点也是其缺点, 因为 Hexo 是静态博客, 因此无法做到 WordPress 一样的即时发布.
2. 需要一点前端开发经验, 否则照着网上的教程和命令是无法把 Hexo 玩明白的.

## 从 WordPress 换到 Hexo + Vercel

说了这么多应该进入正题了, Hexo 这么好, 如何使用他部署一个站点呢?

PS: 本文章只负责教会快速部署一个 Hexo 站点, 不负责教会使用.

### Github 篇

首先你需要一个 Github/GitLab/BitBucket 账号, 你的博客仓库将托管到这些平台上面, 这里我推荐全球最大的同性交友网站 Gayhub(什么?).

免费推广: [KiteAB - GitHub 到底怎么用？十分钟学会 GitHub 基础知识](https://www.bilibili.com/video/BV1yo4y1d7UK/)

### NodeJS & Git 篇

Hexo 是基于 NodeJS 开发的, 因此你需要在你的计算机安装 NodeJS 以管理你的博客.

Git 是一个开源的分布式版本控制系统, 需要使用它来管理你的博客仓库.

**我认为都已经打算使用 Hexo 了, 不应该不会查资料, 因此这一部分交给 Hexo 官方文档**.

直达链接: [https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)

按照官方文档配置完环境后不要直接使用 `hexo init` 创建站点(因为这样会让自动部署麻烦一点).

### Vercel 篇

使用你的 Github 账号登陆 [Vercel](https://vercel.com) 并授予其所有权限, 直达模板 -> Hexo 然后创建.

此时你的网站已经可以访问了.

创建完成后使用 Git 把仓库 `clone` 下来进行下一步操作.

### 本地操作篇

打开系统终端(什么你都到这一步了都不知道什么是终端, 建议用回 WordPress), 进入你的项目并执行 `yarn install` .

> 这里我使用了 yarn 包管理器而不是 npm, 因为 yarn 的装包速度更快.
>
> 要使用 yarn , 请执行 `npm install yarn -g`

然后你就可以使用 `hexo` 命令管理你的博客了. So ez right?

### 数据迁移篇

我的博客是直接使用 Markdown 写的, 因此迁移对我来说不是问题.

如果你使用 WordPress 块编辑器写的, 我建议安装一个 Markdown 插件(找安装量最高的那款), 并开启 html 转 Markdown. 之后你就可以获得 Markdown 格式的文章了.

## 实际体验

比 WordPress 好.

目前我的主站点仍在使用 WordPress, Hexo 作为子站点上线. 待完全备份迁移后我会将 WordPress 删除并使用 Hexo 作为我的博客程序.

## 结语

推荐另一个项目 [xlog.app](https://xlog.app) , 这个项目使用 IPFS + 区块链技术开发, 实现了去中心化, 非常的酷.
