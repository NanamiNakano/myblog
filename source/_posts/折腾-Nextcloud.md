---
title: 折腾 Nextcloud
date: 2022-11-30 20:06:48
tags:
---

Nextcloud 发布了最新版 25.0.1，更强大，更难装，这里讲讲 Nextcloud 一些报错解决方案

## 一些文件未通过完整性检查。

一般是 .user.ini 安装时被锁定，没有替换为 Nextcloud 的文件。

执行命令

```bash
 chattr -i /home/wwwroot/example.com/.user.ini
```

将目录替换为你的虚拟主机目录

然后[从 Github 下载安装包](https://github.com/nextcloud/server/releases/)，将 .user.ini 提取出来，替换服务器中的文件。

如果仍提示这个错误，再执行一次这个命令。

## 通过 HTTP 不安全访问站点。

询问服务商解决即可。

## 该实例缺失了一些推荐的 PHP 模块。

其他的 PHP 模块都挺容易装的，这里说说 sodium 这个模块，直接安装的话会提示失败，需要安装前置包 libsodium

```bash
wget -N --no-check-certificate https://download.libsodium.org/libsodium/releases/libsodium-1.0.17.tar.gz
tar xvf libsodium-1.0.17.tar.gz
cd ./libsodium-1.0.17
./configure
make && make check
make install
```

安装完后再安装 sodium，重启服务。

## PHP 模块 "gmp" 和/或 "bcmath" 未被启用。

作为个人网盘不需要这些模块，忽略即可，如果你需要这些模块，自行安装。

## 您的安装没有设置默认的电话区域。

在你的网站根目录下找到 /config/config.php

在倒数第二行插入

```php
'default_phone_region' => 'CN',
```

## 您的网页服务器未正确设置以解析

一共4个

>您的网页服务器未正确设置以解析“/.well-known/webfinger”。
>
>您的网页服务器未正确设置以解析“/.well-known/nodeinfo”。
>
>您的网页服务器未正确设置以解析“/.well-known/caldav”。
>
>您的网页服务器未正确设置以解析“/.well-known/carddav”。

修改 nginx 配置文件

```bash
cd /usr/local/nginx/conf/vhost
nano example.conf
```

添加以下内容

```properties
location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /index.php$request_uri;
    }
```

注意 80 端口和 443 端口的 server 块都要添加，如果配置文件中有相同位置的规则请删除。

## 内存缓存未配置。为了提升性能，请尽量配置内存缓存。

安装 Memcached 并配置，本文不再展开。

## PHP 内存限制低于建议值 512MB

编辑 PHP 配置文件

```bash
cd /usr/local/php/etc
nano ./php.ini
```

找到 memory_limit，将他的值设为 512M

## PHP 的安装似乎不正确，无法访问系统环境变量。getenv(“PATH”) 函数测试返回了一个空值。

编辑 php-fpm.conf

```bash
cd /usr/local/php/etc
nano ./php-fpm.conf
```

在 php-fpm.conf 中加入此行

```properties
env[PATH] = /usr/local/bin:/usr/bin:/bin:/usr/local/php/bin
```

## 建议开启 HSTS

在 nginx 配置文件中插入

```properties
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
```

注意 80 端口和 443 端口的 server 块都要插入

## 总结

总的来说 Nextcloud 对新手确实不友好，这也是为什么国内用户没 Cloudreve 多的原因。

而且在写这篇指南的时候官网文档居然是 404 状态？？？

## 2023 年 1 月 20 日更新

Docker 真香
