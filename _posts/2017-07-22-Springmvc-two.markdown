---
layout:     post
title:      "Spring MVC源码分析系列之拦截器的运行流程"
subtitle:   ""
date:       2017-07-22 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Spring MVC
    - Spring
    - 源码分析
---





#### 拦截器的运行流初认识
- 情景一
  笔者创建了一个自定义的拦截器，并在拦截器方法的和核心方法里面打印对应的方法名。运行Tomcat，我们打印出如下内容。
  
```
preHandler()
method()
postHandler()
afterCompletion()
```
- 情景二
笔者创建一个自定义的拦截器，在拦截器的`preHandler()`中将返回值由**true**替换为**false**，运行Tomcat，我们打印出如下内容。
  
```
preHandler()
```
- 情景三
笔者创建两个拦截器，并在拦截器方法的和核心方法里面打印对应的方法名。运行Tomcat，我们打印出如下内容。
  
```
[first]preHandler()
[second]preHandler()
method()
[second]postHandler()
[first]postHandler()
[second]afterCompletion()
[first]afterCompletion()
```
- 情景四
笔者创建两个拦截器，以一个保持不变，在**第二个**拦截器的`preHandler()`中将返回值由**true**替换为**false**，运行Tomcat，我们打印出如下内容。
  
```
[first]preHandler()
[second]preHandler()
[first]afterCompletion()
```




#### 对拦截器源码分析

![java-javascript](/img/in-post/third-mvc/1.png)
<small class="img-hint">445行，通过执行applyPreHandler()方法的返回值进行判断，如果为true则继续往下，如果为false则return不在向下执行。我们点击进入这个方法。</small>
![java-javascript](/img/in-post/third-mvc/2.png)
<small class="img-hint">applyPreHandler()方法对所有的拦截器进行遍历，如果发现拦截器的preHandle()方法返回false的话，则直接执行triggerAfterCompletion()方法，并返回false，运行停止，如果获取的布尔类型为true,则将对interceptorIndex进行赋值，记住这个interceptorIndex，在下面我们还会用到。（根据情景四，如果第二个拦截器preHandler()方法返回值为false的话，此时我们的interceptorIndex值为1（除了自定义的两个拦截器，springmvc自带一个默认的拦截器。），因为我们在执行i=2的时候，返回值为false进入if，直接进入triggerAfterCompletion() 方法，所以不再将i赋值给interceptorIndex）</small>
![java-javascript](/img/in-post/third-mvc/3.png)
<small class="img-hint">从遍历中出来，下一步开始调用目标方法，得到ModelAndView。</small>
![java-javascript](/img/in-post/third-mvc/4.png)
<small class="img-hint">我们在得到ModelAndView之后才去调用我们的applyPostHandle()方法，并且将ModelAndView传入到我们的方法中，也就是说我们可以再拦截器的postHandler()中使用这个ModelAndView(根据情景二和情景四所述我们设置返回值为false，所以在调用applyPostHandle()之前就已经停止了代码的运行，所以运行不到applyPostHandle()这个方法。情景二四的问题也得到一定解决)</small>
![java-javascript](/img/in-post/third-mvc/5.png)
<small class="img-hint">我们进入applyPostHandle()这个方法，同样的也是对拦截器的postHandle()方法进行遍历执行，这里我们只需注意两点，一是我们的i是进行--，所以情景三中先执行拦截器二后执行拦截器一也就可以解释了，二是如上图所述，我们将ModelAndView对象传给拦截器的postHandle()方法，其他的没什么要注意的。</small>
![java-javascript](/img/in-post/third-mvc/6.png)
<small class="img-hint">打断点这一行执行的是渲染视图，我们只要知道渲染视图是在执行完拦截器的postHandle()方法之后进行的就可以。然后我们点击进入processDispatchResult()这个方法。</small>
![java-javascript](/img/in-post/third-mvc/7.png)
<small class="img-hint">在processDispatchResult()这个方法里渲染视图，之后我们开始找到最后面的一行代码，执行triggerAfterCompletion()方法。</small>
![java-javascript](/img/in-post/third-mvc/8.png)
<small class="img-hint">我们点击进入triggerAfterCompletion()这个方法，同样如上面两个方法，这里我们对拦截器的afterCompletion()进行便利执行。这里要注意这时我们的i等于我们执行applyPreHandler()方法里面的interceptorIndex，同样执行--操作，遍历顺序也就是先拦截器二后拦截器一了，我们这里主要说一下情景四，由于在遍历applyPreHandler()方法时我们进入了if，所以我们跳过applyPostHandle()直接进入了triggerAfterCompletion()方法，而就如笔者上面所述，此时我们的interceptorIndex值为1，所以我们就只能遍历笔者自定义的第一个拦截器和springmvc默认创建的拦截器的afterCompletion()方法，由于interceptorIndex不为2所以我们就不执行第二个自定义拦截器的afterCompletion()方法了，四个情景也都解释通了。</small>

#### 对拦截器的总结


我们总结一下拦截器的运行流程，先执行`preHandle()`方法，进行拦截器的初始化，若返回值为 true, 则继续调用后续的拦截器和目标方法，若返回值为 false, 则不会再调用后续的拦截器和目标方法。之后我们开始调用``目标方法``再然后调用拦截器的``postHandle()``方法，在这个方法中我们可以获得ModelAndView等对象，我们可以在这里对请求域中的属性或视图做出修改，然后进行视图的渲染，最后执行拦截器的``afterCompletion()``方法对资源进行释放，这就是拦截器的执行的全过程。


#### 笔者说


由此，通过对源代码的分析我们清楚地认识到了拦截器是什么，它在不同情景下的运行流程，为我们更好地使用Springmvc拦截器奠定了基础。最后还要谢谢大家的阅读，笔者也是一个小学生，如果有任何的疑惑欢迎读者联系我提出疑问和见解。




#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

