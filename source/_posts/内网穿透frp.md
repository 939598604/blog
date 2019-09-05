---
title: frp服务端的安装
date: 2019-08-09 20:32:59
password: 123
abstract: 欢迎来到test, 请输入密码.
message: 欢迎来到test, 请输入密码.
toc: false
mathjax: false
tags:  [工具] 
category: 工具
---

#内网穿透frp

##  一、frp服务端的安装

###  1.内网穿透环境的准备

- 必须准备一台公网的服务器

- 公网服务器有域名指定
- 同时配置在域名中添加一台A记录   * 指向该公网的服务器  原因是为了更好的匹配二级域名

###  2.下载解压frp

在地址：http://diannaobos.iok.la:81/frp/frp-v0.20.0/ 中找到`frp_0.20.0_linux_386.tar.gz `下载到服务器

```shell
tar -zxvf frp_0.20.0_linux_386.tar.gz
```

###  3.编辑frp的服务端配置文件

vim frps.ini

```shell
[common]
bind_port = 7001
vhost_http_port = 80 
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
subdomains_host = gzstrong.cn
```

###  4.启动frp的服务端

```shell
nohup /soft/frp/frps -c /soft/frp/frps.ini>/dev/null 2>&1 &
```



##  二、frp客户端的安装

###  1.客户内网穿透环境的准备

只需在公司内网随便一台机器就可以

###  2.下载解压frp

在地址：http://diannaobos.iok.la:81/frp/frp-v0.20.0/ 中找到`frp_0.20.0_linux_386.tar.gz `下载到服务器

```shell
tar -zxvf frp_0.20.0_linux_386.tar.gz
```

###  3.编辑frp的服务端配置文件

vim frpc.ini

```shell
[common]
server_addr = 47.106.228.4
server_port = 7001

[ssh]
#29的ssh端口映射
type = tcp
local_ip = 192.168.197.29
local_port = 22
remote_port = 61000

[www]
#思创官网www.gzstrong.cn指向 www.gzstrong.com
type = http
local_ip = 112.124.183.21
local_port = 80
custom_domains = www.gzstrong.cn

[myremote]
#远程主机的端口映射
type = tcp
local_ip = 192.168.197.223
local_port = 3389
remote_port = 61001

[rep]
type = http
local_ip = 192.168.197.67
local_port = 9090
custom_domains = rep.gzstrong.cn

[tomcat]
type = http
local_ip = 192.168.197.29
local_port = 8081 
custom_domains = tomcat.gzstrong.cn

[mysql]
type = tcp
local_ip = 192.168.197.205
local_port = 3306
remote_port = 61003
```

###  4.启动frp的客户端

```shell
nohup /soft/frp/frpc -c /soft/frp/frpc.ini>/dev/null 2>&1 &
```



##  三、访问frp

- web端的服务

  因frp服务端的配置文件frps.ini中配置了

```
vhost_http_port = 80 
```

所以客户端的配置

```
[rep]
type = http
local_ip = 192.168.197.67
local_port = 9090
custom_domains = rep.gzstrong.cn
```

这个配置中的192.168.197.67:9090可以映射成rep.gzstrong.cn

即浏览器中直接输入http://rep.gzstrong.cn就可以访问到内网的机器192.168.197.67:9090

- windows远程服务

  ```
  [myremote]
  #远程主机的端口映射
  type = tcp
  local_ip = 192.168.197.223
  local_port = 3389
  remote_port = 61001
  ```

  打开windows的远程mstsc

  输入`服务的ip地址:61001`即可访问到内网服务器的`192.168.197.223:3389`端口

