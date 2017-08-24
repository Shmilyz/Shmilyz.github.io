---
layout:     post
title:      "带你了解DispatcherServlet的url-pattern配置-第一讲"
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

2. `/` 将会覆盖容器的default servlet，会将除`.jsp`和`*.jspx`以外的所有请求全部拦截，包括静态资源的请求。我们虽然可以通过第一种方法拦截我们想拦截的后缀结尾的请求，但无法实现当下统一的rust风格，但第二种通过该配置是可以实现rustful风格的URL的。

3. `/*` 这种请求会拦截所有的请求包括`.jsp`和`*.jspx`，我们常常在配置`HiddenHttpMethodFilter`和配置字符编码过滤器`CharacterEncodingFilter`的时候通常要将url-pattern配置为`/*`,但对于DispatcherServlet配置来说，这是一种错误的配置，我们可以通过请求找到对应的映射，并返回一个`.jsp`请求，去请求这个`.jsp`视图，但如果我们设置为了 `/*`DispatcherServlet将会拦截这个请求，并继续去找对应的映射，然而我们不会找到对应的映射，进而报错。


#### 验证url-pattern配置为 ‘/*’, 为什么是错误的

我们进行一个极其简单的测试，我们用controller进行一个转发，并在网页上测试请求路径/test

```java
@Controller
@RequestMapping("")
public class Test {
    @RequestMapping("/test")
    public String test() {
        return "test";
    }
}
```

执行结果得到404页面，并在控制台打印如下的错误警告日志:

```
DispatcherServlet with name ‘ ’ processing GET request for [/test] 
RequestMappingHandlerMapping]-[DEBUG] Looking up handler method for path /test 
RequestMappingHandlerMapping]-[DEBUG] Returning handler method [public java.lang.String com.taotao.sso.controller.Test.test()] 
DispatcherServlet]-[DEBUG] Rendering view [org.springframework.web.servlet.view.InternalResourceView: name ‘test’; URL [/WEB-INF/jsp/test.jsp]] in DispatcherServlet with name ‘ ’ 
InternalResourceView]-[DEBUG] Forwarding to resource [/WEB-INF/jsp/test.jsp] in InternalResourceView ‘test’ 
DispatcherServlet]-[DEBUG] DispatcherServlet with name ‘ ’ processing GET request for [/WEB-INF/jsp/test.jsp] 
RequestMappingHandlerMapping]-[DEBUG] Looking up handler method for path /WEB-INF/jsp/test.jsp 
RequestMappingHandlerMapping]-[DEBUG] Did not find handler method for [/WEB-INF/jsp/test.jsp] 
PageNotFound]-[WARN] No mapping found for HTTP request with URI [/WEB-INF/jsp/test.jsp] in DispatcherServlet with name ‘ ’ 
DispatcherServlet]-[DEBUG] Successfully completed request
```

通过阅读日志我们发现，我们第一次的请求可以找到对应的映射，紧接着springmvc转发请求’/WEB-INF/jsp/test.jsp’，但这里注意，由于我们这里进行错误测试，将url-pattern配置为 ‘/*’，所以发出的请求/WEB-INF/jsp/test.jsp 同样被DispatcherServlet拦截并处理，希望可以找到对应的映射handler，但我们并没有对该请求进行映射，自然会找不到handler, No mapping. 所以这样的请求会一直为404.并在控制台发送错误警告。


> 作者注：根据servlet的规定，如果我们设置为(/*)则会优先于扩展匹配(*.jsp, *.jspx)后缀的文件,所以不难解释为什么jsp的请求也被DispatcherServlet拦截掉了。



#### 将url-pattern配置为 ‘/’，不拦截*.jsp，但为什么依然可以得到对应的*.jsp页面呢？


这里有一个比较细节的地方要讲，我们将url-pattern配置为 ‘/’之后，我们不在拦截(*.jsp, *.jspx)后缀的文件，但是当Springmvc转发刚刚的请求’/WEB-INF/jsp/test.jsp’，我们依然得到了渲染成功得test.jsp页面，证明虽然DispatcherServlet不在处理拦截(*.jsp, *.jspx)后缀的文件，但依然会有别的Servlet拦截了该请求，这里我们无法从我们项目的web.xml文件中查看到该Servlet，这些默认的Servlet存在于我们的TOMCAT中。

![java-javascript](/img/in-post/DispatcherServlet/1.png)

![java-javascript](/img/in-post/DispatcherServlet/2.png)


图一中的JspServlet会处理后缀为(*.jsp, *.jspx)的文件，这也就解释了为什么我们可以得到对应页面了，我们可以自己做一个小实验，由于在WEB-INF的jsp无法直接诶访问，我们在WEB-INF外写一个简单的jsp文件，并发送该请求，我们可以直接得到对应的页面，这就是JspServlet的作用。


而我们的图二将和我们的第二讲有一定的联系，对于Springmvc我们如果想访问静态资源是要通过配置"mvc:default-servlethandler"，我们通过这种方式解决静态资源访问问题。我们这里先了解一下，我们将在第二讲研究这个延伸出来的问题。





#### 结论

通过该篇文章的讲解，我想大家对DispatcherServlet的url-pattern配置也不再那么模糊了，而第二讲我们将重点讨论Springmvc中是如何处理静态变量的，这也是我们延伸出来的问题。






#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

