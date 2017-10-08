---
layout:     post
title:      "教你写框架系列之Spring MVC第二讲框架的运行流程源码分析(巨详细)"
subtitle:   ""
date:       2017-10-11 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Spring MVC 
    - Spring MVC教学
---


	


#### 作者说

从这一讲我们将对Spring mvc 的运行流程进行一个分析，我们通过debug源码的方式来为大家呈现出spring mvc的秘密。

#### Spring MVC框架的运行流程源码分析

作者先搭建好一个spring mvc的项目，我们写一个目标方法login()，我发请求执行这个方法，直到渲染视图显示对应的网页为止，我们对这一个过程做一个流程源码分析。我不会对每一句代码都做一个讲解，包括后面如何渲染视图我们都不会仔细想，我们应该把重心放在整个流程上面，不管以后我们面试还是做开发，我们脑子里都要有这套流程。下面我开始截图进行源码分析。

![java-javascript](/img/in-post/first-mvc/11.png)

首先Spring mvc执行到如图所示的方法doDispatch(),我们看到括号内将request和response传入。
并且我们往下看Spring mvc为我们创建了一个HandlerExecutionChain对象，名字叫做mappedHandler，我们记住这个对象，关系整个执行流程。我们看下一张图。看mappedHandler是如何获取的。

![java-javascript](/img/in-post/first-mvc/12.png)

看上图蓝底标注的位置，我们的mappedHandler对象是由getHandler()这个方法获得的，我们进入这个方法看看getHandler()方法的作用。

![java-javascript](/img/in-post/first-mvc/13.png)

这里有些乱，没关系我慢慢的讲，首先执行一个for循环，我们遍历的是一个个HandlerMapping对象，如果这个对象的getHandler()有值，那么返回handler，如果为null则继续遍历。如果一直没有找到handler，那么返回null。我们下面来看看什么是HandlerMapping

![java-javascript](/img/in-post/first-mvc/14.png)

从字面的解释这是一个请求与处理器之间的映射。我们每次不都是设置一个@RequestMapping吗，里面写上请求，而这个注解的下面写的就是我们的处理方法handler，而HandlerMapping要做的就是通过请求找到我们的处理方法handler。我们看上图handlerMappings里面有四个值，0对应的那个值得HandlerMapping就是做通过请求找方法的，而3对应的值我们这里重点说一下，我们每次配置mvc的时候都要配置一句话  <mvc:default-servlet-handler/>这句话的意思我不知道大家还记不记得，每个通过DispatcherServlet的请求如果是一个静态资源的请求，则会被3对应的HandlerMapping找到静态资源。也就是说如果配置了上面的那句就会有这个HandlerMapping，如果没配置就没有HandlerMapping那么也就不会去找静态资源文件。

我们继续回到刚才的思路，现在我们的请求会被0对应的HandlerMapping处理，找到请求对应的方法，有了handler就可以返回了。我们返回来看看HandlerExecutionChain是个什么。

![java-javascript](/img/in-post/first-mvc/15.png)

![java-javascript](/img/in-post/first-mvc/16.png)

我们看这两张图，刚才调用getHandler()返回了我们的HandlerExecutionChain对象，我们点开这个对象，发现HandlerExecutionChain对象一共有两个重要的属性，我们的过滤器和我们的处理器handler，点开这个handler，我们发现红色框位置就是我们获得到的handler方法，这样我们就可以很清晰的了解了HandlerExecutionChain是干什么的了。我们来梳理一下啊，我们通过HandlerMapping对象得到我们的HandlerExecutionChain对象，HandlerExecutionChain对象里面有我们要执行的目标fa方法。很清晰。

为了能更好的了解这个过程，我们在做一个小实验，如果这时候我们发一个静态的html请求，我们会得到什么效果呢，大家可以去自己试一试，我这里就不贴图了，我直接讲结果，如果配置了 <mvc:default-servlet-handler/>这句话，那么我们可以得到一个HandlerExecutionChain对象，哪怕静态资源是不存在的，都会创建HandlerExecutionChain对象，除非你没配置上面的那句话。这部分内容讲完，我们继续往下看。

![java-javascript](/img/in-post/first-mvc/17.png)

debug执行到947蓝标处，我们得到一个HandlerAdapter对象，我们看英文就可以看出来HandlerAdapter是适配器的意思，我们在安卓 web等等好多地方都见过适配器，什么是适配器，电源插头是适配器，转接线是适配器，把不合适弄成合适的就是适配器。那么这里的适配器是干什么的呢，我们可以看到蓝标出距离967行调用目标方法还很远，除了调用前置拦截器外我们还要做很多事情，我们spring mvc为大家提供的表单数据检验，数据类型的转换，这些都是需要适配器的方法去执行。直到调用目标方法，都是它。我们继续往下看。

![java-javascript](/img/in-post/first-mvc/18.png)

如果看过我前面讲解的同学，一定不会陌生拦截器方法执行源码分析这一块内容。如果大家没看过，大家可以先看一看。

我们这里先执行的是拦截器的PreHandle()方法，我们对所有拦截器的PreHandle()进行遍历。我们继续往下看。

![java-javascript](/img/in-post/first-mvc/19.png)

967行我们开始执行目标方法了，我们就去调我们自己写的login()方法，这里就不演示debug了，我们只要知道这一步去执行目标方法，并且注意我们这里返回的是一个什么啊，是一个ModelandView对象。我之前也说过不论你的目标方法handler返回的是String类型还是Model还是Map还是ModelandView最后都会包装成ModelandView，啥是ModelandView，模型和视图。好，我们继续往下看。大家忍一忍很快就结束了。

![java-javascript](/img/in-post/first-mvc/20.png)

974行对拦截器的PostHandle()方法进行遍历。这没有什么要说的。往下看我们的渲染视图。

![java-javascript](/img/in-post/first-mvc/21.png)

![java-javascript](/img/in-post/first-mvc/22.png)

994行我们进行渲染视图的工作，我们进入这个方法看看是如何渲染视图的。首先是我们的异常处理模块。如果说我们在执行目标方法的时候发生异常，或在其它情况下发现异常，我们就要在这个模块进行异常的处理，如果有异常，我们则执行1034行的processHandlerException()方法处理异常。不知道我们还是否记得我们可以通过@ExceptionHandler来配置什么异常对应什么页面。那么processHandlerException(）方法就是做这个的。我们继续往下看。

![java-javascript](/img/in-post/first-mvc/23.png)

蓝标render()要做的是渲染视图的工作，我们进入这个方法。

![java-javascript](/img/in-post/first-mvc/24.png)

![java-javascript](/img/in-post/first-mvc/25.png)

![java-javascript](/img/in-post/first-mvc/26.png)

![java-javascript](/img/in-post/first-mvc/27.png)

![java-javascript](/img/in-post/first-mvc/28.png)

上面这几张图是我们进行渲染视图转发页面的代码，我截取了几张我们写servlet时我们知道的方法。就这样完整的过程就讲完了。下面我画了一张图，跟着图我们先走一遍。

![java-javascript](/img/in-post/first-mvc/29.png)

就这样我们对Spring mvc的运行流程进行了细致的分析，这对我们使用spring mvc框架有非常好的作用。大家读起来很吃力没关系，多看几次我相信一定能摸着规律的。



#### 总结

这一讲我们讲解了spring mvc的拦截器如何配置使用，以及对拦截器部分源码进行了分析。下一讲我们将对spring mvc异常进行处理。

#### 著作权声明

作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

