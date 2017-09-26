---
layout:     post
title:      "大话java系列之线程池"
subtitle:   ""
date:       2017-09-26 12:00:00
author:     "田俊哲"
header-img: "img/post-bg-digital-native.jpg"
tags:
    - 线程
    - java
---


	


#### 大话java系列之线程池之前言



在讲解线程池之前 我们通过一个小故事来了解一下线程池的含义，假设有一个工厂，工厂里面一共有十个工人，每个工人只能做一件事情，这个时候老板开始给他们分配任务，慢慢的老板把所有工人都分配上了任务，这时候老板又接了一单活，但是工人还在做他们手头的任务，没法做别的事情了，一心不可二用阿，这时候老板就会把接到的活进行排序，但是老板的野心很大，接的任务速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招5个临时工进来帮忙，但是如果十四个人依然无法满足老板的需要，做工的速度还是不够，老板就会烦死，考虑不再接受新的任务了或者选择抛弃一些前面的任务，当十四个工人中有人空下来了，并且老板接活的速度比较慢了工厂主管就可以考虑辞掉四个临时工，只留下原来的那十个人。这是我理解的线程池。



讲线程池我们将分两个方向去讲，ThreadPoolExecutor类和Executors类。其中ThreadPoolExecutor是Executors类的底层实现。所以我们还是要简单的讲一下ThreadPoolExecutor类。然后我们再去细讲Executors类的方法。

#### 大话java系列之线程池之内容



- ThreadPoolExecutor类

ThreadPoolExecutor是线程池中最核心的一个实现类，如果我们要了解线程池，我们也要了解这个类，以及构造方法中的属性。这有助于我们优化线程池的配置。使线程池达到我们理想中的最优化。

下面我们来解释一下构造器中的几个参数。

1.`corePoolSize`:这个属性相当于刚才例子中的十个工人，我们创建的线程池里面，一开始是没有线程的，等到任务来临的时候就会创建一个线程来操作任务，当一个一个任务，就会创建一个一个的线程，直到创建的线程等于corePoolSize为止，则不会在创建新的线程了，也就相当于例子中工厂正式工只有十个人。

2.`maximumPoolSize`:相当于上面例子的十四个人，指线程最大的线程数，当任务过多的时候，我们就开始招临时工创建新的线程了，而工厂招的临时工加正式工不能大于maximumPoolSize

3.`keepAliveTime`:这个属性相当于辞退临时工的时间，在默认情况下，只有线程池的数量大于corePoolSize的时候，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。如果我们的临时工没有活干，老板当然不会养闲人了，肯定会辞退他们，但是老板并不会将经验丰富的正式工辞退。

4.`unit`:参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：
	
	TimeUnit.DAYS;               //天
	
	TimeUnit.HOURS;             //小时
	
	TimeUnit.MINUTES;           //分钟
	
	TimeUnit.SECONDS;           //秒
	
	TimeUnit.MILLISECONDS;      //毫秒
	
	TimeUnit.MICROSECONDS;      //微妙
	
	TimeUnit.NANOSECONDS;       //纳秒


5.`workQueue`:相当于例子中，我们的十个工人都在忙各自的活，但还有新的订单老板只能将订单进行排列，等工人完成之后再让他们去干新的任务。那么这些等待任务就要对他们进行排序，而我们要设置他们的执行方式。
  
	ArrayBlockingQueue：基于数组结构的有界阻塞队列，先进先出；

  LinkedBlockingQuene：基于链表结构的阻塞队列，先进先出，吞吐量通常要高于ArrayBlockingQuene；
  
  SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；

6.`threadFactory`:创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。

7.`handler`:还是拿上面的例子，当老板发现哪怕招了四个临时工依然解决不了订单阻塞无法执行的问题，那么解决办法就是将订单遗弃或者抛出异常等。线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了四种对应策略。
	AbortPolicy：直接抛出异常，这是默认的策略；
	
	CallerRunsPolicy：用调用者所在的线程来执行任务， 它既不会丢弃任务，也不会抛出任何异常，它会把任务推回到调用者那里去,以此缓解任务流，它直接在 execute 方法的调用线程中运行被拒绝的任务；
	
	DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
	
	DiscardPolicy：直接丢弃任务；
	
	
	
	
	

- Exectors类


我们刚才说ThreadPoolExecutor类是Exectors类的底层实现，而Exectors类为我们提供了几种创建策略，下面我们来介绍一下这些策略方法。


![java-javascript](/img/in-post/threadpool/2.png)

`newFixedThreadPool`

