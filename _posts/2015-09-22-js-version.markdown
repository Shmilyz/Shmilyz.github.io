---
layout:     post
title:      "Spring MVC运行流程中关于postHandler与调用HandlerExceptionHandler之间的关系"
subtitle:   ""
date:       2017-07-16
author:     "田俊哲"
header-img: "img/post-bg-js-version.jpg"
tags:
    - Spring MVC
    - JavaScript
    - Spring
    -	解决问题
---

#### 简述SpringMVC的运行流程

我们先简单的对SpringMVC的运行流程进行一些简单的回顾。

首先用户发送请求后有我们的SpringDispatcherServlet进行拦截，然后交由处理映射器去检查我们在SpringMVC中是否存在对应的映射，如果不存在我们则去查看我们是否配置了default-servlet-handler，没有控制台报错，并显示404页面，如果存在目标资源，则显示目标资源，同样
不存在则显示404页面，另一个分支，当我们SpringMVC存在对应的映射后，我们由HandlerMapping获取到HandlerExecutionChain处理器调用链，在这个处理器调用链中存在两个东西，一是我们相关的interceptors拦截器，二是我们的handler（目标处理器）以及目标方法。获取到我们
这个处理器调用链后，我们去获取HandlerAdapter对象，这个适配器的作用，是可以对我们的提交表单中的内容进行校验，更换格式，格式化处理，过滤等相关工作，再然后调用我们拦截器的PreHandle方法，在之后获取我们的ModelAndView对象（不论我们用的是map或者model等最后都
会转化成一个ModelAndView类型的对象），得到对象后，我们就可以调用拦截器的postHandle方法了（注:这个过程的细节正式我下面要说的），如果目标方法存在异常则由HandlerExceptionHandler组件处理异常，得到新的ModelAndView对象，最后我们将ModelAndView对象转换成实际的View,渲染视图，调用拦截器的afterCompletion方法释放后这个过程就结束了了。

> 译者注：添加<mvc:default-servlet-handler/>的同时不要忘记添加 <mvc:annotation-driven/>,作用是如果映射过 Springmvc会找到对应的映射方法，如果没有映射则帮你去找相关资源。


#### 问题发现

我们都知道在运行的过程中我们可能会发生运行期异常，所以我们可以定制错误页面，当我们无法通过代码避免未知错误进而发生运行期异常时我们可以转向到我们的错误页面，笔者在目标方法中用（10/表单获取的值）后发生了运行期异常，这个异常精确地说叫ArithmeticException，
这时我们跳转到了我们自己定制的错误页面，但是笔者发现笔者自定义的拦截器只执行了PreHandle方法和最后的afterCompletion方法并没有执行postHandle方法，就这个疑问我们队源码进行了Debug,分别在表单中输入0和2，得到我们的对应截图。

> 译者注：一定要注意运行期异常和编译期异常的不同，不要混淆！

![java-javascript](/img/in-post/first-handler/1.png)
<small class="img-hint">我们先打上断点</small>
![java-javascript](/img/in-post/first-handler/2.png)
<small class="img-hint">我们执行到我们打断点的这一行（笔者表单输入0）</small>
![java-javascript](/img/in-post/first-handler/3.png)
<small class="img-hint">我们看到由于我们的目标方法报错，所以我们执行了catch中的内容，跳出了try，也就没有执行将拦截器postHandle方法的for循环那一行，获取到的运行期异常信息进行赋值，同时mv为null（笔者表单输入0）</small>
![java-javascript](/img/in-post/first-handler/4.png)
<small class="img-hint">执行方法，传入多个参数，我们可以看到我们的错误信息java.lang.ArithmeticException:/by zero，说明这个错误信息传入到了下一个方法（SpringMVC会默认将错误信息存入key为exception的map中）（笔者表单输入0）</small>
![java-javascript](/img/in-post/first-handler/5.png)
<small class="img-hint">我们执行到我们打断点的这一行（笔者表单输入2）</small>
![java-javascript](/img/in-post/first-handler/6.png)
<small class="img-hint">由于目标方法成功执行，所以我们执行图中所示方法，我们可以看到此时我们获得到了转化后的ModelAndView对象（笔者表单输入2）</small>
![java-javascript](/img/in-post/first-handler/7.png)
<small class="img-hint">从这行我们看出我们要开始对拦截器postHandle方法的for循环（笔者表单输入2）</small>



#### 解决问题

我们通过这次Debug了解到了Spring MVC运行流程中关于postHandler与调用HandlerExceptionHandler之间的关系，对笔者了解Spring MVC运行流程产生很大的帮助，Spring MVC运行流程越来越清楚。在自定义拦截器的时候我们要考虑稳重相关内容。

#### 著作权声明


作者 [田俊哲](http://weibo.com/huxpro)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

