---
layout: post
title : MongoDB数据的导入和导出
category : skill
tags : [MongoDB]
---
{% include JB/setup %}

首先CD到MongoDB的安装目录，接着CD到bin目录，以我本机地址为例

    cd D:\MongoDB
    cd bin

### 导入

    mongoimport -d testDB -c categories backdata/categories.dat

参数说明：

* d - 库名
* c - 表名
* o - 导出位置（相对当前路径）

### 导出

    mongoexport -d testDB -c categories -o categories.dat

参数与导入操作一样

### 为什么不能批量导入导出？

找来找去，MongoDB貌似没有提供批量导出的工具，目前是对指定数据库针对单表导入导出，不过要实现批量操作也并不是很困难，可以用Shell脚本实现