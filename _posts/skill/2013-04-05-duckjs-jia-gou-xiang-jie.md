---
layout: post
title : JavaScript模块加载器duckJS架构详解
category : skill
---
{% include JB/setup %}

### 写在前面
很早就想写这篇文章了，一方面是作个总结，另外也可以重新梳理一下思路。

duckJS早在去年就已完成，后来由于工作繁忙，以及github操作还不太熟练，搁置了很长时间。

最近在用node写一个CRUD应用，前端JS文件打算用duck来管理，就想到了这个事，这两天正好放假，索性来码字写好此文。

### duckJS是什么
参考 [Javascript模块加载器—duckJS简介](/skill/2013/02/22/javascript-mo-kuai-jia-zai-qi-duckjs/)

### 整体结构

* module
* global
* markCache
* config
* STATUS
* use
* load
* define

主要功能属性与方法都存于duckJS构造函数，除了define。

	var duckJS = function(){};

提供给用户的是use和define。

module是一个二维map，作用是存储模块状态及模块所属加载队列。 结构如下：

    {
      moduleName1: {// moduleName1是模块全名（即解析之后的）
          queue:	'queuename1',
          status:	1
      },
      ...
    }

queue和status属性到后面讲load的时候再说。

global存放配置信息，主要有三个属性，`charset`，`base`，`alias`，charset是加载JS文件时指定的编码，默认是utf-8， base放置加载模块时的相对路径，默认会设置为duckJS所在的路径。 alias用于存放模块别名。解析模块路径之前会先检查此项设置，如果此模块存在别名，则解析对象将替换为别名值，再进行解析。

markCache是一个二维map，用于在解析模块之后缓存模块的名值对信息，名是模块传入名，值是解析之后的模块全名和模块URL， 解析模块之前会根据传入模块ID先检查此map，如果存在，则代表已解析过，直接返回缓存中的模块信息。

config是提供给用户的配置工具，可以指定模块解析相对路径，模块别名及编码，一般来说，我们只关注配置模块别名， 格式示例：

    D.config({
        alias : {
            'jquery' : 'jquery-1.9.1.min.js',
            'bootstrap' : 'bootstrap.min.js',
            'alertify' : 'alertify.min.js',
            'alertify-core' : '/css/alertify/alertify.core.css',
            'alertify-default' : '/css/alertify/alertify.default.css',
            'common' : 'common.js'
        }
    });


调用此方法后，将会覆盖掉默认的初始值。这样就可以用别名去调用模块。

STATUS代表模块状态，包含一组数字常量

    STATUS : {
        FETCHING : 1, // 开始下载（输出script标签到页面）
        READY : 2, // 已下载到本地，并准备就绪（所有依赖模块已保存）
        COMPILING : 3, // 编译中（正在执行factory）
        COMPILIED : 4 // 编译完成（正确输出了exports）
    }

### 模块调用（use）
一个use即一个加载队列，队列格式

    QueueCache[ queue ] = {
        names : modNames,
        markNames : modNames.slice(), // 拷贝一个副本（JS中对象是按址传递）
        urls : modUrls,
        callback : callback
    };

modNames和urls是数组，分别存放依赖模块解析之后的模块全名和模块URL，markNames是modNames的一个副本，用于在队列加载完成时获取依赖模块的输出，即exports。 至于为什么要特意制造一个副本，客官且接着往下看。

callback是一个函数，将在队列加载完成时触发。

第一次use开始加载队列，之后每次use，将会新增一个队列加到队列缓存QueueCache里。目的是控制队列的加载顺序，只有前一个队列加载完成了， 才会开始加载下一个，直到加载完所有队列。队列加载完成时将会从队列缓存中删除。

###加载队列（load）
加载队列通过load来执行，开始会先检查当前模块状态。

    if( mod.status && mod.status > STATUS.FETCHING ){
        // 检查该模块是否需要特殊回溯
        mod.toDepListBack && duckModule.fireFactoryBack( mod );

        return complete();
    };

