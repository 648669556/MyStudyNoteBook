# java并发编程学习



### 进程和线程

一个进程往往包含多个线程，至少包含一个。

> **Java默认有几个线程？** 

​	2个线程  main，GC线程

java Thread、Runnable、Callable

> **java真的可以开启线程吗？** 

不能！java通过调用本地的c++native方法来开启线程。

> **并发、并行**

并发：多线程操作同一个资源

- 例如多个线程同时访问修改一个数组

并行：多个CPU同时执行

```java
//获取cpu的核数 
Runtime.getRuntime().availableProcessors();
```

**并发编程的本质**

- 充分利用CPU的资源。

### 线程知识

> **线程有几个状态？**

- NEW 新生
- Runnable 运行
- Blocked 阻塞
- Waiting 等待，死死的等
- Timed_Waiting 超时等待
- Terminated 终止

> **wait/sleep区别？**

1. **来自不同的类**
   - wait=>object
   - sleep=>java.util.concurrent
2. **关于锁的释放**
   - wait会释放锁，sleep睡觉了，抱着锁睡觉，不会释放
3. **使用的范围**
   - wait必须在同步代码块中
   - sleep可以在任何地方睡
4. **释放需要捕获异常**
   - wait 不需要捕获异常
   - sleep 必须要捕获异常

###  lambda表达式

```java
new Thread(()->{},"A").start();
```



### Lock 锁

------

> 传统的是 synchronized

线程就是一个单独的资源类，没有任何附属的操作

synchronized 本质是一个队列，锁。

> lock接口

ReentrantLock() 一个锁，在实例化的时候可以传入true或者false来生成一个使用公平锁的或者是一个不公平锁的类，java中默认为非公平锁可以插队。

Lock三部曲：

- new一个锁
- 加锁
- 解锁 finally中写

> **Synchronized 和 Lock的区别**

1. Synchronized 是内置的关键字，Lock一个java类
2. Synchronized 无法判断获取锁的状态，Lock可以判断是否获取到了锁
3. Synchronized 会自动释放锁,Lock锁必须要手动解锁！如果不释放就会**死锁**
4. Synchronized 线程1（获得锁）、线程2（等待）Lock锁就不一定会等待下去了。
5. Synchronized 可重入锁，不可以中断的，非公平；Lock，可重入锁，可以判断锁，可以自己设置公平锁和非公平锁
6. Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码。

> **锁是什么？如何判断锁的是谁？**

### 生产者和消费者写法

----------

> **Synchronized 版本**

 ```JAVA
while(number==0){//使用while防止虚假唤醒
    this.wait();
}
number--;
this.notifyAll();
=====================================
while(number!=0){
    this.wait();
}
number++;
this.notifyAll();
 ```

> **Lock版本**

```java
Lock lock=new Lock();
Condition condition = lock.newCondition();

//标准模板
lock.lock();
try{
    //业务代码
    while(number!=0){
        condition.await();
    }
    condition.signalAll();
}catch(Exception e){
    e.printStackTrace();
}finally{
    lock.unlock();
}
```

> **Condition 的优势**

**精准的通知和唤醒线程**

选择对应 的condition监视器执行signal方法就可以了。

### 8锁问题

---------

> **什么是8锁现象?**

就是关于锁的8个问题。

1. 标准情况下，两个线程先执行那一个？
   - 锁的存在，synchronized锁的对象是方法的调用者！谁先拿到对象谁先执行！
2. 如果没有锁先执行哪一个？
   - 如果没有被锁影响，普通方法不会被影响。
3. 如果是两个对象呢？
   - 两个不同的对象之间不会互相影响。

## 容器与实现

### CopyOnWriteArrayList

使用：

```java
List<String> list =new CopyOnWriteArrayList<>();
```

这样生产的一个列表是线程安全的。通过读写分离的方式来新增，性能有保障。

```java
 public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
    }
```

写入的时候是需要加锁的为了保证copy的时候是同一个副本。

#### CopyOnWrite的应用场景

　　CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。当用户搜索时，会检查当前关键字在不在黑名单当中，如果在，则提示不能搜索。

#### CopyOnWrite的缺点　

CopyOnWrite容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题。所以在开发的时候需要注意一下。

　　`内存占用问题`。因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务由于每晚使用CopyOnWrite机制更新大对象，造成了每晚15秒的Full  GC，应用响应时间也随之变长。

　　针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

