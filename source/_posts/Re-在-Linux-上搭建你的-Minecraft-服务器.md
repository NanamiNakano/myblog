---
title: '在 Linux 上搭建你的 Minecraft 服务器'
date: 2022-11-22 20:03:16
tags:
---

## 前言 · Linux Server 并没有那么难

Windows Server 作为 GUI 最成熟的服务器系统深受许多 Minecraft 服务器服主的喜爱, 但是其系统占用率高, 功能冗杂, 并不是很适合长期作为服务器系统使用. Linux 系统占用率很低, 而且升级维护不需要重启服务器, 非常方便. 新手服主可能会被 Linux CLI 劝退, 这篇指南旨在帮助你使用 Linux Server 搭建 Minecraft 服务器.

## 前期准备

> 系统选择

如果没有特殊环境需求, 主流发行版都能满足需求, 本指南使用 Ubuntu Server 22.04 LTS 作为系统版本.

> 软件准备

与服务器通信的 SSH 连接工具, FTP 工具(可选).

> 本文格式

在 Bash 命令前会添加 ~$ 以表明此行是 Bash 命令, 未添加即为命令行输出, 添加 # 则表明此行是上一行的注释.

## 安装环境

### 安装 Java[^1]

```bash
~$ sudo apt install openjdk-19-jdk
```

> 检验 Java 环镜

```bash
~$ java --version
openjdk 19.0.1 2022-10-18
OpenJDK Runtime Environment (build 19.0.1+10-Ubuntu-1ubuntu122.04)
OpenJDK 64-Bit Server VM (build 19.0.1+10-Ubuntu-1ubuntu122.04, mixed mode, sharing)
```

### 安装 LNMP 环境(可选)

因为我的服务器有插件需要用到 MySQL, 而我懒得手动配置 phpMyAdmin, 因此使用 LNMP 环境一次解决. 有类似需求的也可仅安装 MySQL 手动配置, 本文不详细展开. 

```bash
# 使用 lnmp.org 的无人值守安装脚本
~$ sudo apt install screen
# 安装 screen 以保证进程不被系统杀死并可随时回到终端查看状况
~$ screen -S lnmp
~$ su
# 使用 root 用户, 若未设置过 root 用户密码请用 sudo passwd root 命令设置
# 请勿一直使用 root 用户进行操作, 会有安全隐患
~$ wget http://soft.vpser.net/lnmp/lnmp1.9.tar.gz -cO lnmp1.9.tar.gz && tar zxf lnmp1.9.tar.gz && cd lnmp1.9 && LNMP_Auto="y" DBSelect="4" Bin="y" DB_Root_Password="<数据库  Root 用户密码>" InstallInnodb="y" PHPSelect="12" SelectMalloc="2" ./install.sh lnmp
# 使用 lnmp.org 生成的无人值守安装命令安装, 请勿直接复制此处命令使用!!!
Install lnmp takes 13 minutes.
Install lnmp V1.9 completed! enjoy it.
~$ exit
# 退出 root 用户
exit
```

## 配置服务端

在 home 目录[^2]下创建一个新目录, 用来作为服务端根目录

```bash
~$ cd
# 回到 home 目录
~$ mkdir server
~$ cd ./server
# 进入新目录
```

你可以选择在个人 PC 上配置完后使用 FTP 上传到服务器运行, 也可以选择在服务器上配置. 本文不详细展开

## 运行服务端

在服务器根目录下创建一个新文件, 重命名[^3]为 start.sh 作为启动脚本

```bash
~$ > start.sh
```

这里使用 Aikar.co 提供的 JVM 参数

```bash
~$ nano start.sh
# 使用 nano 打开文件后, 键入 JVM 参数
# java -Xms6G -Xmx6G -XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:MaxGCPauseMillis=100 -XX:+DisableExplicitGC -XX:TargetSurvivorRatio=90 -XX:G1NewSizePercent=50 -XX:G1MaxNewSizePercent=80 -XX:G1MixedGCLiveThresholdPercent=35 -XX:+AlwaysPreTouch -XX:+ParallelRefProcEnabled -Dusing.aikars.flags=mcflags.emc.gs -jar paperclip.jar nogui
# 根据需要修改 -Xms 与 -Xmx 参数
```

按 Ctrl + O 并回车保存, 按 Ctrl + X 退出 nano[^4]

运行脚本

```bash
~$ screen -S server
~$ bash ./start.sh
[Server thread/INFO]: Time elapsed: 4624 ms
[Server thread/INFO]: Done (5.381s)! For help, type "help"
```

至此, Minecraft 服务端已成功运行, 断开 SSH 连接后, 输入

```bash
~$ screen -r server
```

回到服务端终端[^5]

## 文章引用及脚注

[fabric](https://fabricmc.net/)

[Aikar: 调整JVM —— 非常有效的服务器启动参数](https://www.mcbbs.net/thread-867786-1-1.html)[^6]

[^1]: 需要根据 Minecraft 版本选择正确的 Java 版本
[^2]: 即输入命令 cd 回到的目录, 也可根据现实情况选择其他目录
[^3]: 注意文件后缀也要修改
[^4]: 如果你的个人电脑系统是 macOS, 请使用 SSH 工具映射的按键代替 Ctrl
[^5]: 命令不包含 ~$
[^6]: 此链接为转载链接, 原页面为英文不方便阅读, 转载及翻译者不是我且与我无关

