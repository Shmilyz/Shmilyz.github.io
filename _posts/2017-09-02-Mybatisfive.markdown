---
layout:     post
title:      "Mybatis源码分析第五讲之Mybatis与数据库交互核心前铺垫(一)"
subtitle:   ""
date:       2017-09-02 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第五讲之Mybatis与数据库交互核心前铺垫(一)

在前面几讲中我们一步一步的接触到了mybatis执行增删改查的核心，而这一讲我们将真正的接触mybatis实现增删改查的核心，我们以查询实现为例，上一讲我们得到了我们的代理对象mapperProxy，我们也知道了在我们的mapperProxy中保存了sqlSession这个属性，我们通过调用
mapperProxy的增删改查方法来进行我们与数据库的交互。而这篇讲解我们将通过调用我们的查询方法，继续以打断点的方式来看一看我们是如何实现查询的。



![java-javascript](/img/in-post/five-mybatis/1.png)
我们在我们调用的查询方法这句话上打上断点，我们进入这个方法。
![java-javascript](/img/in-post/five-mybatis/2.png)
![java-javascript](/img/in-post/five-mybatis/3.png)
![java-javascript](/img/in-post/five-mybatis/4.png)
我们发现我们进入的是InvocationHandler的方法invoke()，我们之前说过我们以后的增删改查其实都是一个代理类在替我们操作的，所以我们在执行目标方法之前我们要执行我们这个InvocationHandler的invoke()方法，进入这个方法我们首先将我们的方法（这里我们的方法是selectByPrimaryKey()）包装成一个mybatis自己的方法，mapperMethod,然后执行mapperMethod的execute()执行方法。调用的时候我们传入sqlSession和args，这个args就是我们在调用我们查询方法的时候传入的内容。我们进入这个方法，查看执行流程。
![java-javascript](/img/in-post/five-mybatis/5.png)
![java-javascript](/img/in-post/five-mybatis/6.png)
我们进入了这个方法，在这里首先是进行判断，由于之前 我们解析了这些标签xml文件，所以我们可以找到我们增删改查的操作，而这里我们是查询SELECT，进入查询方法。
![java-javascript](/img/in-post/five-mybatis/7.png)
进入到我们的查询方法看似很复杂，其实很好梳理，我们先去判断我们这个方法是否由返回值或者返回的是map，游标的，或者返回多个的而我们这里返回的是我们自己定义的Employee对象。
![java-javascript](/img/in-post/five-mybatis/8.png)
我们进入我们else流程中，第一步就是调用我们method的convertArgsToSqlCommandParam()方法,该方法的作用我们就不在详细讲解了，我这里把这个方法的源码贴上，简单的说一下。
![java-javascript](/img/in-post/five-mybatis/9.png)
该方法的作用是将我们传入的参数进行判断，由于我们传入的其实是一个数组，所以我们要对数组进行解析，如果数组里面的值只有一个我们就将这个一个值进行返回，如果有多个值，则将数组遍历并进行包装成一个map返回（我们都知道我们可以自定义我们在#{}中取值的key值，还有我们的param写法，pojo写法，map写法等等，都是靠我们这个解析器来让它成为一个完整的map，我们才能通过#{}来获取到我们的值），我们这里只有一个值1，所以我们在下图可以看到该方法返回了一个Integer类型的值。
![java-javascript](/img/in-post/five-mybatis/10.png)
在完成对集合的解析之后，我们执行这一步，在对这一步进行解析前我们看一下传入的两个值，一个是获取到我们方法的名字com.shmilyz.mbg.dao.EmployeeMapper.selectByPrimaryKey，而另一个传入的值对应我上面所说的，传入一个Integer类型的参数1，在我们解决了这些小问题之后，我们来看我们这个比较重要的方法，我们下面来解析一下这个方法。调用sqlSession.selectOne()方法，我们进入这个方法。
![java-javascript](/img/in-post/five-mybatis/11.png)
我们进入到这个方法，第一步我们是调用DefaultSqlSession的selectList方法，所以说虽然我们调用的是我们的selectOne方法，但实际上我们依然调用的是selectList来进行操作，按照查询多个得方法进行操作，只是我们在下面只调用方法只取得我们lis集合的第一个值。在铺垫完这些小的细节之后，我们就要进入一个比较核心的方法了，我们来看看mybatis是如何完成查询的。

#### 总结

在这一篇中，我们用更多的篇幅来讲核心方法之前的准备步骤，而这些准备步骤也将很好的为我们的核心方法进行服务。下面我们继续靠近mybatis最最核心的交互方法。


#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

