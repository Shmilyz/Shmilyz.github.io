---
layout:     post
title:      "Mybatis源码分析第七讲之Mybatis与数据库交互核心源码"
subtitle:   ""
date:       2017-09-04 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第七讲之Mybatis与数据库交互核心源码



在写这一篇文章之前，作者都记不清之前写过多少篇mybatis源码分析的文章了，但最终目的都是为了今天这篇文章做一个铺垫，感谢大家一直以来的关注，有些人还特意给我发邮箱指出我的错误和对我的一些鼓励，我非常感谢他们，废话不说，我们接着开启我们新的一讲，我们继续探究mybatis的源码。我们接着上一讲的内容，贴出来上一讲的最后一张源码截图。



![java-javascript](/img/in-post/seven-mybatis/1.png)
上一讲我们讲到，mybatis去判断不存在一级缓存之后，先去创建了一个一级缓存，并且设置了key，紧接着就是执行doQuery()的方法。我们进入这个方法。
![java-javascript](/img/in-post/seven-mybatis/2.png)
该方法的第一步是创建一个我们熟悉的不能再熟悉的Statement对象出来，我们在之前反复的使用原生的JDBC进行增删改查交互的时候，一直在用这个Statement，而我在第一讲的时候反复强调，我们的mybatis的核心其实就是JDBC。
![java-javascript](/img/in-post/seven-mybatis/3.png)
![java-javascript](/img/in-post/seven-mybatis/4.png)
我们先是讲我们的configuration得到，然后下一步去创建一个StatementHandler对象，这个StatementHandler是我们四大对象的一个，这也是我们遇见的第二个。StatementHandler的作用是可以创建出我们的Statement对象。我们来看看是如何完成创建的。
![java-javascript](/img/in-post/seven-mybatis/5.png)
我们首先new一个RoutingStatementHandler()，我们进入这个方法。
![java-javascript](/img/in-post/seven-mybatis/6.png)
![java-javascript](/img/in-post/seven-mybatis/7.png)
![java-javascript](/img/in-post/seven-mybatis/10.png)
![java-javascript](/img/in-post/seven-mybatis/8.png)
![java-javascript](/img/in-post/seven-mybatis/9.png)
![java-javascript](/img/in-post/seven-mybatis/15.png)
我们首先拿到我们设置的属性，我们getStatementType()，作者并没有设置过这个属性，所以我们使用默认的PREPARED，该属性设置为可以预编译。我们创建了一个PreparedStatementHandler。这里要注意一点的是我们在创建PreparedStatementHandler的时候，调用了父类BaseStatementHandler的构造方法，我们可以看到在父类中我们还创建了两个Handler，也就是我们四大对象的三和四，parameterHandler和resultSetHandler。(这里我们要注意一点的就是我将两个handler创建过程提前拿出来讲一下，跟前两个对象一样，我们都是执行了遍历拦截器和包装handler的过程)创建后我们继续debug执行。
![java-javascript](/img/in-post/seven-mybatis/11.png)
看这一行不知道大家熟不熟悉，在之前的executor创建讲解中我们也有相同的步骤。我们遍历拦截器，用拦截器包装我们的StatementHandler。
![java-javascript](/img/in-post/seven-mybatis/12.png)
我们用prepareStatement()来创建我们的prepareStatement对象。
![java-javascript](/img/in-post/seven-mybatis/13.png)
prepareStatement()方法中我们先创建一个连接，然后调用handler.prepare()进行预编译，这一步与我们原生的 connection.prepareStatement()方法一样，进行预编译。最终我们得到了这个prepareStatement对象。
![java-javascript](/img/in-post/seven-mybatis/14.png)
我们是通过parameterHandler.setParameters()进行参数预编译的，我们parameterHandler的作用就是进行参数预编译设置的。
![java-javascript](/img/in-post/seven-mybatis/16.png)
而parameterHandler进行参数预编译的原理就如图所示，我们使用创建好的 typeHandler的setParameter()方法，往里面传入我们刚刚设置的prepareStatement对象来进行参数设置。
![java-javascript](/img/in-post/seven-mybatis/17.png)
![java-javascript](/img/in-post/seven-mybatis/18.png)
我们调用我们的StatementHandler的query()方法来执行最后的交互，这部分工作也就交给了JDBC.我们也能看到，其实跟我们写JDBC的步骤是一样的。调用PreparedStatement的execute()方法将我们设置的参数发送给我们的数据库。
![java-javascript](/img/in-post/seven-mybatis/19.png)
我们调用我们四大对象最后一个resultSetHandler的方法来封装我们得到的结果。
![java-javascript](/img/in-post/seven-mybatis/23.png)
我们都知道在执行完ps.execute();方法之后，通过调用getResultSet();就可以得到ResultSet，然后再对其遍历，然后在进行类型转换，映射等操作，最后得到我们想要的结果对象。
![java-javascript](/img/in-post/seven-mybatis/20.png)
![java-javascript](/img/in-post/seven-mybatis/21.png)
![java-javascript](/img/in-post/seven-mybatis/22.png)
通过一步一步的返回，包括我们是按照多返回值查询的，我们还要进行一步将我们的list集合只去我们的第一个，最终返回得到结果。





#### 总结


就此，我们真正的对mybatis进行数据库交互这部分代码有了完整的认识，但并没有结束，不知道大家是否发现我们没有还没有对四大对象的拦截器这部分操作进行任何的源码分析，而下一讲我们除了要进行总结外，我们对拦截器这部分源码要进行一个讲解，为我们的插件开发做铺垫。



#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

