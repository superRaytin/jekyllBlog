---
layout: post
title : Rock! Markdown编辑器开发手记
category : skill
tags : [node-webkit]
---
{% include JB/setup %}

## 前言
因为每周都要写周报，用markdown写文档是我的首选，不只是因为它的简洁高效，排版。公司办公电脑大都是win7系统，而windows平台markdown编辑器不多，其中功能比较好的当属markdownpad了，可打开速度每次都要让我菊花一紧，界面更谈不上美观，
没用多久便卸载了，转到在线编辑器，一个不错的就是 `http://mahua.jser.me`，体验还不错，在线编辑器虽然方便，却也有其局限性，如果既能写文档，也能将本地众多的md文档管理起来，写完之后还能直接发邮件，想想多么酷~！
再加上之前接触到 [Node-webkit](https://github.com/rogerwang/node-webkit) ，于是萌生了自己写一个编辑器的想法。

如果你不了解Node-webkit是什么，可以在网上搜一下资料，也可看看 [以web的方式写桌面程序——Node-Webkit](/skill/2013/07/15/node-webkit-desktop-app-develop/)

## 最终的界面
![Rock MarkDown Editor](/assets/posts/images/rock-markdown-1.png)

## 功能分布
编辑器的布局从上到下：

- 菜单栏
- 工具栏
- 标签栏
- 编辑区
- 预览区
- 状态栏

布局上借鉴了markdownpad，菜单栏的实现主要借助了Node-webkit（以下简称nw）的 `Menu` API，用法很简单：

    var gui = require('nw.gui');
    var fileMenu = new gui.Menu();
    fileMenu.append({
        label: '新建文档 (Ctrl+N)',
        click: function(){
            ...
        }
    });

菜单栏包含了整个编辑器所有的功能。

工具栏提供一些常用功能的快捷调用。

状态栏显示当前文档的信息。


## 用到的库
编辑区用的是 `codemirror`，关于 [codeMirror](http://codemirror.net)：

> CodeMirror 是一款“Online Source Editor”，基于Javascript，短小精悍，实时在线代码高亮显示，他不是某个富文本编辑器的附属产品，他是许多大名鼎鼎的在线代码编辑器的基础库。

解释markdown标记用到 [showdown](https://github.com/coreyti/showdown)

邮件发送直接用Node第三方模块 `Node-mailer` 速度非常快。

标签功能