complete为当前模块下载完触发onload（IE里是onreadystatechange）后执行，如果模块已经下载，则过滤掉直接执行complete， 你可能已经看到上面例子有一行代码，注释是`检查该模块是否需要特殊回溯`，关于特殊回溯的作用我们留到下面讲模块定义define的时候作解释。

接着讲complete方法，它起着承上启下的作用，为什么这么说呢，先贴上compelte方法内容：

    var complete = function(){
        // 返回继续加载未完成的模块
        if( modNames.length ) return duckJS._load( queue );

        // 所有模块及依赖都已下载完成
        var exports = duckModule.getModExports( markNames );

        // 执行回调
        if( data.callback ){
            data.callback.apply( null, exports );
        };

        // 删除加载完成的队列
        delete QueueCache[ queue ];

        // 检查是否还有未加载的队列
        duckModule.getFirstItem( QueueCache ) && duckJS._load( duckModule.getFirstItem( QueueCache ) );
    };

总共五步，缺一不可。

暂且按着这五步来分析，第一步，这里是一层判断操作，modNames即当前队列里的模块们，modNames里的模块是以shift方式取出来加载的， 这意味着，到所有模块都加载完的时候，它会变成一个空的数组，这样看意思就很清楚了， 如果当前队列还有未加载的模块，递归重新加载，直到所有模块加载完成。

modNames数组会一直伴随当前队列，直到此队列里的模块全部加载完成，最后会随着队列的delete而消亡（即第四步）。

好，现在就可以解释前面的那个疑问，为什么要制造一个modNames的副本了，且再看第二步，此时队列状态是所有模块包括依赖模块 都已加载完，它的作用是获取模块的输出，要取得模块们的输出就必须有一个模块队列，而此时modNames已是一个空队列， 所以在use时才会特意制造一个副本markNames，以备这时候用。

第三步，前面已经取得了模块的输出，现在就可以触发队列回调了。

最后一步，前面我们讲过，只会在第一次use时才会转到load执行加载队列，如果后面还有N个use调用，如何衔接起来呢。 这里这有答案了，在第四步删除当前已加载完的队列之后，再去检查队列缓存里是否还有其他队列存在， 有的话再次递归，转到下一个队列，重复上面的步骤，直到所有队列加载完成。

load过程中会给当前模块加上queue和status属性，即当前模块所属加载队列和状态，并把当前模块name赋给闭包顶级变量onLoadingModName， 至于为什么这么干，老规矩，接着往下看。此时模块状态为FETCHING。

### 模块定义（define）
这里我们要先明白一件事，就是加载一个模块的整个流程，即：调用模块（use），加载队列（load），触发回调。

我们知道，JS是下载完即执行的，define发生在模块下载完成时，可以把它当成是一个包装器，使用define包装一个模块， 当执行它的时候，它去收集模块的依赖和factory，factory即模块工厂，负责输出模块的接口。

define是一个全局方法，挂在window下。它最多接受两个参数，模块依赖列表和模块工厂，唯独没有模块名称， 是的，考虑到大部分人的使用习惯，duckJS所有模块都设计成了匿名，调用模块无需知道模块名称，只需知道模块URL即可，很方便。

虽然说是“匿名”，名字其实还是有的，通过config工具即可给模块URL定义任意一个名字，如果未配置别名，duckJS会根据模块URL 解析出一个唯一的“名字”，事实上通过别名与通过模块URL调用同一个模块，最终生成的这个唯一名是一样的，别名只是调用模块的 快捷通道。当然，这一切都是悄悄地进行，用户感觉不到。

匿名的方式有利有弊，利就是更加方便，弊就是到了这里之后，无法确认当前模块信息，所以才有了上面提到的onLoadingModName， onLoadingModName是一个字符串，指向当前正在执行define的模块名，有了这个name，就可以通过在load里已经缓存好模块信息的module 缓存里定位当前模块了。

	module[ onLoadingModName ]

