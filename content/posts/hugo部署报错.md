---
title: 'hugo部署报错'
date: '2022-06-13'
tags: ["hugo","DoIt"]
categories: ["Web"]
draft: false
---
## 使用DoIt主题进行hugo server时报错，问题处理流程如下

### 错误信息描述

```bash
$ hugo server
Error: add site dependencies: load resources: loading templates: "/themes/DoIt/layouts/partials/meta/author.html:8:1": parse failed: template: partials/meta/author.html:8: unclosed action
```

### 安装最新版本环境变量

1. 查看当前版本

    ```bash
    $ hugo version
    Hugo Static Site Generator v0.68.3/extended linux/amd64 BuildDate: 2020-03-25T06:15:45Z
    ```

2. 查询并下载最新[发行版本](https://github.com/gohugoio/hugo/releases)，本文使用v0.100.2版本

3. 以linux ubuntu为例，查看hugo环境变量路径，进行替换即可

    ```bash
    $ whereis hugo
    hugo: /usr/bin/hugo /usr/share/man/man1/hugo.1.gz

    $ mv /download/hugo /usr/bin
    $ chmod 755 /usr/bin/hugo
    $ hugo version
    hugo v0.100.2-d25cb2943fd94ecf781412aeff9682d5dc62e284 linux/amd64 BuildDate=2022-06-08T10:25:57Z VendorInfo=gohugoio
    ```

4. 重新hugo server正常

```bash
Start building sites …
hugo v0.100.2-d25cb2943fd94ecf781412aeff9682d5dc62e284 linux/amd64 BuildDate=2022-06-08T10:25:57Z VendorInfo=gohugoio

                   | ZH-CN
-------------------+--------
  Pages            |    32
  Paginator pages  |     0
  Non-page files   |     0
  Static files     |    83
  Processed images |     0
  Aliases          |    12
  Sitemaps         |     1
  Cleaned          |     0

Total in 6615 ms
```
