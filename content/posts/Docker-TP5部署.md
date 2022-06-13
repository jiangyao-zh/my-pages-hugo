---
title: 'Docker+TP5部署'
date: 2022-05-26 10:25:19
tags: ["Docker","Compose","PHP","Thinkphp"]
categories: ["运维"]
draft: false
---
## 使用Compose对TP5进行部署

### 软件版本说明

1. Nginx 1.16
2. PHP 7.2
3. CentOS 7.9（已安装）
4. Docker [20.10.14](https://docs.docker.com/engine/release-notes/)
5. docker-compose [1.29.1](https://docs.docker.com/compose/release-notes/)（文件格式版本3.0，建议安装最新版本）

{{< admonition type=note title="注意" open=true >}}
数据相对独立，本文不涉及数据库安装，建议使用mysql
{{< /admonition >}}

### 安装Docker

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

[详细教程](https://www.runoob.com/docker/centos-docker-install.html)

### 安装docker-compose

1. 下载最新版的docker-compose文件

    ```bash
    sudo curl -L https://github.com/docker/compose/releases/download/1.29.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    ```

2. 添加可执行权限

    ```bash
    sudo chmod +x /usr/local/bin/docker-compose
    ```

3. 查看版本

    ```bash
    docker-compose -v
    docker-compose version 1.29.1, build 8dd22a9
    ```

### 安装PHP

1. 创建php7.2的docker目录(每个项目支持PHP较为不同，所以版本号命名方便区分)

    ```bash
    mkdir /docker/php7.2
    ```

2. 在php7.2目录下新建docker-compose.yml文件

    ```bash
        version: "3"

        services:
        fpm:
            build:
                context: ./ #Dockerfile路径
                dockerfile: php7.2_fpm_dockerfile #服务除了可以基于指定的镜像，还可以基于一份Dockerfile，我把Dockerfile放到下面
            restart: always
            networks:
               - db
            environment:
               - TZ=Asia/Shanghai
            privileged: true
            volumes:
               - /home/code/tp5:/var/www/tp5 #挂在程序目录，建议此目录与nginx挂在目录保持一致
        networks:
        db:
            external: true 
    ```

    ```bash
    FROM php:7.2.24-fpm
    RUN docker-php-ext-install pdo pdo_mysql #增加mysql扩展
    ```

3. 启动服务

   ```bash
   docker-compose up -d 
   ```

   在/home/code/tp5目录下创建index.php进行测试

### 安装Nginx

1. 创建nginx的docker目录

    ```bash
    mkdir /docker/nginx
    ```

2. 在nginx目录下新建docker-compose.yml文件

    ```bash
    version: "3"
    services:
    nginx:
    restart: always
    container_name: nginx
    image: nginx
    networks:
       - db
    ports:
       - 80:80
    volumes:
       - ./conf/conf.d:/etc/nginx/conf.d
       - ./etc/nginx/nginx.conf:/etc/nginx/nginx.conf
       - ./log:/var/log/nginx
       - ./wwww:/var/www
       - /home/code/tp5:/var/www/tp5 #挂在程序目录，建议此目录与php挂在目录保持一致
    networks:
    db:
        external: true    
    ```

3. 配置TP5的conf，在/docker/nginx/conf/conf.d下创建tp.conf文件

   ```bash
    server {
        listen 80;
        listen [::]:80;
        server_name 127.0.0.1;
            root /var/www/tp5;
            index index.html index.htm index.php default.html default.htm default.php;

        location / {
                index index.html index.php;
                if (!-e $request_filename) {
                        rewrite  ^(.*)$  /index.php?s=$1  last;
                        break;
                }
        }

        location ~ .*\.(php|php5)?$
        {
            fastcgi_pass    php72_fpm_1:9000; #与docker ps命令列表中的NAMES保持一直
            fastcgi_index   index.php;
            fastcgi_param   SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param   SCRIPT_NAME      $fastcgi_script_name;
            include         fastcgi_params;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }

        location ~ /.well-known {
            allow all;
        }

        location ~ /\.
        {
            deny all;
        }

        access_log  /var/log/nginx/tp5.access.log;
        error_log  /var/log/nginx/tp5.error.log; 
   ```

4. 启动服务

   ```bash
   docker-compose up -d 
   ```

   测试//127.0.0.1/index.php

### 目录结构

   ```bash
    ├── docker
    │   ├── nginx
    │   │   ├── conf
    │   │   │   └── conf.d
    │   │   │       └── tp5.conf
    │   │   ├── docker-compose.yml
    │   │   ├── etc
    │   │   │   └── nginx
    │   │   │       └── nginx.conf
    │   │   ├── html
    │   │   ├── log
    │   │   │   ├── tp5.access.log
    │   │   │   └── tp5.error.log
    │   │   ├── timezone
    │   │   └── wwww
    │   ├── php72
    │   │   ├── docker-compose.yml
    │   │   └── php7.2_fpm_dockerfile
   ```
