---
layout:     post
title:      "Mybatis源码分析第四讲之代理对象的获取"
subtitle:   ""
date:       2017-09-01 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第四讲之代理对象的获取

我们这一讲我们继续研究mybatis的运行原理，我们将研究getMapper如何获取到接口的代理对象，我们对 EmployeeMapper employeeMapper=sqlSession.getMapper(EmployeeMapper.class);这句话进行断点分析。


![java-javascript](/img/in-post/four-mybatis/1.png)
前三讲我们分别讲解了如何获取sqlSessionFactory对象没我们是通过将我们的全局配置文件和我们的xml文件一一解析都保存到我们的configuration中，然后我们返回得到的DefaultSqlSessionFactroy也就包含了我们的configuration。而第三讲我们讲解了如何得到SQLSession对象，我们通过返回得到的DefaultSqlSession对象，包含了我们经常提到的configuration和新创建出来的四大对象之一executor。executor的作用就是完成我们底层的增删改查的，而我们的executor里面要注意一下，我们首先去检查我们的是否设置了二级缓存，如果有我们使用我们的CaChingExecutor来讲executor进行包装，而更加重要的是下一步将对所有的拦截器进行遍历，再次包装，得到我们的executor。
![java-javascript](/img/in-post/four-mybatis/2.png)
我们在这一行打上断点，我们来分析一下getMapper()这个方法，我们进入这个方法。
![java-javascript](/img/in-post/four-mybatis/3.png)
我们发现源码中根本的执行方法是调用configuration.getMapper(接口类型)，我们的configuration里面保存了大量的解析出来的信息，由这个configuration来进行下一步操作。
![java-javascript](/img/in-post/four-mybatis/4.png)
我们惊喜的发现configuration的getMapper()依然是一个幌子，而真正操作者是一个mapperRegistry，我们之所以觉得惊喜是因为mapperRegistry是我们之前反复提到过的，我们再来回顾一下这个属性。
![java-javascript](/img/in-post/four-mybatis/5.png)
mapperRegistry以KV的形式进行保存，每一个接口对应一个代理工厂，我们通过这个工厂获取一个代理对象，由这个代理对象来进行操作，创建代理对象的原因是为了方便我们进行插件的开发。而下面我们来看一看我们的mapperRegistry是如何进行完成getMapper()方法的
![java-javascript](/img/in-post/four-mybatis/6.png)
.进入方法的第一句映照了我们之前所说关于mapperRegistry的讲解，这一步中我们通过type得到我们的value，也就是想要得到我们的mapperProxyFactory对象，我们对拿到的mapperProxyFactory对象进行操作。
![java-javascript](/img/in-post/four-mybatis/7.png)
调用我们刚刚得到的mapperProxyFactory的newInstance方法，并传入我们的sqlSession，我们进入这个方法。
![java-javascript](/img/in-post/four-mybatis/8.png)
进入这个方法后，第一步我们先将我们的sqlSession等对象传入我们的方法，而最后我们获得到了一个mapperProxy对象，也就解决了我们之前的一个小问题，我们也就知道了我们是由我们的mapperProxyFactory工厂来获得mapperProxy对象。
![java-javascript](/img/in-post/four-mybatis/9.png)
我们来看看mapperProxy是个什么东西，我们看到，mapperProxy类中有很多属性，比如我们的
sqlSess还有我们在mapperProxyFactory中看到的mapperInterface还有methodCache这两个属性，我们都被mapperProxy类包含，而最重要的是要看这个类实现了InvocationHandler这个接口，我们在学Spring的时候在讲ioc原理的时候我们遇到过这个InvocationHandler，而它在我们java中做动态代理所用到，我么就此也知道了我们的mapperProxy除了包含大量mybatis要用到的属性外，它还是一个InvocationHandler。
![java-javascript](/img/in-post/four-mybatis/10.png)
![java-javascript](/img/in-post/four-mybatis/11.png)
 接下来我们执行了newInstance()方法，我们将我们的mapperProxy传进去，我们调用的就是我们Java中的Proxy.newProxyInstance(mapperProxy)方法，该方法的执行可以返回得到一个创建好的代理对象，我们的mapperProxy就成为了一个代理对象了。
![java-javascript](/img/in-post/four-mybatis/12.png)
我们一步一步的返回，回到我们最开始打断点的地方，我们查看我们得到的employeeMapper对象，我们的employeeMapper对象变成了一个代理对象，就是我们返回的mapperProxy，我们的mapperProxy同样实现了我们的EmployeeMapper接口，而下面的每一步将有我们的代理对象来操作我们定义的方法。


#### 总结
最终我们通过我们对EmployeeMapper employeeMapper=sqlSession.getMapper(EmployeeMapper.class);这句话的分析，我们得到结论，我们最后得到的employeeMapper对象实际上是一个mapperProxy对象，是一个代理对象，而我们接下来的每一次调用employeeMapper中方法的时候我们其实都是由mapperProxy来调用的。而下一讲我们来看一看代理对象是如何执行增删改查方法的。




#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