　　`数据一致性问题`。CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，**马上能读到**，请不要使用CopyOnWrite容器。

####  CopyOnWriteArrayList为什么并发安全且性能比Vector好

　我知道Vector是增删改查方法都加了synchronized，保证同步，但是每个方法执行的时候都要去获得锁，性能就会大大下降，而CopyOnWriteArrayList 只是在增删改上加锁，但是读不加锁，在读方面的性能就好于Vector，CopyOnWriteArrayList支持`读多写少`的并发情况。

### CopyOnWriteArraySet

#### 本质

 CopyOnWriteArrayList类似，**其实CopyOnWriteSet底层包含一个CopyOnWriteList**，几乎所有操作都是借助**CopyOnWriteList** 

####  CopyOnWriteArraySet具有以下特性：

1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

### ConcurrentHashmap

### Callable

- `Callable`接口类似于`Runnable` ，因为它们都是为其实例可能由另一个线程执行的类设计的。 然而， `Runnable`不返回结果，也不能抛出被检查的异常。 

  该`Executors`类包含的实用方法，从其他普通形式转换为`Callable`类。  

- 返回结果并可能引发异常的任务。 实现者定义一个没有参数的单一方法，称为`call` 

1、可以有返回值

2、可以抛出异常

3、方法不同  call()方法

#### 怎么去使用

如果要放入线程Thread中我们不能直接放入，因为Thread只能接收Runnable类型的类。

所以我们使用一个`FutureTask`来辅助我们，相当与一个装饰器。

```java
FutrueTask task =new FutrueTask(Callable);
```

在实例化`FutrueTask`的同时传入一个`Callable`类这样。也可以使用lamda表达式来简化。

```java
FutureTask futureTask = new FutureTask(()->{
            System.out.println("hello");
            return "d";
        });
```

因为需要返回值所以我们需要return;

启动线程的方式

```java
new Thread(futureTask,"A").start();
```

启动后就可以看到我们的callable被调用了。



如果要获取返回值需要我们执行

``futureTask.get()``方法

这个方法是会有可能阻塞主线程的，因为可能子线程没有执行完毕返回结果的话，就需要主线程阻塞等待结果的返回。

futureTask是有缓存的，一个futureTask调用只能被调用一次，保留返回值。

简单来说，**就是对于同一个FutureTask类进行get第二次会很快**

### 辅助工具

#### CountDownLatch

一个帮助记数的减法器。

我们需要初始化计数器

``` java
CountDownLatch cdl=new CountDownLatch(5)
```

这样就有了一个减法计数器。这个辅助类必须传入参数才能构造。

在线程中我们需要执行减法器的countDown()方法，或者是线程中断了都会导致记数减一



```java
new Thread(()->{
    cdl.countDown();
}).start();
```

在主线程中我们需要使用await()方法来等待计数器归零。

计数器归零后则会向下继续执行。



##### 案例

```java
CountDownLatch countDownLatch =new CountDownLatch(5);

for(int i=0;i<5;i++){
    new Thread(()->{
        //业务代码
        countDownLatch.countDown();
    }).start();
    countDownLatch.await();
}
```

`await() `，方法将会一直等待直到计数器归零。

我们可以使用`await(long timeout,TimeUnit unit) `方法，

使用这个方法，如果时间超过规定的时间后就会停止等待，继续向下执行

`timeout`,是数字，`TimeUnit` 是单位。

`countDownLatch.getCount()`可以获取到当前的记数；

#### CyclicBarrier

一种循环触发计数器

这个是当记数数量达到我们设定的数量的时候，就会执行我们原先设定的一个Runnable方法，并重置记数.

这个方法内，对于记数用了ReentrantLock 锁来同步。

##### 使用方法

```java
 CyclicBarrier cyclicBarrier=new CyclicBarrier(2,()->{
            System.out.println("达成目标");
        });
        for (int i = 1; i <=6; i++) {
            final int tempint=i;
            new Thread(()->{
                System.out.println(tempint+"报名");
                try {
                    cyclicBarrier.await();
                    System.out.println(String.valueOf(tempint)+":进入了比赛");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }

```



#### Semaphore

信号量

这个就直接看使用方法把

``` java
Semaphore sp=new Semaphore(1,true);
//第一个一般是资源数量，第二个如果是true的话说明它可以保证此信号量可以在争用条件下先进先出。
sp.acquire(); //可以填入需要的资源数量。
//就是请求资源，如果资源被用完了，就需要等待别人释放release或者中断了才可以拿到资源。
sp.release();//释自己的资源是释放。
```

