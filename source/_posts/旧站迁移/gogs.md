---
title: '【折腾】一个小时用树莓派搭建私有Git站'
date: 2019-08-09 22:23:26
tags: [gogs,nginx,树莓派,旧文归档]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kaf-train.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
将吃灰的树莓派改造成一个属于自己的 Git 站点。

<!-- more -->

## 前言

之前一时冲动买的树莓派一直放在宿舍吃灰，在我入手 Mi Air 轻薄本之后他彻底地失去了作为 Linux 二号机的作用。今天看到 GitHub 疑似封锁伊朗用户的消息，突发灵感，将树莓派搞成了一个个人 Git 站。

## 前置

- 吃灰树莓派 一块 （本文为 Model 3B / Raspbian 10 buster）
- 用来内网穿透的VPS 一台（本文为 Ubuntu 18.04 LTS ）
- 内网穿透工具 [frp](https://github.com/fatedier/frp) 
  - 树莓派请下载 linux_arm 版本包，VPS 则具体情况具体分析，本例中为 linux_amd64
- 轻量 Git 部署包 [gogs](https://github.com/gogs/gogs)
  - 树莓派请下载 raspi2_armv6 版本包

## 局域网搭建 gogs

1. 先将 gogs 的二进制安装包放到一个安全的位置，如 `/var/gogs/` 解压放好。
2. 创建 `/lib/systemd/system/gogs.service` 以后台运行 gogs：

```shell
[Unit] 
Description=Gogs 
After=syslog.target 
After=network.target 

[Service] 
Type=simple 
User=pi 
Group=pi 
WorkingDirectory=/var/gogs/          # 路径
ExecStart=/var/gogs/gogs web         # gogs路径
Restart=always 
Environment=USER=pi HOME=/home/pi 

[Install] 
WantedBy=multi-user.target 
```

3. `sudo systemctl enable gogs && sudo systemctl start gogs`来启动服务。
4. `sudo systemctl status gogs`查看运行状态，若为 active 则进行下一步。
5. 局域网访问 `树莓派局域网ip:3000`来进入 gogs 的首次运行设置。
6. 设置完成后，可以编辑 `/var/gogs/custom/app.ini`来调整站点设置。
7. 至此，本地 gogs 服务已经搭设完毕。

## 内网穿透 - 服务器设置

1. 将对应版本的 frp 放到安全的位置，如 `/var/frp/`。
2. 修改 server 端的配置文件`/var/frp/frps.ini`：

```shell
[common]
bind_port = 7000               # 穿透用端口
vhost_http_port = 80           # http 访问端口
vhost_https_port = 443         # https 访问端口
dashboard_port = 8888          # frps 控制面板访问端口
dashboard_user = USERNAME      # 控制面板用户名/密码
dashboard_pwd = PASSWORD
auth_token = TOKEN             # 穿透用 token
```

3. 创建服务 `/lib/systemd/system/frps.service`：

```shell
[Unit]
Description=frps service
After=network.target network-online.target syslog.target
Wants=network.target network-online.target

[Service]
Type=simple
ExecStart=/var/frp/frps -c /var/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

4. `sudo systemctl enable frps && sudo systemctl start frps`来启动服务。
5. `sudo systemctl status frps`，若服务启动完成，则可以通过 服务器ip:控制面板端口查看状态。

## 内网穿透 - 树莓派设置

1. 将对应版本的 frp 放到安全的位置，如 `/var/frp/`。
2. 修改 server 端的配置文件`/var/frp/frpc.ini`：

```shell
[common]
server_addr = 服务器IP
server_port = 7000
login_fail_exit = false
auth_token = TOKEN

# 树莓派SSH
[sshpi]
type = tcp
local_port = 22
remote_port = 2222 
local_ip = 127.0.0.1 

# gogs HTTP
[httpgit]
type = http
local_port = 80
custom_domains = YOUR.DOMAIN.COM

# gogs HTTPS
[httpsgit]
type = https
local_port = 443
custom_domains = YOUR.DOMAIN.COM

# gogs ssh # 需打开 gogs 对应功能
[sshgit]
type = tcp
local_port = 对应gogs设置端口
local_ip = 127.0.0.1
remote_port = 50022
```

3. 创建服务 `/lib/systemd/system/frpc.service`：

```shell
[Unit]
Description=frpc service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/var/frp/frpc -c /var/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

4. `sudo systemctl enable frpc && sudo systemctl start frpc`来启动服务。
5. 若一切正常，现在已经可以通过设置的域名访问 gogs 服务了。

## 配置 SSL

没有小绿锁的站点是没有灵魂的。

本过程需要一个对应域名的 SSL 证书（pem 和 key），树莓派需要 nginx 环境。

1. 将 SSL 证书放到安全的位置，如`/var/nginx/ssl/domain.pem`和`./domain.key`
2. 编辑 nginx 默认站点配置，一般在`/etc/nginx/sites-enabled/default`或者`/var/nginx/`相应位置：
3. 添加新 server 项目：

```nginx
server {
    listen 80; 
    listen 443;
    server_name YOUR.DOMAIN.COM;

    ssl on;
    ssl_certificate /var/nginx/ssl/domain.pem;
    ssl_certificate_key /var/nginx/ssl/domain.key;
    
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    if ($ssl_protocol = "") { return 301 https://$host$request_uri; }

    location /{
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
    }
}
```

4. 保存，通过`sudo nginx -s reload`刷新配置。

大功告成！Enjoy it。

## 示例

我树莓派上部署的 gogs 站点：[https://git.dctewi.com](https://git.dctewi.com)

## 截图

![](https://s2.ax1x.com/2019/08/09/eqsa24.png)

![](https://s2.ax1x.com/2019/08/09/eqsdxJ.png)