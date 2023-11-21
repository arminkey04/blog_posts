---
title: Nginx程序结构和主配置文件示例
date: 2023-11-20 14:05:17
categories:
- 学习
tags:
- nginx
- 运维
---
## Nginx程序结构
### 日志切割文件目录  
`/etc/logrotate.d/nginx`

### Nginx主程序存放目录
`/etc/nginx`

### Nginx的自配置文件目录
`/etc/nginx/conf.d`

### Nginx默认配置文件
`/etc/nginx/conf.d/default.conf`

### Nginx与PHP交互的内置变量文件
`/etc/nginx/fastcgi_params`

### 字符集文件
`/etc/nginx/koi-utf`  
`/etc/nginx/koi-win`

### 存放响应报文中回传的文件类型文件
`/etc/nginx/mime.types`

### Nginx程序模块目录
`/etc/nginx/modules`

### Nginx主配置文件
`/etc/nginx/nginx.conf`

### 存放uwsgi交互的内置变量文件
`/etc/nginx/uwsgi_params`

### 存放启动Nginx参数文件
`/etc/sysconfig/nginx`

### Nginx二进制文件 调用控制Nginx
`/usr/sbin/nginx`

### Nginx默认网站源码目录
`/usr/share/nginx/html`

### Nginx默认网站首页
`/usr/share/nginx/html/index.html`

### Nginx日志目录
`/var/log/nginx`
## 主配置文件nginx.conf示例
```nginx.conf
#指定Nginx服务的运行用户
user www-data;

#定义Nginx的worker进程数量
worker_processes auto;

#指定Nginx错误日志
error_log /var/log/nginx/error.log warn;

#指定Nginx进程pid号文件
pid /run/nginx.pid;

events {
    #指定Nginx当前worker进程同时可以处理的最大连接数量
	worker_connections 768;
}

http {

    #包含文件 引用文件中的内容
    include /etc/nginx/mime.types;

    #当Nginx无法识别当前访问内容时，触发下载动作
    default_type application/octet-stream;

    #指定Nginx访问日志
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    #定义Nginx访问日志位置
    access_log /var/log/nginx/access.log main;

    sendfile on;
    #tcp_nopush on;

    #当Nginx建立tcp连接之后多长时间无动作自动断开
    keepalive_timeout 65;

    #gzip on;
    
    #包含自配置文件下的所有以.conf结尾的文件
    include /etc/nginx/conf.d/*.conf;
}
```