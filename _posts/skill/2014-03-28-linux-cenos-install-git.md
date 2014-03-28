---
layout: post
title : Linux(CentOS)下安装git
category : skill
tags : [linux]
---
{% include JB/setup %}

### 前言
上个月把VPS迁到budgetVM，终于不用再受digitalOcean的气了，入手很方便，重点是支持支付宝付款——paypal的界面真是不习惯，开通速度挺快的，1G的内存够我折腾一段时间了~，额外送了俩IP，过段时间再研究下把我那几个二级域名也绑定过来，一直在win7下开发，对linux还不是很熟悉，要学习的东西还挺多的

今天刚把主站部好，发现CentOS默认没有git工具，[Git官网](http://git-scm.com/download/linux) 提示可以通过yum安装

    yum install git

敲下命令，过了几分钟提示

    Setting up Install Process

    No package git available.

    Nothing to do

查了下资料，运行如下命令

    rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm

随后 `yum install git` 后安装成功

Linux CentOS 5.x 32位适合上面的方式，其他系统版本的参考下面：

CentOS 5.x 64-bit(x64):

    rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm

CentOS 6.x32-bit (x86/i386):

    rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm

CentOS 6.x 64-bit(x64):

    rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-5.noarch.rpm
