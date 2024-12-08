---
title: 用 Certbot 简单注册一个 SSL 证书
date: 2024-12-07 03:00:00 +08:00
modified: 2024-12-07 03:00:00 +08:00
tags: [Certbot, https, SSL证书]
description: "最近在开发新网站，牵扯到很多与 SSL 证书有关的问题。比如说，作为前后端分离的网页架构，如果前端页面上了 https，那么后端接口也必须是 https，否则接口用不了。而商业的 SSL 证书价格死贵，最低的一年也要大几千，对于测试环境来说实在是觉得没必要花这个冤枉钱。于是我大概调研了一下免费获取 SSL 证书的方法，发现了 Certbot 这个工具。"
comments: true
---
最近在开发新网站，牵扯到很多与 SSL 证书有关的问题。比如说，作为前后端分离的网页架构，如果前端页面上了 https，那么后端接口也必须是 https，否则接口用不了。而商业的 SSL 证书价格死贵，最低的一年也要大几千，对于测试环境来说实在是觉得没必要花这个冤枉钱。于是我大概调研了一下免费获取 SSL 证书的方法，发现了 Certbot 这个工具。

网上搜索到的教程里，大部分都很类似，但是过程却略显繁琐。但是根据我的实践经验，用 Certbot 注册 SSL 证书极其的简单，应该说比大部分教程里的步骤更简单。

Certbot 的安装很简单，建议执行下面的命令把 certbot 和 nginx 插件一起装了：

```bash
$ sudo yum install certbot python2-certbot-nginx  # CentOS 7
```

其他平台请自行查找安装命令。

注意，到这一步之后，很多文章就开始教你执行 certbot 命令了，而且大部分都是类似于这种：

```bash
$ sudo certbot certonly ...
```

这个命令中 `certonly` 是指的只生成证书，不安装到 Nginx 里。如果你需要在生成以后增加到 Nginx 配置里，那么就不应该加这个 `certonly`，否则你还需要手动去配置 Nginx 就显得很傻。

假如我们的 Nginx 服务器有这样一个反向代理的配置：

```nginx
server {
    listen 80;

    server_name foo.bar.com;  # 域名
    client_max_body_size 50M;
    location / {
        proxy_pass http://localhost:8004;  # 反向代理的地址
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
当然，这个反向代理的域名（这里的 foo.bar.com 应替换成你自己的域名）应该是已经完成了解析，可以通过 http 访问到。

这个时候我们只需要去执行
```bash
$ sudo certbot --nginx
```
它会输出一些提示，让你输入邮箱（首次使用）同意条款之类的
```
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): foo@bar.com
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
```

之后，会给你列一个表，这个表是你当前 Nginx 里所有已经配置好的反向代理的域名，你只需要填一个编号选择其中一个就行了。

```
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: aaa.com
2: bbb.team
3: ccc.com
4: foo.bar.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 4
```

例如我们想要给 foo.bar.com 注册 ssl 证书，那么我们只需要输入 4 回车就行了。然后我们就什么都不需要管了，certbot 会给我们把证书生成好，然后自动配置到 Nginx 里。

当然，如果你想要手动配置，那么你可以执行 `sudo certbot certonly --nginx`，然后手动配置 Nginx 就行（就像大部分文章教的那样），就是流程会变得繁琐。

不管是自动还是手动配置，得到的 ssl 证书都是可以直接使用的，你可以把它上传到 OSS 的 CDN 或者静态页面的证书托管服务里，实现自己的静态资源也支持 https 访问。

使用上述方法注册证书，有效期应该是只有3个月。你可以直接用 crontab 配置定时更新证书的任务。

```bash
0 2 * * * /usr/bin/certbot renew --quiet  # 每天2点更新证书
```

之后就什么也不用管了。
