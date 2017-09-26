---
layout:     post
title:      "大话java线程池"
subtitle:   ""
date:       2017-09-26 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - 线程
    - java
---


	


#### 大话java线程池之前言



在讲解线程池之前 我们通过一个小故事来了解一下线程池的含义，假设有一个工厂，工厂里面一共有十个工人，每个工人只能做一件事情，这个时候老板开始给他们分配任务，慢慢的老板把所有工人都分配上了任务，这时候老板又接了一单活，但是工人还在做他们手头的任务，没法做别的事情了，一心不可二用阿，这时候老板就会把接到的活进行排序，但是老板的野心很大，接的任务速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招5个临时工进来帮忙，但是如果十四个人依然无法满足老板的需要，做工的速度还是不够，老板就会烦死，考虑不再接受新的任务了或者选择抛弃一些前面的任务，当十四个工人中有人空下来了，并且老板接活的速度比较慢了工厂主管就可以考虑辞掉四个临时工，只留下原来的那十个人。这是我理解的线程池。



讲线程池我们将分两个方向去讲，ThreadPoolExecutor类和Executors类。其中ThreadPoolExecutor是Executors类的底层实现。所以我们还是要简单的讲一下ThreadPoolExecutor类。然后我们再去细讲Executors类的方法。

#### 大话java线程池之内容

- ThreadPoolExecutor类

ThreadPoolExecutor是线程池中最核心的一个实现类，如果我们要了解线程池，我们也要了解这个类，以及构造方法中的属性。这有助于我们优化线程池的配置。使线程池达到我们理想中的最优化。

下面我们来解释一下构造器中的几个参数。

1.`corePoolSize`:这个属性相当于刚才例子中的十个工人，我们创建的线程池里面，一开始是没有线程的，等到任务来临的时候就会创建一个线程来操作任务，当一个一个任务，就会创建一个一个的线程，直到创建的线程等于corePoolSize为止，则不会在创建新的线程了，也就相当于例子中工厂正式工只有十个人。

2.`maximumPoolSize`:相当于上面例子的十四个人，指线程最大的线程数，当任务过多的时候，我们就开始招临时工创建新的线程了，而工厂招的临时工加正式工不能大于maximumPoolSize

3.`keepAliveTime`:这个属性相当于辞退临时工的时间，在默认情况下，只有线程池的数量大于corePoolSize的时候，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。如果我们的临时工没有活干，老板当然不会养闲人了，肯定会辞退他们，但是老板并不会将经验丰富的正式工辞退。

4.`unit`:参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：
	```
	TimeUnit.DAYS;               //天
	TimeUnit.HOURS;             //小时
	TimeUnit.MINUTES;           //分钟
	TimeUnit.SECONDS;           //秒
	TimeUnit.MILLISECONDS;      //毫秒
	TimeUnit.MICROSECONDS;      //微妙
	TimeUnit.NANOSECONDS;       //纳秒
```

5.`workQueue`:相当于例子中，我们的十个工人都在忙各自的活，但还有新的订单老板只能将订单进行排列，等工人完成之后再让他们去干新的任务。那么这些等待任务就要对他们进行排序，而我们要设置他们的执行方式。
  ```
  ArrayBlockingQueue：基于数组结构的有界阻塞队列，先进先出；
  LinkedBlockingQuene：基于链表结构的阻塞队列，先进先出，吞吐量通常要高于ArrayBlockingQuene；
  SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；
	```
	
6.`threadFactory`:创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。

7.`handler`:还是拿上面的例子，当老板发现哪怕招了四个临时工依然解决不了订单阻塞无法执行的问题，那么解决办法就是将订单遗弃或者抛出异常等。线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了四种对应策略。
	```
	AbortPolicy：直接抛出异常，这是默认的策略；
	CallerRunsPolicy：用调用者所在的线程来执行任务， 它既不会丢弃任务，也不会抛出任何异常，它会把任务推回到调用者那里去,以此缓解任务流，它直接在 execute 方法的调用线程中运行被拒绝的任务；
	DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
	DiscardPolicy：直接丢弃任务；
	```




![java-javascript](/img/in-post/seven-mybatis/22.png)


#### 大话java线程池之延伸



#### 大话java线程池之总结


就此，我们真正的对mybatis进行数据库交互这部分代码有了完整的认识，但并没有结束，不知道大家是否发现我们没有还没有对四大对象的拦截器这部分操作进行任何的源码分析，而下一讲我们除了要进行总结外，我们对拦截器这部分源码要进行一个讲解，为我们的插件开发做铺垫。



#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

