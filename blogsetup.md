---
title: 搭建此Hexo博客之流程
date: 2022-10-06 15:58:16
---
## VPS信息  
- cpu: 1核  
- 内存: 768M
- 系统: Ubuntu 20.04
- 但是他真的很便宜  
## 环境/依赖/组件
- Nginx 
- Node.js 
- Git
- Hexo
- hexo-theme-reimu https://github.com/D-Sketon/hexo-theme-reimu

## 安装准备    
1. 安装Node.js  
    ```bash
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl gnupg
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
    ```
    ```bash
    NODE_MAJOR=20
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
    ```
    ```bash
    sudo apt-get update
    sudo apt-get install nodejs -y
    ```
2. 安装Git  
    ```bash
    sudo apt-get install git-core
    ```  
3. 安装Nginx  
    ```bash
    sudo apt install nginx
    ```
4. 安装Hexo
    ```bash
    npm install -g hexo-cli
    ```
5. 开始建站
## 搭建流程
1. 配置工作目录  
    ```bash
    cd blog
    hexo init Blog_cn
    cd Blog_cn
    ```
2. 试运行  
    ```bash
    hexo g
    hexo s
    ```
    hexo g 用来生成网页文件  
    hexo s 用来启动测试服务器

3. 配置主题  
    安装灵梦主题
    ```bash
    cd /root/blog/Blog_cn/theme
    git clone https://github.com/D-Sketon/hexo-theme-reimu.git
    ```
## 配置服务器
1. 修改_config.yml文件  
    ```bash
    deploy:
        type: git
        repo: <用户名>@<公网IP>:/var/repo/hexo_static
        branch: master
    ```
2. 创建私有Git仓库  
    创建/var/repo目录
    ```bash
    sudo mkdir -p /var/repo/
    ```
3. 修改目录的所有权和用户权限  
    ```bash
    sudo chown -R $USER:$USER /var/repo/
    sudo chmod -R 755 /var/repo/
    ```
4. 初始化私有Git仓库
    ```bash
    cd /var/repo/
    git init --bare hexo_static.git 
    ```
5. 配置Nginx托管文件目录  
    创建 /var/www/hexo 目录，用于 Nginx 托管。
    ```bash
    sudo mkdir -p /var/www/hexo
    sudo chown -R $USER:$USER /var/www/hexo
    sudo chmod -R 755 /var/www/hexo
    ```
6. 修改Nginx的default文件
    ```bash
    sudo vim /etc/nginx/sites-available/default
    ```
    ```bash
    server {
        [...]
        #listen 80 default_server;
        #listen [::]:80 default_server;
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        ssl_certificate "xxxxxx.pem";
        ssl_certificate_key "xxxxxx.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 10m;
        ssl_ciphers EECDN+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EECDH+EDH;
        ssl_prefer_server_ciphers on;
        [...]
    }
    ```
7. 重启Nginx服务
    ```bash
    sudo service nginx restart
    ```
8. 创建Git钩子  
    在自动生成的hooks目录下创建一个新的钩子文件：
    ```bash
    vim /var/repo/hexo_static.git/hooks/post-receive
    ```
    添加指定的Git的工作树和Git目录
    ```bash
    #!/bin/bash
    git --work-tree=/var/www/hexo --git-dir=/var/repo/hexo_static.git checkout -f
    ```
    保存文件并退出，赋予文件可执行权限
    ```bash
    chmod +x /var/repo/hexo_static.git/hooks/post-receive
    ```
## 后续步骤
- 更新操作  
更新博文可在工作目录`Blog_cn`下执行
    ```bash
    hexo g
    hexo d
    ```
- 如更新后显示效果有误甚至无法更新可以尝试执行`hexo clean`后再执行更新操作  
## 最后效果
![效果图](/image/setupfinal.png)


