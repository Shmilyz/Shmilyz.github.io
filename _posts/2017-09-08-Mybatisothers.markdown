---
layout:     post
title:      "Mybatis源码分析第十讲之最后总结"
subtitle:   ""
date:       2017-09-08 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第十讲之最后总结



通过解析我们的全局配置文件和我们自己编写的mapper文件来获取到Configuration对象。紧接着创建出一个SQLSession(DefaultSqlSession)对象，里面包含了Configuration对象和四大对象之一的Executor对象，我们在全局配置中根据设置不一样的Executor属性，来创建对应的Executor实现类，(在配置文件中我们可以设置Executor的类型，SIMPLE，REUSE,BATCH,我们在批量插入等操作时常用)，Executor的作用反复强调是为了完成我们的增删改查方法的。再然后我们创建了我们对应类的代理对象，代理对象中有我们的SqlSession，我们以后的调用方法实际就是调用我们的代理对象。紧接着我们开始执行我们的代理对象，其实是执行我们的Executor(SimpleExecutor)实现类中的查询方法，该查询方法首先创建了一个Statement(PrepareStatment)对象，然后创建四大对象之二StatementHandler，我们通过StatementHandler来执行预编译，插入值，执行sql，封装结果这四大步骤。而我们可以更加具体的来对StatementHandler进行一个分工，我们StatementHandler执行了预编译，相当于创建出来我们的PrepareStatment对象来。紧接着使用我们的四大对象之三parameterHandler来进行一个设置参数的工作，然后StatementHandler进行stm.execute();将我们设置的参数传入数据库，最后使用四大对象的最后一个resultSetHandler对象来完成映射处理封装结果的作用。而一有个打零工的Handler默默的做着类型转换和参数解析的工作，他就是TypeHandler，parameterHandler和resultSetHandler的参数设置和处理结果都需要它的帮助。最终我们将我们得到的封装对象进行输出。


#### Mybatis源码分析十讲链接

[Mybatis源码分析第一讲之Mybatis概述](https://shmilyz.github.io/2017/08/10/Mybatisonec/)

[Mybatis源码分析第二讲之SQLSessionFactory的初始化](https://shmilyz.github.io/2017/08/11/Mybatistwo/)

[Mybatis源码分析第三讲之获取SqlSession对象](https://shmilyz.github.io/2017/08/22/Mybatisthree/)

[Mybatis源码分析第四讲之代理对象的获取](https://shmilyz.github.io/2017/09/01/Mybatisfour/)

[Mybatis源码分析第五讲之Mybatis与数据库交互核心前铺垫(一)](https://shmilyz.github.io/2017/09/02/Mybatisfive/)

[Mybatis源码分析第六讲之Mybatis与数据库交互核心前铺垫(二)](https://shmilyz.github.io/2017/09/03/Mybatissix/)

[Mybatis源码分析第七讲之Mybatis与数据库交互核心源码](https://shmilyz.github.io/2017/09/04/Mybatisseven/)

[田俊哲](https://shmilyz.github.io)

[田俊哲](https://shmilyz.github.io)
[田俊哲](https://shmilyz.github.io)

#### 总结


我们通过这几讲我们对mybatis的使用应该变得更加游刃有余了，mybatis的好处和缺点我在第一讲的时候已经跟大家阐述过个人观点了，这里就不在阐述了。总之作为现在和数据库交互的新兴框架，它还是解决了hibernate没有解决的问题。mybatis源码的分析也就到此结束了，下面我也在着手对Linux和数据结构的讲解。谢谢大家的鼓励。我们下一个专题见。



#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

