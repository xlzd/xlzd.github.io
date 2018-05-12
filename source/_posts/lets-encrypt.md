---
title: "通过 Lets's Encrypt 启用 HTTPS"
date: 2018-05-12 17:49:35
tags:
  - Linux
  - HTTPS
---

 
记录下通过 Lets's Encrypt 开启 HTTPS 加成的步骤。这里以 Ubuntu16.04 + nginx 为例。我原本服务器 nginx 就已经配置好对应的域名，这里略过。

首先，安装 certbot ：
```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

开放 443 端口，先检查下防火墙：
```bash
sudo ufw status
```

如果防火墙没有打开，是这样的：
```bash
xlzd$ sudo ufw status
Status: inactive
```

那就什么也不管就好。如果开了，大概输出是这样：
```bash
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
```

如果防火墙处于打开状态，执行一下：
```
sudo ufw allow 'Nginx Full'
```

然后就可以生成证书了：
```
sudo certbot --nginx -d fuckthe.app -d www.fuckthe.app
```

第一次执行的时候，会提示你输入一个邮箱地址。

等执行成功之后，会看到大概这样的输出：
```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
```

选 2 即可，会被自动写入到 nginx 配置中去。

在 crontab 中配置一下自动更新证书：
```bash
30 2 * * 1 /usr/bin/certbot renew --dry-run >> /var/log/le-renew.log
35 2 * * 1 /usr/bin/nginx -s reload
```

下次再要加新域名的时候，只需继续执行 `sudo certbot --nginx -d domain.com` 就可以啦~

Done.