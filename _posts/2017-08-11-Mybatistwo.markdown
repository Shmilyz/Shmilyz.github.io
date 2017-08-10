---
layout:     post
title:      "Mybatis源码分析第而讲之SQLSessionFactory的初始化"
subtitle:   ""
date:       2017-08-11 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - Mybatis
    - 源码分析
---


	


#### Mybatis源码分析第而讲之SQLSessionFactory的初始化

![java-javascript](/img/in-post/first-mybatis/1.png)
![java-javascript](/img/in-post/first-mybatis/2.png)
我们在这一行打上断点，我们探究如何获得sqlSessionFactory对象,首先我们进去build()方法。
![java-javascript](/img/in-post/first-mybatis/3.png)
我们首先得到一个XMLConfigBuilder对象parser，从字面理解它是一个xml的解析器，我们看方法里面传入了输入流等对象，我们看build(parser.parse())这个方法都执行了什么。
![java-javascript](/img/in-post/first-mybatis/4.png)
这一步往 parseConfiguration()对象里面传入一个parser.evalNode("/configuration")，而这个就是要我们的parser对象解析出xml文件的"/configuration"这个节点，而我们都知道"/configuration"节点，是Mybatis配置文件中最大的一个节点标签。我们接着进入parseConfiguration()
![java-javascript](/img/in-post/first-mybatis/5.png)
![java-javascript](/img/in-post/first-mybatis/6.png)
![java-javascript](/img/in-post/first-mybatis/7.png)
我们发现我们进入的这个方法跟刚才一样，继续解析这些标签，有我们熟悉的setting等等。挨个的解析，我们看到我们将setting节点的内容保存到一个Properties对象中，紧接着将这个对象传入 settingsElement()方法中，而我们进入这个方法发现这个方法里面又对setting这个标签的子标签进行解析，包括后面还写有默认值。而且我们看到所有解析完的值都传入了configuration这个对象中，这个configuration很重要，现在已知的是全局配置文件中的内容都解析到了configuration对象中。
![java-javascript](/img/in-post/first-mybatis/8.png)
由于mappers涉及到另一个xml文件，所以我们进入这个对mapper解析的方法中。
![java-javascript](/img/in-post/first-mybatis/9.png)
![java-javascript](/img/in-post/first-mybatis/10.png)
首先先判断我们在mappers中使用的是普通的mapper标签还是package标签或者其他的，我们进入到了resource这个标签判断中，因为作者xml文件中就是用resource指定的mapper位置，我们发现这里我们又生成了一个mapperParser对象，而这个对象的作用跟前面的一样，解析我们所有的mapper文件中的一个个标签，解析过程则是执行mapperParser.parse();这个方法。我们进入这个方法。
![java-javascript](/img/in-post/first-mybatis/11.png)
![java-javascript](/img/in-post/first-mybatis/12.png)
我们进入到这个核心解析方法configurationElement()中，在这里拿到了mapper文件的每一个标签。
![java-javascript](/img/in-post/first-mybatis/13.png)
![java-javascript](/img/in-post/first-mybatis/14.png)
 我们进入方法我们发现又新建了一个statementParser对象，这又是一个解析器，而目的就是解析一个一个的增删改查标签的，我们点击parseStatementNode()这个解析标签方法。
![java-javascript](/img/in-post/first-mybatis/15.png)
![java-javascript](/img/in-post/first-mybatis/16.png)
我们进入这个方法，我们看到了与前面相似的代码，不一样的是每个不同标签以及标签里面可选设置项的名称，我们将所有信息一一解析，并装入addMappedStatement()这个方法中。
![java-javascript](/img/in-post/first-mybatis/17.png)
我们发现，解析出来的一个个增删改查标签，并封装成一个MappedStatement对象，一个MappedStatement就代表一个增删改查标签的详细信息。而且这个对象放入了configuration中。
![java-javascript](/img/in-post/first-mybatis/18.png)
![java-javascript](/img/in-post/first-mybatis/19.png)
我们点开这个statement看一眼这里面都装了什么。我们发现里面有我们的id值我们要返回的类型还有我们的sql语句等等属性。这个对象很重要，我门后面要用到。
![java-javascript](/img/in-post/first-mybatis/20.png)
最后返回我们的configuration，在这个对象中我们看到有好多的属性其中有两个我们要理解的属性。
![java-javascript](/img/in-post/first-mybatis/21.png)
configuration中的一个属性mappedStatements这个属性中装载了每一个接口类以及对应的statement，用KV的方式进行保存，这也就能想明白我们为何通过调用方法用可以得到我们xml文件中的sql语句及其他属性了。
![java-javascript](/img/in-post/first-mybatis/22.png)
而另一个比较重要的属性就是mapperRegistry了，同样通过KV进行存储，key是我们的每一个接口类，而value则是我们的MapperProxyFactory对象，我们都知道，最后生成的实体类对象是一个被封装的代理对象，所以在这里会通过MapperProxyFactory生成对应的代理对象。
![java-javascript](/img/in-post/first-mybatis/23.png)
最后我们得到了我们想要的sqlSessionFactory对象，而sqlSessionFactory里面唯一的属性就是这个configuration。






#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

