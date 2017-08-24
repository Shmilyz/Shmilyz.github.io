---
layout:     post
title:      "带你了解DispatcherServlet的url-pattern配置"
subtitle:   ""
date:       2017-08-24 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Spring MVC
---


	


#### 前言

对于springmvc中前端控制器url-pattern配置,我发现有很多人并不知道原理或者带着错误的观念去理解，比如会遇到`/`和`/*`不知道两者区别，或者并不清楚各自都会拦截何种请求，另一个问题，为什么我们直接访问不在WEB-INF目录下的jsp, 可以直接找到并解析的原因。而本篇文章将一一为大家解析。让这些小细节也能引起大家的关注。



> 作者注：观点仅代表个人观点，如有错误，欢迎指点。


#### DispatcherServlet常见的配置

我们每次在写web.xml关于Springmvc这部分拦截配置的时候，我们常常会进行如下操作。

```java
 <!-- 配置 SpringMVC 的 DispatcherServlet -->
  <!-- The frcom.shmilyz.springmvc.controllerller of this Spring Web application, responsible for handling all application requests -->
  <servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc-config.xml
        classpath:applicationContext.xml
      </param-value>
      <!--默认的配置文件为：/WEB-INF/<servlet-name>-servlet.xml-->
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!-- Map all requests to the DispatcherServlet for handling -->

  <servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

#### 常见的三种url-pattern配置

1. `*.xxx` 以指定后缀结尾的请求都交由DispatcherServlet处理。

2. `/` 会将除`.jsp`和`*.jspx`以外的所有请求全部拦截，包括静态资源的请求。我们虽然可以通过第一种方法拦截我们想拦截的后缀结尾的请求，但无法实现当下统一的rust风格，但第二种通过该配置是可以实现rustful风格的URL的。

3. `/*` 这种请求会拦截所有的请求包括`.jsp`和`*.jspx`，我们常常在配置`HiddenHttpMethodFilter`和配置字符编码过滤器`CharacterEncodingFilter`的时候通常要将url-pattern配置为`/*`,但对于DispatcherServlet配置来说，这是一种错误的配置，我们可以通过请求找到对应的映射，并返回一个`.jsp`请求，去请求这个`.jsp`视图，但如果我们设置为了 `/*`DispatcherServlet将会拦截这个请求，并继续去找对应的映射，然而我们不会找到对应的映射，进而报错。





后续待完成。。。


#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

