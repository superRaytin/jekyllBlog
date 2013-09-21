---
layout: post
title : git项目的本地化过程（webstorm）
category : skill
---
{% include JB/setup %}

用github来管理多人协同开发的项目是一个不错的选择，它也是一个很棒的版本管理工具。

### 正常过程
使用起来也比较简单：

a·  首先需要先安装git的本地版本管理工具tortoiseGit，下载地址：http://code.google.com/p/tortoisegit/

b·  在github上新建一个仓库

c·  然后，在本地新建一个文件夹，右键 git init here，完成初始化，这时右键菜单的tortoiseGit里会出现 设置 等信息，点击设置

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/git-webstorm-1.gif">

将建好的仓库地址进行配置

d·  右键 git同步

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/git-webstorm-2.gif">

点击拉取

### 在webstorm下本地化
上面说的是一般的同步方式， 这里主要讲在webstorm中如何将一个git项目本地化

步骤：

a· 在github.com上创建一个新仓库，按照提示新建README.md，.gitignore, LICENSE等文件

b· 打开webstorm，VCS – checkout from version control – git – 输入仓库地址，之后本地将会生成一个与仓库同名文件夹

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/git-webstorm-3.gif">

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/git-webstorm-3.jpg">

c· 此时可以测试git的提交与更新，随意更新一个文件，点击工具栏上的commit changes，提交更新，成功

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/git-webstorm-4.gif">