模块定位到了之后，开始分析模块的依赖信息，如果无依赖或者依赖已加载完成，则直接触发factory。

    // 无依赖或依赖已加载完成，触发factory
    if( !deps || !deps.length ){
        mod.status = STATUS.READY;
        var exports = duckModule.getModExports( mod.deps );

        mod.status = STATUS.COMPILING;
        mod.exports = factory.apply( null, exports );
        mod.status = STATUS.COMPILIED;

        // 回溯
        if( mod.toDepListBack ){
            duckModule.fireFactoryBack( mod );
        }else{
            duckModule.fireFactory( mod );
        };
    };

我们看到，触发factory时，模块状态变了三次，其实status的作用还有一点，前面没有讲，就是定位错误信息，方便调试。 如果没有错误出现，模块状态会安全到达compilied（已编译）,接着往上回溯。

其实回溯这里完全可以单独写一篇。因为三言两语很难讲清楚，回溯涉及模块之间的依赖关系，必须依靠图示，才会让人更容易理解， 接下来会多放一些图示，不只是单纯的代码。

刚刚说的是无依赖或依赖都加载完了的情况，现在开始讲有依赖且未加载的情况。

依赖树如下：

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-01.jpg">

图1

通过use调用A和B模块

	duckJS.use(['A', 'B'])

此时生成一个加载队列，一直到队列加载完成，队列里的模块及其所有层级依赖模块都会共用此队列。

模块factory触发过程如下：

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-02.jpg">

图2

factory触发过程有着严格的顺序，按照上图绿色箭头所指的顺序，从最深的依赖模块开始，一直到队列最后一个模块， 有了这个顺序，就可以保证执行当前模块factory时所需的依赖模块都已经提供了其接口，可以安全使用了。

拿上图来说，A2触发factory时，A2所需要的A1已经提供了其接口，不用关心A1所依赖的 A3和A4，因为A1提供好了接口说明A3和A4也是执行完了的，这个层级关系很重要。

这就像是连在一根引线上的鞭炮，这根引线就是队列，上面有着N个模块，用use点燃，就按照这个顺序挨个引爆， 一个鞭炮引爆了代表它前面的鞭炮全都引爆完了。

回溯的作用就是保证这个顺序。

回溯发生在最后一个依赖模块，即上图中黄色的A4、A2、B2，暂且称它们为回溯点，它的作用就像是螺丝钉，衔接起整个factory触发顺序。

我们从define分析依赖开始，来慢慢了解回溯发生的过程。

首先需要遍历传过来的依赖列表，过滤掉其中已经加载过的

    if( depMod.status > STATUS.FETCHING ){
        deps.splice( i, 1 );
    }

未加载的塞入加载队列

    // 如果最后一个依赖模块已加载，则依次往前推
    if( !lastDepMod ){
        lastDepMod = depMod;
    };

    isLast = depMod.isLast || ( depMod.isLast = [] );
    isLast.unshift( i === len - 1 );

    // 塞入加载队列
    queueNames.unshift( depModName );
    queueUrls.unshift( depModUrl );

lastDepMod指向最后一个模块，isLast用于特殊回溯的情况，下面讲特殊回溯的时候会涉及，暂且不管。 还是拿图1来说，加载队列里会有这样的变化：

    1. [A, B]
    2. [A1, A2, B]
    3. [A3, A4, A2, B]
    4. [A4, A2, B]
    5. [A2, B]
    6. [B]
    7. [B1, B2]
    8. [B2]
    9. []

当队列为空，说明加载队列都完成了，队列的生命周期就结束了，此时就可以触发队列回调了。

处理完这些之后，开始保存模块信息

    // 先查找当前模块的依赖列表
    lastDepMod.toDepList = toDepList || [];

    lastDepMod.toDepList.unshift({
        name : loadingModName,
        factory : factory
    });

    mod.status = STATUS.READY;

