---
layout: post
title : Javascript在浏览器端生成文件并打包创建zip
category : skill
tags : [Javascript]
---
{% include JB/setup %}



偶然看到一项目，创建文件（文本or图片）然后打包成zip文件提供给用户下载，所有操作都是在浏览器端完成，反复测试，既不是利用HTML5新特性也不需要额外插件，
也没有发出请求，甚至支持到IE8，震惊了有木有， 那是怎么做到的？一探究竟发现还是浏览器那些东西，感叹， 你以为已经很懂浏览器，其实不然;
如果浏览器是一条河，很多时候也只是在河边打湿了鞋就着急上了岸。

项目为叫JSZip，下面是官网的示例，用法简洁，我在其中增加了中文注释：

    var zip = new JSZip();

    // 创建一个文本文件，内容为hello World
    zip.file("Hello.txt", "Hello World\n");

    // 创建一个images的文件夹
    var img = zip.folder("images");

    // 创建一个图片文件
    img.file("smile.gif", imgData, {base64: true});

    // 生成blob大对象
    var content = zip.generate({type:"blob"});

    // 打包保存为example.zip
    saveAs(content, "example.zip");

    /*
    example.zip包含以下文件
    Hello.txt
    images/
        smile.gif
    */

附上项目地址：[https://github.com/Stuk/jszip](https://github.com/Stuk/jszip)