---
layout:     post
title:      "Mybatis源码分析第一讲之Mybatis概述"
subtitle:   ""
date:       2017-08-10 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### MyBatis简介与其他同类框架的对比

MyBatis原名iBatis，它是一个支持普通SQL查询,存储过程和高级映射的优秀持久层框架等等等，这些概念和历史百度都能搜到，我就不在这里赘述了，简单的说MyBatis就是一个基于JDBC的可以与数据库进行交互的半自动的框架，正式因为JDBC的繁琐，导致我们有必要将JDBC进行封装作为底层方法，并且又添加了多种功能，比如对sql语句编写等，更方便更优雅的让我们与数据库进行交互。这也就引出了MyBatis与另一款全自动全映射的数据库交互框架hibernate的区别。我们都知道hibernate标榜的就是全自动全映射，啥叫全自动呢，就是说hibernate旨在让程序员不用会写sql都可以与数据库进行交互，这种简单直接的方法得到了很多程序员的青睐，而慢慢的大家发现，hibernate并不能替我们编写一些复杂的sql语句，也不能对这些复杂的sql语句进行优化，大大的影响的交互效率，而如果我们想要自己编写sql就得要去学hql，反而增加了我们的负担。由于是全自动，所以我们无法控制我们想到javabean中的哪个或哪几个值，而是映射了全部。这也大大增加了内存负担。所以我个人的观点，全自动全映射是hibernate的优点，同样也是hibernate的缺点，致命的缺点。而MyBatis则完美的解决了这个问题。



> 作者注：观点仅代表个人观点，如有错误，欢迎指点。


我们都知道jdbc的几个核心的流程:
- 编写sql
- 预编译
- 设置值
- 执行sql语句
- 封装结果并返回

而MyBatis将编写sql这一步流程开放给开发人员，所有的sql，不论简单的还是复杂的全部由开发人员去完成。而其他几步仍交由MyBatis来去做，所以在介绍MyBatis的时候我说，MyBatis是一个半自动的框架。我并不了解MyBatis和hibernate两者与数据库交互的效率哪个更高，我也没有做过测试，但我认为两者由于底层方法都是JDBC所以两者之间的效率应该没有多大的差距。而更多的开发人员选择MyBatis就是看中它可以自由的编写sql语句，而MyBatis还为我们提供了两个好玩的福利，MyBatis的插件开发和MyBatis的mbg逆向工程。插件开发不多说，mbg对很多开发者比较陌生，mbg旨在不让用户写一个javabean不让用户写一个sql语句也就是不让用户创建一个接口类和对应的mapper。全部由MyBatis为我们生成这些文件。是不是很有趣。

> 作者注：MyBatis另一大优点sql语句与java文件的分离，我们将sql语句写在mapper文件中，这样就可以不会因为修改javade中的sql语句导致java文件进行重新编译。


#### 我们为什么要分析MyBatis的源码
- 我们通过源码分析可以为我们进行插件开发提供帮助，我们可以通过源码找到MyBatis中最重要的四大对象，并且查看拦截器的调用位置等。
- 通过一层一层的分析我们为了最终会拨开MyBatis的面纱，也就是底层方法JDBC
- 通过对源码的分析提升我们的逻辑分析能力，更好地理解与运用MyBatis
- 。。。。

#### MyBatis源码分析步骤
我们都知道我们要使用MyBatis与数据库进行交互要创建接口类里面有增删改查的方法，还要有全局配置文件以及接口类对应的mapper文件，里面写sql语句等其他属性，最后我们要一个一个调用这些文件。代码如下:
```html
  String resource = "mybatis-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory =new SqlSessionFactoryBuilder().build(inputStream);
  SqlSession openSession=sqlSessionFactory.openSession(true);
  EmployeeMapper mapper=openSession.getMapper(EmployeeMapper.class);
	Employee employee=mapper.getEmpById(1);
```

而我们将按照代码的顺序分几次来分析Mybatis的源码。



#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