为什么要先查找当前模块的依赖列表呢。

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-03.jpg">

图3

这种情况下，A2和A3都是回溯点，A3要把A2的回溯信息继承过来。

toDepList存的是回溯信息，factory指向当前模块。此时模块状态更新为READY，代表模块信息保存完成，hold住，就等着回溯来触发它的factory了。

下面讲讲几种特殊回溯。

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-04.jpg">

图4

此时，lastDepMod指向A1，而非A2，`如果最后一个依赖模块已加载，则依次往前推`，A2已加载过。 第一层依赖模块回溯点从A2往前移至A1，即A1的factory触发完可以安全地去找它的上级A了。

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-05.jpg">

图5

我们看到，A2这时候既是A的依赖模块，同时也是A3的，这时候就要控制使得A2回溯点往上是回到A3而不是A（主持人：iaLast可以上场卖萌了）， 当初设计到这里的时候，迷惑了很久该如何处理这种情况，脑细胞死伤无数啊 (╯_╰) isLast是一个数组，在解析模块依赖时标示该依赖是否是最后一个，是则表示当前为回溯点。妙啊！当时一想出来这方法，乐得洒家直拍大腿<(￣︶￣)>

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-example-06.jpg">

图6

这种情况就比较蛋T了，A2作为A和A3的依赖，同时都是回溯点，现在就需要创建一个特殊的factory队列toDepListBack，有点像isLast

    // 拷贝作为一个特殊回溯列表
    lastDepMod.toDepListBack = lastDepMod.toDepListBack || lastDepMod.toDepList.slice();

    // 当前模块有依赖列表
    if( toDepList ){
        lastDepMod.toDepListBack = toDepList.concat( lastDepMod.toDepListBack );
    };

上面这段代码会在解析A3的依赖的时候执行，同样也需要先检查当前模块的依赖，最后A2的toDepListBack兜里就会有两个东西了， A3和A，即A2第一次作为回溯点的时候，先回到A3，A3触发完同时从回溯队列里删除，第二次的时候回到A。

以上三种特殊回溯可以覆盖所有可能的特殊模块依赖树。

下面是特殊回溯的代码：

    fireFactoryBack : function( depMod ){
        // 当前为特殊回溯模块，且后面还有未触发factory的模块，则停止回溯
        if( !depMod.isLast.shift() ) return;

        var module = duckJS.module,
            toDepListBack = depMod.toDepListBack,
            toDepListBackItem = toDepListBack.shift(),
            STATUS = duckJS.STATUS,
            exports, mod, name, factory, depExports;

        if( toDepListBackItem ){
            name = toDepListBackItem.name;
            factory = toDepListBackItem.factory;
            mod = module[ name ];
            depExports = duckModule.getModExports( mod.deps );

            mod.status = STATUS.COMPILING;
            exports = factory.apply( null, depExports );
            mod.exports = exports;
            mod.status = STATUS.COMPILIED;
        };
    }

duckJS加载流程图：

<img class="lazy" src="/assets/posts/images/grey.gif" data-original="/assets/posts/images/duckJS-process2.jpg">

限于个人知识与经验，其中代码难免有算法及效率上的不足，如果你有更好的实现思路，欢迎留言拍砖，节操奉上。

后话：

当前市面上还有多款优秀的JS模块加载器，除了有名的requireJS，再有支付宝大牛玉伯的seaJS， 带刀的easyJS，司徒正美的mass也集成了模块加载器，等等…

感谢他们在JS模块化之路上所做的努力与贡献，感谢前人大牛的分享精神，致敬。

duckJS从中受益颇多，正基于此，才有了duckJS。分享实可贵，duckJS会延续这一点。

github：[https://github.com/superRaytin/duckJS](https://github.com/superRaytin/duckJS)