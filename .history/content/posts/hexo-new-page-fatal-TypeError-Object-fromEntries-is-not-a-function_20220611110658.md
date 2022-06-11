---
title: 'hexo new page fatal.(TypeError: Object.fromEntries is not a function)'
date: 2022-04-25 17:12:52
lastmod: 2022-05-25 17:12:52
#tags: ["NodeJs","Hexo"]
#categories: "Web"
draft: false
---
## 生成hexo文章遇到TypeError: Object.fromEntries is not a function错误

### 排查问题

NodeJs版本过低原因，需要 12.x 以上版本才行，本地版本是10.13

### 处理结果

升级NodeJs，升级方法如下（Linux环境）：

1. 查看版本

   ```text
   node –v 
   ```

2. 安装n模块

   ```text
   npm install -g n
   ```

3. 安装最新的稳定版本

   ```text
   n stable
   ```

4. 安装完成后，查看Node的版本，检查升级是否成功

   ```text
   node -v
   ```

最后，重新创建hexo即可。