创建了一个可重用的具有固定线程的线程池，初始化一个指定线程数的线程池，我们设置corePoolSize == maximumPoolSize，如图所示，并且我们使用LinkedBlockingQuen作为阻塞队列的策略，先进先出的链表结构。由于我们的corePoolSize == maximumPoolSize所以我们并不存在释放临时工。

![java-javascript](/img/in-post/threadpool/1.png)

`newCachedThreadPool`

这里的corePoolSize 为0，maximumPoolSize值为Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列策略，SynchronousQueue是一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，也就是说我们插入一个任务就必须有一个线程过来移除这个任务，相当于老板拿到一个任务，一个员工过来就要接收这个任务然后去执行，反过来工人找老板去哪一个任务，老板手上的任务不能留在老板手上。

我们比较一下newFixedThreadPool和newCachedThreadPool，我们的newCachedThreadPool在没有任务执行时，当线程的空闲时间超过了keepAliveTime，会自动释放线程资源，这会导致一定的系统开销，由于我们设置的核心线程池数量为0，所以我们所有的线程(临时工)都会被释放解雇，我们在使用线程池的时候我们就要从新建立新的线程，这也导致了一定的系统开销。

![java-javascript](/img/in-post/threadpool/3.png)

`newSingleThreadExecutor`

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行，内部使用LinkedBlockingQueue作为阻塞队列。

![java-javascript](/img/in-post/threadpool/4.png)

`newScheduledThreadPool`

初始化的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期的同步数据。

现在我们知道了该如何创建ExecutorService对象，但我们还不知道如何将我们想要做的事情，也就是我们的runnable对象一个一个传入我们的线程池，以及线程池的关闭等。


`Executor.execute(Runnable runnable)`

![java-javascript](/img/in-post/threadpool/7.png)

我们通过这种方来提交任务，但我们要传入我们的runnable对象，注意这里并没有获取返回值，无法判断任务是否成功执行。

`ExecutorService.submit()`

![java-javascript](/img/in-post/threadpool/8.png)

通过这种方式提交任务，可以获取任务执行的返回值。

下面我们通过一个个线程池模板为大家讲一下线程池的使用及深入了解几种策略的运行方式。



#### 大话java系列之线程池之例子



![java-javascript](/img/in-post/threadpool/5.png)
![java-javascript](/img/in-post/threadpool/6.png)

我们在这里调用了newFixedThreadPool方法，方法体里的参数要求我们写入corePoolSize的值。我们这里的核心线程池设置为5，紧接着我们通过匿名内部类实现的一个runnable对象，我们通过submit()方式提交我们的任务给我们的线程池，可以看到我们的运行结果，由于我们使用的是newFixedThreadPool策略，该策略可以创建缓存指定个数的线程，我们的提交两个任务，所以线程池创建出两个线程为我们的任务提供服务。最后我们看结果，线程一二的确立并执行了我们提交的两个任务，两个线程之间的不同步的。

![java-javascript](/img/in-post/threadpool/9.png)
![java-javascript](/img/in-post/threadpool/10.png)

我们这里调用了newCachedThreadPool方法，在这里我为了让大家更清楚的知道newCachedThreadPool的策略，我把这个方法的底层实现拿到了我的main方法，其实就是new了一个ThreadPoolExecutor。该策略的含义实现是将线程池中的核心线程数设置为0，虽然正式工为0个，但我们可以设置我们的临时工数量为Interger的最大值,并且排队使用的策略类似于管道模式，来一个走一个。所以我们可以创建一个一个的临时工去接活，老板手上不能有没人接的活。而且这种策略最为重要的特点就是设置了临时工解雇时间，这里设置的是60秒。所以我这里模拟了一下执行流程，首先临时工线程一个一个的接任务，任务完成之后，我们让主线程休息一定时间，这个时间要大于临时工被解雇的时间，休息过后我们再看看线程池中的数据，最后我们发现，线程池里的临时工线程为0，我们的临时工线程被解雇了。

![java-javascript](/img/in-post/threadpool/11.png)
![java-javascript](/img/in-post/threadpool/12.png)

我们这里调用newSingleThreadExecutor，该方法的特点是老板定的规矩，这个企业里面最多只能有一个人，并且这个人是正式工，被设定好的了，无法改变，一个任务来了就只能让这个正式工去做，多余的任务都排列好了放在老板手上，等这个正式工做完了，再让他去做下一件事。所以我们可以通过我写的代码的运行结果查看到我提交的三个任务最后全部由一个线程按顺序完成的。

![java-javascript](/img/in-post/threadpool/13.png)

该方法规定我们可以通过是用该线程池定期的同步数据，我们在一定周期内循环执行所提交的任务。具体的写法看我的上图例子就好，我们可以设置线程的延迟执行和线程的延迟周期性执行。