#### ReadWriteLock

读写锁



需要先了解一下`独占锁`和`共享锁`是什么意思

- 独占锁 就是一个资源被人使用了，别人不可以读，不可以写入，只有占有者可以使用
- 共享锁 就是一个资源可以被别人读，但是别人不能写入。

```java
 ReadWriteLock readWriteLock=new ReentrantReadWriteLock();

```

![1607849658666](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\1607849658666.png)

在可重入读写锁中，有读锁和写锁。

我们使用起来比较简单。



如果我们要写入

就获取写锁

写法与lock锁一致

## 阻塞队列BlockingQueue



![A BlockingQueue with one thread putting into it, and another thread taking from it.](http://tutorials.jenkov.com/images/java-concurrency-utils/blocking-queue.png)

阻塞队列，一边生产一边消费类似。

阻塞队列，其实就是队列，但是与一般的队列不同的是，你可以等待阻塞的去加入队列。

他有两个实现类，一个ArrayBlockingQueue（有界队列） 一个 LinkedBlockingQueue（无界队列） 很明显一个是数组的，一个是链表形式的，数据结构不同罢了，但是他们的调用方法相似。



 > #### 四组API

| 方法 | 抛出异常  | 不会抛出异常          | 阻塞等待                   | 超时等待(如果时间超过就会退出) |
| ------------ | ---------------------- | --------------------- | -------------------------- | :--------------------------------- |
| 添加 | add()    返回值boolean | offer() 返回值boolean | put() 无返回值，会一直等待 | offer(object,long time,TimeUnit e) |
| 移出 | remove() 返回值E     | poll() 返回值e    | take() 会阻塞一直等待      | poll(object,long,TimeUnit e)  |
| 检测队首元素 | element() | peek() |  ||

### ArrayBlockingQueue

--------

`ArrayBlockingQueue`实现了BlockingQueue接口。

`ArrayBlockingQueue`是有界的，这就是意味着我们不能无限制的向里面存放元素，你必须在实例化的时候就设定好他的元素上限，而且在之后是无法改变的。 

`ArrayBlockingQueue`储存元素通过FIFO的方式，头部的元素是存放最久的，尾部是最新的。

​    Here is how to instantiate and use an `ArrayBlockingQueue`:

```java
BlockingQueue queue = new ArrayBlockingQueue(1024);

queue.put("1");

Object object = queue.take();
```

​    Here is a `BlockingQueue` example that uses Java Generics.

 Notice how    you can put and take String's instead of :

```java
BlockingQueue<String> queue = new ArrayBlockingQueue<String>(1024);

queue.put("1");

String string = queue.take();
```



### DelayQueue

-------------

元素在阻塞队列中到达延迟时间才会被释放，这个元素必须实现`java.util.concurrent.Delayed`接口

例如：

```java
public interface Delayed extends Comparable<Delayed> {

 public long getDelay(TimeUnit timeUnit);

}
```

`getDelay`的返回值应该是当前元素还会阻塞多少时间。如果返回的是0或者一个负值，那么这个元素将会被考虑释放，可以通过take()方式获取到。

​    The `TimeUnit` instance passed to the `getDelay()` method is an `Enum` that    tells which time unit the delay should be returned in. The `TimeUnit` enum can take these values:

```
DAYS
HOURS
MINUTES
SECONDS
MILLISECONDS
MICROSECONDS
NANOSECONDS
```

```java
public class DelayQueueExample {

    public static void main(String[] args) {
        DelayQueue queue = new DelayQueue();

        Delayed element1 = new DelayedElement();

        queue.put(element1);

        Delayed element2 = queue.take();
    }
}
```

All in All ,DelayQueue是一个有着奇怪特性的队列，可以在延迟一段时间后再去被获取到。



### PriorityBlockingQueue

------

优先级阻塞队列，因为优先级队列的原因，他的规则和`java.util.PriorityQueue`相同，你不能插入空值到这个队列中。

所有要插入这个队列的元素都要实现`java.lang.Comparable`接口。 队列里面的元素会排序按照你写的比较规则实现。

需要注意的是，如果你使用`Iterator`获取元素在`PriorityBlockingQuquq`中，迭代器不一定能保证返回正确的优先级顺序。



> #### SynchronousQueue 同步队列

```java
BlockingQueue<String> blockingQueue=new SynchronousQueue<>();
```

这个同步队列的线程如果一个元素没有被取出或者放入，他的放入`put()`和拿出`take()`就会一直阻塞。



## 线程池

### 池化技术

实现准备好一些资源，有人要用就来我这里拿。

线程池必考：**三大方法，7大参数，4种拒绝策略**

#### 线程池的好处

1. 降低资源的消耗
2. 提高响应的速度
3. 方便管理

线程服用、可以控制最大并发数，管理线程



#### 线程池三大方法

```java
//1、单个线程
Executors.newSingleThreadExecutor();
//2、创建一个固定大小的线程池
Executors.newFixedThreadPool(int n);
//3、可变大小的线程池
Executors.newCachedThreadPool();
//4、一个可以执行定时任务的线程池
Executors.newScheduledThreadPool();

```

很多公司不推荐使用`Executors`来创建线程池，因为这样创建的线程池，是我们不知道具体详情的，可能会导致创建太多的线程导致资源被耗光。最好我们手动创建线程，就需要我们下面的在这个方法。



##### 执行方法

- `execute()`通过这个方法可以执行一个`Runnable`这样来使用我们的线程池
- 在线程池使用完毕后需要关闭线程池释放资源我们使用 `shutdown()`方法

#### 7大参数

是ThreadPoolExecutor中的构造函数的参数

1. int corePoolSize //核心线程池大小
   - 预热完毕后情况下的线程池大小
2. int maximumPoolSize //最大核心线程池大小
   - 当线程池大小不够用的时候，就去拓展线程池大小，达到最大为止。
3. long keepAliveTime 超时了没有人去调用就会释放
   - 超时之后会去释放。
4. TimeUnit unit  //超时单位
5. BlockingQueue<Runnable> workQueue //阻塞队列 
   - 可以使用 ArrayBlockingQueue （有界队列）如果使用这种队列我们需要设置最大核心线程池大小
   - LinkedBlockingQueue（无界队列）使用了这种队列我们的阻塞队列就不会满，也就不需要设置最大核心线程池大小和超时。
   - SynchronousQueue （同步队列）使用了同步队列我们的线程 执行和我们的放入是同步的，就相当于没有阻塞队列（个人理解）来了就处理，没有线程就创建，不然就拒绝
6. ThreadFacory threadFactory //线程工厂：创建线程的一般不用动,阿里手册里面需要我们自己去写一个线程创建工程，这样我们可以方便给线程命名，方便后期排查。
   1. 创建一个类去实现我们的ThreadFactory接口
   2. 我们可以设置线程组，线程前缀名，统一的优先级。
7. RejectedExecutionHandler hanler //拒绝策略
   1. AbortPolicy
      - 线程池的默认策略，使用该策略的时候，如果线程池队列满了就会丢弃这个任务并且抛出RejectedExecutionException异常.
   2. DiscardPolicy
      - 这个也是直接抛弃但是不会抛出异常
   3. DiscardOldestPolicy
      - 会将最早进入队列的任务删除腾出空间，再尝试加入队列。
   4. CallerRunsPolicy
      - 使用此策略，如果添加到线程池失败，那么主线程会自己去执行该操作，不会等待线程池中的线程去执行。
   5. 自定义拒绝策略
      1. 实现RejectedExecutionHandler接口就可以了。

> 线程池的请求过程

收先我们如果有需要用到线程，就去线程池里面拿现有的线程使用。如果没有线程的话就会创建线程，一直到达到corePoolSize。这个时候如果我们设置的阻塞队列是有容量的，并且，我们的线程数量还在提高，那么我们的任务将会进入阻塞队列。如果我们的阻塞队列容量达到了上限，并且我们的最大线程数量大于核心线程数量，那么我们就会继续创建线程，来帮助我们处理任务。一直到我们的线程数量达到了最大核心线程数量。如果还有任务到达，那么就需要我们的拒绝策略来帮忙了，根据我们拒绝策略的不同我们会做出不一样的判断。

#### 线程池的数量选择

如果是CPU密集型应用，则线程池的 大小设置为N+1;

如果是IO密集型的应用，则线程池的大小设置为2N+1；

最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目 

---------

## ThreadLocal 详解

使用ThreadLocal可以实现线程隔离，是实现无锁编程的重要手段。



> JDK 1.8 ThreadLocal 如何实现线程隔离的。

首先