---
title: 'PHP实现PDF转图片'
date: 2022-09-08 10:25:19
tags: ["PHP","ImageMagick","Ghostscript"]
categories: ["WEB"]
draft: false
---
## 使用PHP对PDF格式文件的图片转换

### 功能概述

此功能操作需要ImageMagick、Ghostscript、imagick的PHP扩展模块来共同完成，下面先详细介绍下三者的关联：

1. ImageMagick是第三方的图片处理软件，类似GD，[官网](https://www.imagemagick.org)，[中文介绍](https://jelly.jd.com/article/5c34081bd7aa2c0055d09a71)。
2. imagick是php的一个扩展模块，它调用ImageMagick提供的API来进行图片的操作，[官网](https://www.php.net/manual/zh/book.imagick.php)。
3. Ghostscript是一套建基于Adobe、PostScript及可移植文档格式（PDF）的页面描述语言等而编译成的免费软件，[官网](https://www.ghostscript.com)。Ghostscript最初是以商业软件形式在PC市场上发售，并称之为“GoScript”。但由于速度太慢（半小时一版A4），销量极差。后来有心人买下了版权，并改在Linux上开发，成为了今日的Ghostscript。已经从Linux版本移植到其他操作系统，如其他Unix、Mac OS X、VMS、Windows、OS/2和Mac OS classic。
4. 三者关系，ImageMagick无法直接实现pdf文档到图片的转换，需要借助于gostscript软件包，然后由ImageMagick处理图片，最后如果是选择PHP开发就要使用imagick对接ImageMagick。

### 软件版本说明

1. PHP 7.2.19
2. ImageMagick 7.1.0-47
3. Ghostscript 9.56.1
4. imagick 3.7.0

### 安装ImageMagick

1. 下载ImageMagick

    ```bash
    wget https://imagemagick.org/archive/ImageMagick.tar.gz
    ```

2. 解压安装

    ```bash
    tar zxvf ImageMagick.tar.gz
    cd ImageMagick-7.1.0-47/
    ./configure --prefix=/usr/local/imagemagick
    make && make install
    ```

### 安装PHP的imagick拓展

1. 下载imagick

    ```bash
    wget https://pecl.php.net/get/imagick-3.7.0.tgz
    ```

2. 解压安装

    ```bash
    tar zxvf imagick-3.7.0.tgz
    cd imagick-3.7.0/
    /usr/local/php/bin/phpize   #用phpize生成
    ln -s /usr/local/imagemagick/include/ImageMagick-7 /usr/local/imagemagick/include/ImageMagick #ImageMagick6.8以上版本为/usr/local/include/ImageMagick-X,在configure之前先做下软连接
    ./configure --with-php-config=/usr/local/php/bin/php-config --with-imagick=/usr/local/imagemagick  #编译
    make && make install  #安装
    ```

3. 查看版本

    ```bash
    php --ri imagick
    imagick module => enabled
    imagick module version => 3.7.0    
    ```

4. PHP imagick扩展安装可能会遇到的问题解决：configure通过,在make时出现错误error: wand/MagickWand.h: No such file or directory，解决方法：

    ```bash
    yum install gtk2-devel #https://pkgs.org/download/gtk2-devel
    export PKG_CONFIG_PATH=/usr/local/imagemagick/lib/pkgconfig/   
    ```

### 安装Ghostscript

1. 下载Ghostscript

    ```bash
    wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs9561/ghostscript-9.56.1.tar.gz
    ```

2. 安装Ghostscript

    ```bash
    tar -xzf ghostscript-9.56.1.tar.gz
    cd ghostscript-9.56.1
    ./configure
    make && make install
    ```

3. 测试gs

   ```bash
   gs -dQUIET -dNOSAFER -dBATCH -sDEVICE=pngalpha -dNOPAUSE -dNOPROMPT -sOutputFile=/home/wwwroot/demo/a%d.png test.pdf # sOutputFile=图片生成路径 PDF文件路径
   ```

### PHP实现PDF转图片实例

1. 代码块

    ```php
    $pdf = "/home/demo/test.pdf"; // PDF文件路径
    $path = "/home/demo/imgs/"; // 图片生成路径
    if(!extension_loaded('imagick')) {
        return false;
    }
    if (!file_exists($pdf)) {
        return false;
    }
    $im = new \Imagick();
    $im->setResolution(120, 120); // 设置分辨率，值越大分辨率越高
    $im->setCompressionQuality(100);  // 压缩比1-100，100压缩比最低
    $im->readImage($pdf);
    $return = [];
    foreach ($im as $k => $v) { // 循环输出图片
        $v->setImageFormat('png');
        $fileName = $path . md5($k . time()) . '.png';
        if ($v->writeImage($fileName) == true) {
            $return[] = $fileName;
        }
    }
   ```
