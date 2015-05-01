---
layout: post
title: Unix环境高级编程配置
date:  2014-12-30 09:01:51   
category: Unix
tags: apue
---

1. 下载第三版代码[src.3e.tar.gz](http://www.apuebook.com/src.3e.tar.gz)

2. 解压：tar zxvf src.3e.tar.gz

3. （以下需要root权限）make

4. cp apue.3e/include/apue.h /usr/include
   
   cp apue.3e/lib/error.c /usr/include
   
   cp apue.3e/lib/libapue.a /usr/lib /usr/local/lib

5. 最后经编译的时候加上-lapue:`gcc myls.c -o myls -lapue`

















