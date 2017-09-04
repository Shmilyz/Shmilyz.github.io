---
layout:     post
title:      "Mybatis源码分析第八讲之对Mybatis四大对象总结"
subtitle:   ""
date:       2017-09-05 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第八讲之对Mybatis四大对象总结


通过对上面几讲对mybatis源码的分析，我们对mybatis与数据库的交互有了完整的认识，而本篇文章我们主要是从四大对象的角度，总结mybatis与数据库交互的全过程（查询角度）以及四大对象和插件的联系。

我们在进行总结之前，我们先来回顾一下JDBC进行数据库交互的全过程，总结起来就是五个步骤，编写sql --》预编译  --》插入值 --》执行sql  --》封装结果，我们通过原生JDBC的查询流程来进行一个回顾。

```java
Connection conn = null;
PreparedStatement  stm = null;
ResultSet rs = null;
 try {
   Class.forName("XXX");
   conn = DriverManager.getConnection("XXX");
   stm =conn.prepareStatement("select * from tianjunzhe where id=? ");
   //预编译
    pstmt.setInt(1, 1);
  //设置参数
    stm.execute();
	  //将参数发送给数据库执行查询。
    rs = stm.getResultSet();
  //得到结果
    while(rs.next()){
     XXX
   }
```

#### 四大对象与数据库交互的联系

首先大家是否记得我们是通过一个代理对象proxy来进行增删改查的，而在内部我们是调用我们的四大对象之一Executor来进行实际的增删改查的，然后就是对我们传入参数进行两次包装的过程，紧接着我们创建了四大对象之二StatementHandler，我们通过StatementHandler来执行预编译，插入值，执行sql，封装结果这四大步骤。而我们可以更加具体的来对StatementHandler进行一个分工，我们StatementHandler执行了预编译，相当于创建出来我们的PrepareStatment对象来。紧接着使用我们的四大对象之三parameterHandler来进行一个设置参数的工作，然后StatementHandler进行stm.execute();将我们设置的参数传入数据库，最后使用四大对象的最后一个resultSetHandler对象来完成映射处理封装结果的作用。而一有个打零工的Handler默默的做着类型转换和参数解析的工作，他就是TypeHandler，parameterHandler和resultSetHandler的参数设置和处理结果都需要它的帮助。


#### 四大对象与插件的联系


![java-javascript](/img/in-post/other-mybatis/1.png)

在我们插件开发的时候我们都要实现Interceptor接口，实现接口中的三个方法。intercept() plugin()和setProperties()三个方法。setProperties()的作用是插件的初始化。如图所示我们要遍历所有的插件，并且执行每一个插件的plugin()方法。

而plugin()方法中我们要做的就是将四大对象进行包装，让他变成一个我们熟知的代理对象，代理对象中有三个属性target(四大对象)interceptor(我们写的插件)signatureMap(KV键值对)。


![java-javascript](/img/in-post/other-mybatis/2.png)

在对我们的四大对象包装成代理对象之后返回。在我们定义插件的时候我们都会在插件类前面标注注解，在注解@Interceptor中写明我们要拦截哪一个四大对象的哪一个方法，当我们执行增删改查的时候，在执行到要拦截的方法前，就调用插件的intercept()方法，来执行操作，再然后才执行我们拦截的那个方法。

这里可能有点迷惑，我举个例子，我们通过注解@Interceptor拦截了我们Executor中的某一个方法，其他的三个Handler对象我没有去拦截，我们依然要执行插件类里的plugin()方法时，只是我们会去判断是否要包装成一个代理对象，当发现不需要进行包装的时候我们就直接返回我们原来的对象，而需要被插件处理的对象则要进行包装。并且当执行到对应方法的时候代理对象先执行intercept()方法后执行我们的真正方法。这就是插件的原理。





#### 总结


该篇文章我们主要是对原生JDBC查询实现的流程进回顾，然后又总结梳理了mybatis是如何实现查询的，其实说白了就是通过我们的四大对象将我们原生JDBC查询实现的那几句代码进行了一个拆分与整合的过程。在最后我们又分析了插件和四大对象之间的关系并结合源码走了一遍插件的执行流程，了解了插件。我们通过对源码的分析最后旨在希望大家对mybatis有更深入的了解。最后感谢大家对我的支持。


#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

