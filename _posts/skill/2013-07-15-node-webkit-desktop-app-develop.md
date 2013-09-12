---
layout: post
title : 以web的方式写桌面程序——Node-Webkit
category : skill
tags : [node-webkit]
---
{% include JB/setup %}

Node的火热程度即使我不说，想来你也听说过很多次了，这里必须要说说Node平台迅速崛起的一匹黑马 —— Node-webkit，其实也不能说是归属Node平台，因为它并不是作为Node的模块存在——虽然作者也曾尝试这么干过， 像我一样许许多多曾被它名字的误导的朋友可以醒醒了，
如果你还不清楚这是个什么东东，下面引述一段网上的介绍：

> node-webkit是一个支持跨操作系统（Windows，Linux，MacOS）的利用流行的Web技术（Node.JS, JavaScript，HTML5）来编写应用程序的平台。应用程序开发人员可以轻松的利用Web技术来实现各种应用程序。node-webkit性能和特色已经让它成为当今世界领先的web技术应用程序平台。

官方的解释是`Web应用程序运行时环境`，简单来说就是，你可以利用你所知道的几乎所有web技术来构建本地应用程序，HTML5, JS, Nodejs, jQuery等等。

如果你正好是一名`Web开发人员`，第一次听到这样的介绍——那么你一定会为此血液沸腾的（冷血动物不算），是的，第一次接触就把我的技术三观给颠覆了，这就像是哥伦布发现新大陆时的心情——除了激动还是激动，作为一名Jser，新大陆就是从未接触过的桌面程序开发领域，
载着我向新大陆前进的则是Node-webkit！第一次感觉Javascript如此强大，就像 $美元，通向世界。

能想到将Node和webkit这两个一般人认为无交集的项目合在一起，不得不佩服作者的创意，使用Node-webkit开发本地程序，除了可以尽情地使用第三方Node模块，作者还封装了一层，调用系统级别的API，剪贴板，程序窗口，系统托盘，文件对话框及Shell，现在来看，与本地UI交互的
API数量还有些少，可能也有安全层面的考虑吧，但从现在项目的更新频率来看，相信以后会越来越强大的。

Node-webkit的github主页：[https://github.com/rogerwang/node-webkit](https://github.com/rogerwang/node-webkit)