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