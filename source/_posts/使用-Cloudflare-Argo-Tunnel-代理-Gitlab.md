---
title: 使用 Cloudflare Argo Tunnel 代理 Gitlab
date: 2023-01-27 00:53:33
tags:
---

> TLDR: 使用 Nginx 反向代理 Gitlab 即可解决所有问题(双层反代, 下下策)

## 真实需求

绕过备案检测使 URL 不带端口号

### 原理

通过分析 Gitlab-CE 源代码可以找到内置 Nginx 的工作方式

https://gitlab.com/gitlab-org/gitlab-foss/-/blob/master/lib/support/nginx/gitlab-ssl

```nginx
# 前面忽略
upstream gitlab-workhorse {
  # GitLab socket file,
  # for Omnibus this would be: unix:/var/opt/gitlab/gitlab-workhorse/sockets/socket
  server unix:/home/git/gitlab/tmp/sockets/gitlab-workhorse.socket fail_timeout=0;
}

# 中间忽略
server {
    #中间忽略
  location / {
  client_max_body_size 0;
  gzip off;

  ## https://github.com/gitlabhq/gitlabhq/issues/694
  ## Some requests take more than 30 seconds.
  proxy_read_timeout      300;
  proxy_connect_timeout   300;
  proxy_redirect          off;

  proxy_http_version 1.1;

  proxy_set_header    Host                $http_host;
  proxy_set_header    X-Real-IP           $remote_addr;
  proxy_set_header    X-Forwarded-Ssl     on;
  proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
  proxy_set_header    X-Forwarded-Proto   $scheme;
  proxy_set_header    Upgrade             $http_upgrade;
  proxy_set_header    Connection          $connection_upgrade_gitlab_ssl;

  proxy_pass http://gitlab-workhorse;
  }
}
#后面忽略
```

可以发现几乎就是直接把流量交给 Gitlab Workhorse 处理, Nginx 只负责重定向, 提供静态资源, 之间通过 Socket 连接.

Argo Tunnel 可以将本地的服务映射到公网, 也可作为网关转发流量.

因此使用 Argo Tunnel 代替 Nginx 可行.

## 问题

访问带端口的 URL 会被重定向到不带端口的 URL

### 原因

Nginx 按照 gitlab.rb 的配置将用户重定向到了 external_url, 也就是 gitlab.rb 中的设定, 此设定改成带端口的 URL 也会导致问题(过多的重定向导致现代浏览器报错).

## 解决方案

使用 Argo Tunnel 接管内置 Nginx, 直接与 Gitlab Workhorse 连接

### 步骤

Step1. 注册 Cloudflare 账号, 将域名转移到 Cloudflare, 然后打开 Zero Trust -> Access -> Tunnels

Step2. 在 Gitlab 主机上安装 cloudflared 并按照 Cloudflare 的指示配置连接.

Step3. 修改 gitlab.rb

```ruby
external_url 'https://git.targetdomain.com'
# 修改成最终呈现的 URL
gitlab_rails['trusted_proxies'] = ['0.0.0.0/0']
# 允许目标 IP 与 Gitlab Workhorse 建立连接
# Warning: 因为 Argo Tunnel 的远程主机 IP 并不固定, 我也没测出来所有的 IP 段(太多了), 因此这里允许所有主机建立连接.
# 出于安全原因你需要配置本地防火墙禁止外部连接(关闭对应端口, 即 Gitlab Workhorse 监听的端口), Argo Tunnel 不会因此无法建立连接(在本地建立的自然不影响)
gitlab_workhorse['listen_network'] = "tcp"
gitlab_workhorse['listen_addr'] = "0.0.0.0:10080"
# 让 Workhorse 在 10080 监听
# 接应上文, 你需要阻止一切外部主机连接 10080 端口
# 你也可以使用 Unix Socket
# gitlab_workhorse['listen_network'] = "unix"
# gitlab_workhorse['listen_addr'] = "...省略/socket"
# 使用 Socket 不需要配置防火墙
nginx['enable'] = false
# 禁用 Gitlab 自带的 Nginx
```

Step4. 配置 Tunnel

在 Tunnel 管理界面 -> Public Hostname 新建一个主机名

![Hostname](https://s2.loli.net/2023/01/28/bUPzfOeNDQF4nBR.png)

Subdomain 与 Domain 按需填写, 需要与 gitlab.rb 中保持一致

在示例配置中 URL 是 https://git.targetdomain.com, 因此需要设置 Subdomain 为 git, 设置Domain(需要接入 Cloudflare)为 targetdomain.com, Path 留空

Service Type 选择 TCP, URL 填写 localhost:10080, 同理需要与 gitlab.rb 中保持一致, 在示例配置中它是 10080 端口.

Notice: 如果你使用的是 Unix Socket, Service Type 需要选择 UNIX, URL 需要填写**绝对路径**.

Step5. 现在访问你的 Gitlab, 它应该已经可以正常使用了, 并且所有的 URL 与重定向都是正确的.