这样关于Exectors类的几种方法策略就讲解完了，最后我们要延伸出来两个方法，分别是invokeAny()和invokeAll()




#### 大话java系列之线程池之延伸

`invokeAll()`

我先简单的给他下一个定义，该方法的作用是将任务合计tasks一起交给线程池去操作，并且可以返回我们的Future容器，由于我们的任务实现的是Callable接口，所以我们会返回一个对象，而该对象将放在Future中，一个任务就有一个Future对象，最后我们invokeAll()返回的就是Future对象的List合集。通过调用Future对象的get()方法得到我们返回的对象。

ExecutorService的invokeAll方法有两种用法：

1.exec.invokeAll(tasks)

2.exec.invokeAll(tasks, timeout, unit)

其中tasks指的是我们的任务合集，timeout针对的是所有的tasks，而不是单个task任务的执行时间，如果超时则会取消没有执行完的所有task任务，并抛出超时异常给Future对象。我们这里来通过我写的一个例子来看一下invokeAll()如何使用。

```
public class Person{
 private String name;
 private Integer age;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Person(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```


```
public class InvokeAllTest {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService mExecutor = Executors.newSingleThreadExecutor();
        List<Call> tasks=new ArrayList<Call>();
        for (int i=1;i<=10;i++){

            tasks.add(new Call(new Person("田俊哲")));

        }

        List<Future<Person>> futures = mExecutor.invokeAll(tasks, 10, TimeUnit.SECONDS);
        List<Person> personList=new ArrayList<Person>();

        for (Future<Person> future:futures){

            try {
                personList.add(future.get());

            } catch (ExecutionException e) {
                personList.add(new Person("任务异常",null));

            }catch (CancellationException e){
                personList.add(new Person("任务超时",null));

            }

        }

            for (Person person:personList){

                System.out.println(person);

            }
        mExecutor.shutdown();

    }

}
```

![java-javascript](/img/in-post/threadpool/14.png)
![java-javascript](/img/in-post/threadpool/15.png)


我来分析一下代码，我想做到的效果是用线程更改实体类Person的年龄，但我让线程除了改名字外还要休息三秒，已达到线程运行时间为三秒，将这个任务循环十次放入tasks，再放入invokeAll()方法中，还在invokeAll()中设置了我们允许执行的时间为10秒，超过十秒则不再执行任务。我这里使用的线程池的newSingleThreadExecutor策略，只有一个正式工一个一个的执行排序任务，我们执行一个任务需要三秒三个任务就是九秒，而到执行第四个的时候我们的时间将超过十秒，则直接停止运行，并将返回信息存储到List<Future<Person>>中返回。成功的任务返回结果，超时的任务则返回任务超时，并设置年龄为null。通过运行，我得到了正确的结果。除了前三个任务被线程设置了年龄，到第四个就返回任务超时null。这就是我们的invokeAll()方法。

这里要注意一下我们的Callable和Runnable，在我们方法要求有返回值的时候我们要实现Callable接口，这样才能把我们返回的对象装入Future返回。



`invokeAny()`

同样我先给大家举个例子，假如说我们现在要在我们的硬盘中找一个文件，我们怎做最快，我们会派一个线程去C盘找，在派一个线程去E盘找，有多少盘就派多少线程找，如果有一个线程报告，我找到了，那我们还让其他几个线程找吗？我们肯定就光速关闭其他几个线程去查看我们要找的文件了，同理，这个方法的意思就是，我们派多个线程去执行一些任务，第一个执行出来之后其他的就直接抛弃不在执行。

ExecutorService的invokeAny方法有两种用法：

1.exec.invokeAny(tasks)

2.exec.invokeAny(tasks, timeout, unit)

同上面invokeAll，唯一区别的timeout，因为我们当执行完第一个任务就退出不再执行，那么我们的timeout指的就是唯一的任务执行不能超过的时间 ，超过这个时间任务还没有结束，那么直接退出。

我们下面来看我写的一个简单的例子模板。


![java-javascript](/img/in-post/threadpool/16.png)
![java-javascript](/img/in-post/threadpool/17.png)

例子与上文的例子差不多，只是修改了call()中的任务，我让每一个任务的执行时间都不一样，有长有短，这样就会有一个所需最短的任务跳出来输出，其他的则不再执行。而且并没超出timeout的时间。这就是我们的invokeAny()方法。






#### 总结


就此，线程池的所有内容也就讲完了，我所讲的内容覆盖了大部分线程池知识，谢谢大家的关注。我们下一篇见。


#### 著作权声明


作者 [田俊哲](https://shmilyz.github.io)，首次发布于 [Shmilyz Blog](https://shmilyz.github.io)，本文原创，转载请保留以上链接！

