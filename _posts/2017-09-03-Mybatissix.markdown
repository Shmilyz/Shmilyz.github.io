---
layout:     post
title:      "Mybatis源码分析第六讲之Mybatis与数据库交互核心前铺垫(二)"
subtitle:   ""
date:       2017-09-03 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第六讲之Mybatis与数据库交互核心前铺垫(二)


在上一讲我们用大量的字符去为我们这一讲进行铺垫，我们这一讲将继续接近我们的mybatis核心，而在进行核心交互之前，mybatis依然要进行大量的准备工作，我们来看看mybatis都进行了怎样的准备工作。



![java-javascript](/img/in-post/six-mybatis/1.png)
.我们上一讲的最后一张图拿到这里，我们调用DefaultSqlSession的selectList方法，我们现在进入这个方法。
![java-javascript](/img/in-post/six-mybatis/2.png)
![java-javascript](/img/in-post/six-mybatis/3.png)
我们欣喜的发现，我们终于看到了一个我们熟悉的对象了，就是我们的MappedStatement，我们通过传入的configuration并调用其getMappedStatement()获取到我们的value，也就是我们的MappedStatement，而我们也可以从第三张图再熟悉一遍我们的MappedStatement中都有什么属性。
![java-javascript](/img/in-post/six-mybatis/4.png)
很幸运，我们又看到了一个前面我们讲过的对象executor，我之前说过，executor的作用就是执行增删改查的。我们往这个方法里传入我们刚刚获取到的MappedStatement对象和一个包装过的parameter，我们来看一下我们是如何包装parameter的。
![java-javascript](/img/in-post/six-mybatis/5.png)
如果深入的说，我都可以将这个方法单独提出来变成一篇文章，但在这里我就简单的给大家讲一下其原理，后面有时间我在单独提出来细致的讲一下。不知大家有没有印象我们在上一讲的时候我们对我们传入的值进行第一次包装，如果我们仅仅传入一个值，则直接返回给我们这个值，这个值不一定是普通的String或者Integer类型，也有可能是list，map或者数组等，但如果我们不是传入一个值，则会将我们传入的值做一个包装，每一个值都给他一个唯一的key，这个key可以使自己设定的，也可以是我们的属性名小写，同样mybatis也为我们固定的设置了一个key，就是我们常说的param1，param2。这是我们说的第一个包装，而我们的wrapCollection()将作为我们第二次包装的方法。我们首先去匹配，如果遇到list或者collection或者数组，则将这些做一个包装，key则为对应的list，collection或者arry，但对于我们已经包装过的map或者我们其他基本类型则直接返回。这就是我要说的二次包装，对应我们这次的debug测试，笔者只是简简单单的传入一个1，所以并没有进行这两次包装，就被返回了。
![java-javascript](/img/in-post/six-mybatis/6.png)
我刚才大篇幅的讲了mybatis中关于包装的方法，我们还是回到我们的querry()方法。在知道了我们传入的几个属性之后，我们进入我们的query()方法中，首先第一步，从字面的解释我们从MappedStatement对象中获取绑定的sql，把值给boundSql，而BoundSql又是个啥呢。
![java-javascript](/img/in-post/six-mybatis/7.png)
在boundSql中拥有众多属性，里面有我们的sql语句，还有我们在写sql语句的时候配置的值，还有我们要查询的id值，等等等等。代表了我们要查询的所有信息。
![java-javascript](/img/in-post/six-mybatis/8.png)
这一步是将创建一个二级缓存，如果我们要求有二级缓存，那么在下一步中我们将使用我们创建出来的缓存。
![java-javascript](/img/in-post/six-mybatis/9.png)
紧接着，将我们的众多对象，包括上两步创建出来的boundSql和我们的缓存key都带到我们的query()方法中。
![java-javascript](/img/in-post/six-mybatis/10.png)
进入方法先去判断我们是否设置过缓存，如果设置过则去通过我们的key搜索这个缓存，如果没有则要继续执行查询方法。
![java-javascript](/img/in-post/six-mybatis/11.png)
为了能够了解交互过程我没有在这里设置任何的缓存，所以直接跳到这一步，将我们的众多属性放入我们的query()方法中开始查询交互。
![java-javascript](/img/in-post/six-mybatis/12.png)
进入我们真正的query()之后，mybatis首先去通过key找我们的一级缓存，我们这里留意一下，mybatis先去找我们的二级缓存，然后再找我们的一级缓存的，同样这里并没有做任何的一级缓存。
![java-javascript](/img/in-post/six-mybatis/13.png)
跳过一级缓存的查询方法，我们来到queryFromDatabase()方法。
![java-javascript](/img/in-post/six-mybatis/14.png)
首先我们先给一级缓存设置上我们之前传入的key，等待查询之后value的传入。
![java-javascript](/img/in-post/six-mybatis/15.png)
我们接着来到我们的doQuery()方法，我们进入这个方法。


#### 总结



本篇文章我们主要讲解了mybatis是如何做对list,数组的包装，以及在mybatis中最为关键的缓存搜索与创建，mybatis首先去搜索我们的二级缓存，在没有得到二级缓存之后，mybatis又去一级缓存中去找，没有找到则去执行我们最最核心的方法doQuery()方法。而在下一讲，我们将真正的接触到mybatis的核心交互方法，大家是不是很期待呢。


#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

