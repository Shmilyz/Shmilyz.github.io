---
layout:     post
title:      "Mybatis源码分析第九讲之Mybatis两次包装分析"
subtitle:   ""
date:       2017-09-07 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第九讲之Mybatis两次包装分析


在对mybatis进行源码分析的过程中，我们遇到了两个方法，其目的都是一样的，就是将我们传入的值进行包装，这两个方法在mybatis中只是两个很简单的方法，但其实这两个方法也很值得我们去分析以及在未来中进行套用的。本篇的讲解参考了部分网上资源，但并无任何抄袭。



#### 赋值回顾


当我们传入的值为一个的情况下
1.如果我们是传入的是String，基本类型，我们的写法。
“#{正常要与我们传入的属性名一致，但其实在这里我们写任何的字符都可以，原因后面讲。}“

2.如果传入的值为多个，并且没有@Param()注解，但多值都为基本类型。
”#{我们按照规定按照param1，param2，param3的方式去写}“

3.2.如果传入的值为多个，并且没有@Param()注解，但多值有我们自定义类。
”#{我们按照规定按照param1，param2，param3的方式去写，对应的类按照param.类里面属性的方式去写}“

4.如果传入的值为多个，但每个属性有@Param()注解
”#{按照注解的内容去写，或者按照param的方式去写}“

5.传入的值为list，collection或者array
”#{写法为list【0】 collection【0】 array【0】}“

我们下面按照源码来为大家梳理一下mybatis是如何对我们传入的参数进行封装的。


#### 源码分析


![java-javascript](/img/in-post/others-mybatis/1.png)
首先我们看一下我们getNamedParams()方法传入的值，传入的是一个数组，不论我们传入的值为一个还是多个都先转成一个数组args。
![java-javascript](/img/in-post/others-mybatis/2.png)
一上来我们先通过names.size();获取到一个paramCount，而上图则是names的获取方式，mybatis先判断是否使用了注解，如果使用了则获取这个注解的值，赋值给name，并按照{0=获取的name，1=获取的name}这种方式存储，如果没有注解则按照{0=0,1=1...}这种方式进行存储map，我们发现map的共同点都是以{key:参数索引，value:参数name值}得方式进行存储，而我们发现我们有个相同的判断条件，都是当没有使用注解时的情况，而第一个要进行config.isUseActualParamName()判断，这是去获取全局配置，有一个全局配置useActualParamName，它可以将我们的方法的参数名作为我们这里的name，默认为true，但是要求我们的jdk为1.8以上。
![java-javascript](/img/in-post/others-mybatis/5.png)
我们在得到paramCount和names之后，我们要进行判断，如果我们的args为空，我们则直接返回null
![java-javascript](/img/in-post/others-mybatis/4.png)
但如果我们的args中有一个值，也就是我们刚刚获取的paramCount值为1，并且没有使用我们的param注解，则就直接从我们的数组args中拿到第一个元素args【0】然后返回出去。所以也就解释了为什么当我们传入的值只有一个的时候，我们在#{}中可以随意写，因为mybatis对于一个属性并不会封装成map。
![java-javascript](/img/in-post/others-mybatis/3.png)
当我们的args为多元素，或者单个元素却使用了param注解，我们则进入如图所示的方法进行执行，我们先创建了一个map为param，最后我们返回这个param。我们看一看我们如何封装这个param的。
![java-javascript](/img/in-post/others-mybatis/6.png)
整个封装方法的核心来了，这句话很重要，我们把我们刚刚的names进行遍历，将遍历出来的第一个name的value作为param的key放入，而 args【entry.getKey()】作为value来存储，我们都知道我们每一个name也就是entry的key都是以0123456这些数字的形式进行存储的，那也就相当于param的value按照args【0】args【1】args【2】来进行遍历。这就是整个封装的核心所在。
![java-javascript](/img/in-post/others-mybatis/7.png)
这一步算是mybatis为我们提供的标准书写方法了，我们除了要将我们自己定义的name作为key以外，mybatis还要按照它自己的标准设置key，他的标准就是我们前面提到的param1，param2，最后相当于最后返回了两套标准。我们无论写没写注解，或者有没有设置全局变量，我们都可以按照mybatis那套规范去写。


上面是我要讲的第一个封装，而我们还有第二个封装，是对我们的list，collection，数组的封装。


![java-javascript](/img/in-post/others-mybatis/8.png)
由于我们之前讲过第一次封装，这里我们就可以很清楚的看明白第二次封装的源码了，但这里注意，我们只会对list，collection，数组进行封装，但不会对map进行封装，剩下的也没有什么要注意的了，这些源码很好的解释了为什么当我们的属性为list，collection，数组的时候不能直接使用属性名，而要用list，collection，数组来写，因为mybatis对list，collection，数组的封装就是要用list，collection和array。还有一个小点，如果我们是一个list那么我们就即可使用collection也可使用list了。其他的没什么了。





#### 总结


就此，我们真正的对mybatis进行数据库交互这部分代码有了完整的认识，但并没有结束，不知道大家是否发现我们没有还没有对四大对象的拦截器这部分操作进行任何的源码分析，而下一讲我们除了要进行总结外，我们对拦截器这部分源码要进行一个讲解，为我们的插件开发做铺垫。



#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

