---
layout:     post
title:      "Spring MVC源码分析系列之获取HandlerExecutionChain对象"
subtitle:   ""
date:       2017-07-21 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Spring MVC
    - Spring
---



#### 对HandlerExecutionChain的初认识

我先来给大家简单的提供一个简单的认识，这样我们在进行源码分析的时候我们可以更快的理解。

根据官方提供的信息，HandlerExecutionChain的中文名称叫做处理器调用链，这里面包含了handler（handler是目标处理器的意思，里面的核心装载了我们的目标方法，就是我们自己定义的那些方法）还有一些相关的拦截器。而HandlerExecutionChain对象的获取是由HandlerMapping获取的，由于HandlerExecutionChain对象里面搭载了拦截器和核心方法，所以在下面每一步运行流程都会调用这个HandlerExecutionChain对象的Handler或者拦截器。这就是对HandlerExecutionChain的初认识。下面我们将通过Debug一步一步的为大家讲解我们是如何获取HandlerExecutionChain的，以及HandlerExecutionChain的用处都有哪些。




#### 对HandlerExecutionChain源码分析

![java-javascript](/img/in-post/second-mvc/1.png)
<small class="img-hint">断点执行到这一行，通过执行该类的getHandler()方法获得mappedHandler。（由于作者马虎，忘记截取mappedHandler是什么，它就是一个HandlerExecutionChain），我们点进去这个getHandler()方法。</small>
![java-javascript](/img/in-post/second-mvc/2.png)
<small class="img-hint">进入getHandler()方法，我们也就看见，HandlerExecutionChain对象handler是由HandlerMapping获取而来的，</small>
![java-javascript](/img/in-post/second-mvc/3.png)
![java-javascript](/img/in-post/second-mvc/4.png)
<small class="img-hint">我们发现HandlerMapping是一个接口，而由AbstractHandlerMapping来实现这个接口，实现getHandler()这个方法。</small>
![java-javascript](/img/in-post/second-mvc/5.png)
<small class="img-hint">我们找到这个类里面的getHandler()方法，我们看看是如何实现的，首先先获取到核心handler，然后经过一系列的判断，最后将handler放入该类的getHandlerExecutionChain()方法中，而通过这个方法我们能获取到一个AbstractHandlerMapping对象executionChain。</small>
![java-javascript](/img/in-post/second-mvc/6.png)
<small class="img-hint">这张图清楚地显示了Handler到底是个什么东西。</small>
![java-javascript](/img/in-post/second-mvc/7.png)
<small class="img-hint">我们放大窗口看一下这里的核心内容method，我们发现这个method等于我们自己写的方法。</small>
![java-javascript](/img/in-post/second-mvc/8.png)
<small class="img-hint">我们点击进入getHandlerExecutionChain()这个方法，我们可以清楚地看到在这里将handler和interceptor放入到我们的HandlerExecutionChain对象chain</small>
![java-javascript](/img/in-post/second-mvc/9.png)
<small class="img-hint">我们看一下要返回的chain，里面放着handler和interceptors，由于笔者没有自定义intercept，所以里面的interceptors只有默认的，而handler则是完整的存着method等。</small>
![java-javascript](/img/in-post/second-mvc/10.png)
<small class="img-hint">最后我们得到完整的HandlerExecutionChain对象chain,在返回给最初的mapperHandler中。完成HandlerExecutionChain对象的创建</small>


#### HandlerExecutionChain对象到底有什么用呢?

![java-javascript](/img/in-post/second-mvc/11.png)
![java-javascript](/img/in-post/second-mvc/11.png)
<small class="img-hint">我们来搜索mappedHandler，发现我们在HandlerAdapter对象的获取或者其他很多地方都需要HandlerExecutionChain对象（其实就是需要HandlerExecutionChain对象中的handler）而且我们可以调用HandlerExecutionChain对象的三个跟拦截器有关的方法，applyPreHandle，applyPostHandle和applyAfterConcurrentHandlingStarted。</small>

#### 笔者说

由此，通过对源代码的分析我们清楚地认识到了HandlerExecutionChain是什么，它的获取流程以及它的用处，为我们更好地使用Springmvc奠定了基础，而下一篇我们将继续分析Springnvc关于拦截器和完整的运行流程这部分的源代码。不知道到读者有没有发现我们一直围绕着DispatcherServlet.java这个类来分析，这个类在Springmvc中占有很核心的位置，读者不妨也去试着读一读，很有趣的。最后为大家推荐一款可以生成markdown的app，bear。这篇文章就是笔者在闲事拿手机写的很好用。最后还要谢谢大家的阅读，笔者也是一个小学生，如果有任何的疑惑欢迎读者联系我提出疑问和见解。





#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

