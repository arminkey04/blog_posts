---
title: 理解nginx主配置文件格式
date: 2023-01-10 22:05:03
categories:
- 学习
tags:
- nginx
- nginx配置
- 运维
- linux
---
## 说的Nginx
Nginx [engine x] 是一个免费开源的高性能网络服务器，可用作反向代理、负载平衡器、SMTP 代理和通用 TCP/UDP 代理服务器。Nginx 最初由 Igor Sysoev 于 2002 年开发，用于解决 C10k 问题。
## 理解nginx主配置文件格式
Nginx的主配置文件默认为`nginx.conf`，位于`/usr/local/nginx/conf`或`/etc/nginx`或/`usr/local/etc/nginx`
## nginx.conf的文件构成
Nginx 配置文件由称为指令(directive)的键值对组成。指令决定要应用的配置。它们可以被组织和分组成块，称为上下文（Context）。上下文是可以相互嵌套的树状结构，指令只能在上下文中使用。
指令示例：
```nginx.conf
user nginx;
worker_processes 5;
```
上下文示例：
```nginx.conf
# Global Context
....
....

# http Context 
http {
      ....
      ....
      # server Context
      server {
             ....
             ....
      }

}
....
```
## 全局上下文(Global Context)
全局上下文(Global Context)位于Nginx核心配置文件的开头，用于全局设置Nginx的配置，不受任何其他上下文的约束。  
全局上下文允许配置用户、工作进程数量、保存主进程PID的文件、工作进程的CPU亲和性。
```nginx.conf
#global context

# global directive
user nobody;
worker_processes 1;
....
....
```
## 事件上下文(Events context)
事件上下文(Events context)包含在全局上下文中，可用于设置Nginx连接处理的全局选项。
```nginx.conf
#global context

# global directive
user nobody;
worker_processes 1;
....
....

#events context
events {
       # events directive
       ....
       ....
}
```
## HTTP上下文(HTTP Context)
HTTP上下文包含用于处理 HTTP 和 HTTPS 流量的指令。服务器上下文可以放在 HTTP 上下文中。
```nginx.conf
#global context

# global directive
user nobody;
worker_processes 1;
....
....

#events context
events {
       # events directive
       ....
       ....
}
# http context
http {
      # http directive
      ....
      ....
}
```
## 服务器上下文(Server Context)
服务器上下文在HTTP上下文中声明。在HTTP上下文中可以有多个服务器上下文实例。每个服务器上下文实例都是处理客户端请求的虚拟服务器。  

服务器上下文中的 listen 和 server_name 指令将用于确定哪个服务器上下文可用于响应请求。该上下文中的指令可以覆盖HTTP上下文中定义的指令。
```nginx.conf
# http context
http {
      # server context
      server {
             # server directive
             listen: 80;
             server_name: example.com, www.example.com
             ....
             ....
      }      
      ....
}
```
## Location上下文(Location Context)
Location上下文用于定义处理客户端请求的指令。当Nginx收到资源请求时，它会尝试将 URI 与其中一个位置相匹配，并进行相应处理。Location上下文可以嵌套在服务器上下文中，也可以嵌套在另一个位置上下文中。
```nginx.conf
# http context
http {
      # server context
      server {
             listen : 80;
             # first location context
             location parameter {
                      ....
                      ....
             }
             # second location context
             location parameter {
                      # nested location context
                      location parameter {
                               ....
                               ....
                      }
                  ....
             }
      }      
      ....
}
```
location 指令有两种类型的参数：前缀字符串（路径名）和正则表达式。正则表达式的前面是区分大小写的`~`，或不区分大小写的`~*`。
```nginx.conf
# 字符串参数指令
location /home/user/ {
         # URI starting with /home/user/ will match 
         # but /some/home/user/ won't 
         ....
}

# 正则表达式参数指令
location ~ /.html? {
         # URI that  has .html or .htm string
         # in it will match
         ....
}
```
## Upstream上下文(Upstream Context)
Upstream上下文用于定义和配置上游服务器。基本上，该上下文定义了一个命名的服务器池，Nginx可以将请求代理到这些服务器。在配置各种类型的代理时，可能会用到该上下文。上游上下文应在http上下文和服务器上下文之外定义，以便使用。

一旦定义了上游服务器，就可以在服务器上下文中使用相同的名称，将请求传递给后端服务器池。
```nginx.conf
http {
    # upstream context
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }
    # server context
    server {
           location / {
                    proxy_pass http://backend;
           }
    }
}
```
要将请求传递给服务器组，需要在 proxy_pass 中指定组（backend）的名称。
## Mail上下文(Mail context)
Nginx还可用于邮件代理。Mail上下文提供了实现邮件代理的能力。邮件上下文在全局上下文中定义，在HTTP上下文之外。
```nginx.conf
# global  context
....
....
# mail context
mail {
     # mail directive
     ....
     ....
}
```
## If上下文(If context)
If上下文提供条件执行，就像其他编程语言中的 if 一样。if 上下文由重写模块提供，是 if 上下文的主要用途。由于 Nginx会使用许多其他有效指令测试请求条件，因此 if 上下文不应用于大多数形式的条件执行，否则可能导致意外执行。
```nginx.conf
http  {
    server {
           # server context
           ....
           ....
           location location_match {
                    # location context
                    if (test_condition) {
                       # if context
                       ....
                    }
           }
    }
}
```
## Limit_except上下文
Limit_except上下文用于限制在位置上下文中使用某些 HTTP 方法。
```nginx.conf
....
# location context
location /restricted-write {
         # location context
         limit_except GET HEAD {
                      # limit_except context
                      deny all;
         }
}
```
在这里，任何客户端使用 GET 和 HEAD 方法提出的请求都将被允许，但客户端使用其他方法提出的请求将被拒绝。