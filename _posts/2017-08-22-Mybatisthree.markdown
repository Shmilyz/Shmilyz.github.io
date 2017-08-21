---
layout:     post
title:      "Mybatis源码分析第三讲之获取SqlSession对象"
subtitle:   ""
date:       2017-08-22 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第三讲之获取SqlSession对象



上一篇我们学习创建sqlSessionFactory对象的全过程，而这一篇文章我们将了解如何获取到SqlSession对象，我们先简单的回顾一下我们sqlSessionFactory对象中的属性。sqlSessionFactory是由一个configuration组成的，而configuration对象中拥有大量属性，上一篇我们也看了两个比较重要的属性，mappedStatements和mapperRegistry。我们再来看看这两个属性的内容。

![java-javascript](/img/in-post/third-mybatis/1.png)
![java-javascript](/img/in-post/third-mybatis/2.png)
 mapperRegistry以KV的形式进行保存，每一个接口对应一个代理工厂，这个我们后面再去说，我们通过这个工厂获取一个代理对象，由这个代理对象来进行操作，创建代理对象的原因是为了方便我们进行插件的开发。
 
 而另一个属性mappedStatements同样以KV的形式进行存储，每一个接口中的方法都对应一个Statements，而Statements中存放的是我们通过对xml文件解析后的结果，一个MappedStatement就代表一个增删改查标签的详细信息，以达到每一个方法对应一个xml文件中的一个增删改查标签，就像selectAll这个方法，对应的MappedStatement存放了我们的sql语句以及对查询的一些额外设置。
 
 
 我们上面对上一节进行了复习，下面我们简单的研究一下我们是如何获得SqlSession对象的。我们依然通过源码的方式进行讲解。
 
 ![java-javascript](/img/in-post/third-mybatis/3.png)
 我们一上来调用sqlSessionFactory对象的openSession()方法，其实调用的是openSessionFromDataSource()方法。我们来看这个方法，往方法中传入了一个execType，这个对象里面一个属性name值为SIMPLE。这个name属性来自我们的全局配置xml文件，而mybatis默认将这个属性(defaultExecutorType)设置为默认的SIMPLE，意思是这是一个默认的简单的执行器，属性defaultExecutorType的作用我们可能也见过，我们在做mybatis的批量操作时，我们要进行如下的设置，我们要在openSession()中添加ExecutorType.BATCH，如下图。

![java-javascript](/img/in-post/third-mybatis/4.png)
所以一上来我们先获取到我们执行器是调用defaultExecutorType属性的name值，这里我们没有批量操作所以得到默认的SIMPLE。
![java-javascript](/img/in-post/third-mybatis/5.png)
这一步我们要重点注意一下，这是四大对象中最早出现的一个，了解四大对象，有助于我们灵活的创造拦截器插件。通过调用configuration的newExecutor方法，我们得到了Executor，而Executor是一个接口，里面定义了许多增删改查的方法，我们最终底层进行增删改查时是要实现这里面的方法的(这个方法中会生成底层JDBC相关逻辑等等)，我们进入这个方法查看我们是如何得到Executor对象的。
![java-javascript](/img/in-post/third-mybatis/6.png)
进入方法后首先判断我们执行器的类，创建出我们对应的对象，我们这里是默认SIMPLE，所以new了一个SimpleExecutor()，而我们看到下面一行里面
![java-javascript](/img/in-post/third-mybatis/7.png)
如果我们在mapper中进行设置过二级缓存，那么我们就执行这一行我们这里有设置二级缓存，所以这里if判断进入并执行。
![java-javascript](/img/in-post/third-mybatis/8.png)
![java-javascript](/img/in-post/third-mybatis/9.png)
下一步的操作跟springMvc源码一样，我们遍历出所有的拦截器，由于作者设置过PageHelpler分页插件，所有我们可以从下面的executor对象中找到拦截器，我们也看到了拦截器的名称com.github.pagehelper.PageHelper。最后我们返回了executor。
![java-javascript](/img/in-post/third-mybatis/10.png)
 得到我们的executor之后，我们new了一个DefaultSqlSession(),并将executor和我们上一步得到的configuration传入。而DefaultSqlSession是SqlSession的实现类。从而我们得到了一个DefaultSqlSession对象。
![java-javascript](/img/in-post/third-mybatis/11.png)
最终我们获取到了我们想要的sqlSession，而且我们看到我们的sqlSession中的两个重要属性，一个是我们上一步得到的configuration和我们这一步得到的四大对象之一executor。






#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

