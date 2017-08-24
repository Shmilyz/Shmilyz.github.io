---
layout:     post
title:      "带你了解DispatcherServlet的url-pattern配置-第二讲"
subtitle:   ""
date:       2017-08-25 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Spring MVC
---


	


#### 前言

上一节我们重点研究了关于DispatcherServlet的url-pattern配置，而这一讲我们将延伸上一节课的一个小问题，我们将探究"mvc:default-servlethandler"这句配置的执行流程。



> 作者注：观点仅代表个人观点，如有错误，欢迎指点。


#### "mvc:default-servlethandler"的作用

通过上节课的讲解，我们应该不难理解，如果我们想要发送除*.jsp以外的请求时我们的DispatcherServlet将会对这些请求进行拦截，同样的如果我们想要访问我们的静态资源文件(图片，jQuery文件，bootstrap文件等都是静态资源文件)DispatcherServlet依然会拦截下来这些请求，并找对应的映射，但我们并不会写这些静态资源的映射，所以我们现在要做的是如何不让DispatcherServlet对我们这些静态资源文件进行拦截，而我们通过设置一行简单的 ” mvc:default-servlethandler“就可以不再让DispatcherServlet进行拦截了。


#### "mvc:default-servlethandler"的执行流程

"<mvc:default-servlet-handler/>" 将在 SpringMVC 上下文中定义一个DefaultServletHttpRequestHandler，它会对进入 DispatcherServlet 的
请求进行筛查，如果发现是没有经过映射的请求，就将该请求交由web服务器，我们这里是Tomcat默认的 Servlet 处理（这里就很好的解决了我们上节课的问题，所有没被映射过的请求和静态资源请求等都将交由下图中的DefaultServlet来进行处理），如果我们自己映射过该请求，才交给DispatcherServlet 进行处理。

我们可以做一个小实验，例子依然用上一讲的例子，我们请求“/test”，而我们通过控制台日志得到了Springmvc找到“/test”请求的映射，并且也转发显示了对应的页面，但如果我们在“/test”后面加几个字母“/testzhangmeng”，并进行请求，我们知道我并没有写相关映射，而结果就是得到一个404页面，但这里控制台Springmvc并没有得到相关错误警告的日志，证明我们这个苏编写的请求不再有Springmvc来管理了，而是交给了下图中的DefaultServlet了。


![java-javascript](/img/in-post/DispatcherServlet/2.png)


#### 结论

通过两篇文章的讲解，我们对Springmvc中关于Servlet这一块的知识有了清楚地认识。






#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

