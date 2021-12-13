### 前言

之前学习完毕了javaNIO的知识，知道了javaNIO是一种异步的非阻塞的。使得可以让一个选择器线程处理成千上万个客户端连接，性能不会随着客户端的增加而线性下降。

当然NIO的知识，只是基础，对于实现通信的高性能和高并发，我们还要学习高性能服务器必备的设计模式：Reactor反应器设计模式

### 高并发IO的底层原理

#### IO读写的基础原理

​	为了避免用户进程之间操作内核，保证内核安全，操作系统将内存（虚拟内存）划分为两部分，一部分是内核空间，一部分是用户空间。在linux系统中，内核模块运行在内核空间，对应的进程处于内核态，而用户程序运行在用户空间中，对应的进程处于用户态。

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719091002388.png" alt="image-20210719091002388" style="zoom:50%;" />

​	操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内核空间，也有访问底层硬件设备的权限。内核空间总是驻留在内存中，它是为操作系统的内核保留的。

- 应用程序是不允许直接在内核空间区域进行读写，也是不容许直接调用内核代码定义的函数的。
- 每个应用程序进程都有一个单独的用户空间，对应的进程处于用户态，用户态进程不能访问内核空间中的数据，也不能直接调用内核函数的，因此要进行系统调用的时候，就要将进程切换到内核态才能进行。



> ​	 **内核态进程可以执行任意命令，调用系统的一切资源，而用户态进程只能执行简单的运算，不能直接调用系统资源，现在问题来了：用户态进程如何执行系统调用呢？**

**答案为：用户态进程必须通过系统接口（System Call），才能向内核发出指令，完成调用系统资源之类的操作。**

​		<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719091630729.png" alt="image-20210719091630729" style="zoom: 50%;" />

   用户程序进行IO的读写，依赖于底层的IO读写，基本上会用到底层read&write两大系统调用。虽然在不同的操作系统中，read&write两大系统调用的名称和形式可能不完全一样，但是他们的基本功能是一样的。

​	操作系统层面的read系统调用，并不是直接从物理设备把数据读取到应用的内存中；write系统调用，也不是直接把数据写入到物理设备。上层应用无论是调用操作系统的read，还是调用操作系统的write，都会涉及`缓冲区`。具体来说，上层应用通过操作系统的read系统调用，**是把数据从内核缓冲区复制到应用程序的进程缓冲区**；上层应用通过操作系统的write系统调用，是把**数据从应用程序的进程缓冲区复制到操作系统内核缓冲区**。
​	简单来说，应用程序的IO操作，实际上不是物理设备级别的读写，而是缓存的复制。read&write两大系统调用，都不负责数据在内核缓冲区和物理设备（如磁盘、网卡等）之间的交换。这项底层的读写交换操作，是由操作系统内核（Kernel）来完成的。所以，应用程序中的IO操作，无论是对Socket的IO操作，还是对文件的IO操作，都属于上层应用的开发，它们的在输入（Input）和输出（Output）维度上的执行流程，都是类似的，都是在内核缓冲区和进程缓冲区之间的进行数据交换。

##### 	内核缓冲区与进程缓冲区

> ​	为什么要设置那么多的缓冲区呢？

​	缓冲区的目的，是为了减少频繁的和物理设备之间进行物理交换，计算机的外部设备与内存还有cpu 的处理速度有着非常大的差距，如果没有缓冲区的话，会导致外部设备的速度慢严重拖着cpu 的后腿。 例如操作系统的中断操作操作，在中断之前需要保存进程数据和状态等信息，在结束中断之后还要恢复回来，为了减少底层系统的频繁中断导致的时间损耗、性能损耗，于是有了内核缓冲区。

​	有了内核缓冲区，操作系统内核会对内核缓冲区进行监控，等待缓冲区达到一定数量的时候，在进行IO设备的中断处理，集中执行物理设备的IO操作，通过这种方式可以提升系统的性能。

​	在linux系统中，操作系统内核只有一个内核，而每个用户程序都有自己的独立缓冲区，在linux系统中，大多数IO操作并没有进行实际的IO操作，而是在用户缓冲区和操作系统内核缓冲区进行数据交换。

-----

##### 典型的一种系统调用

用户程序所使用的read & write方法并不是在内核缓冲区和物理设备之间的交换。

- read调用把数据从内核缓冲区读入到用户缓冲区
- write调用把用户缓冲区的数据复制到内核缓冲区。

大概流程如下图（网络通讯）：

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719101951439.png" alt="image-20210719101951439" style="zoom:50%;" />

这里有两个完整的流程。

- 应用程序在等待数据完全准备好。
- 把数据从内核缓冲区复制到用户缓冲区。

具体如下：

1. 客户端发送一个请求，然后在服务器的网卡会有数据接收到，并被读入到内核缓冲区。
2. 应用程序等待所有的数据从操作系统内核缓冲区复制到用户缓冲区。
3. 服务器处理收到的请求。

------

#### 四种主要的IO模型

##### 同步阻塞IO

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719102523407.png" alt="image-20210719102523407" style="zoom:50%;" />

在java发起一个socket的read读操作的系统调用：

1. 从java进行IO读后发起系统的调用开始，用户线程进入阻塞状态
2. 当操作系统内核收到read调用的时候，这个时候数据可能还没有到达内核缓冲区，这个时候内核就会等待。
3. 内核一直等待完整的数据到达，就会将数据从内核缓冲区复制到用户缓冲区。
4. 用户程序等待复制完成后才会解除阻塞，继续运行起来。



同步阻塞的特点：

- 在read调用开始到复制完成调用返回到过程中，用户程序一直是阻塞的状态。

  - 优点
    - 这种方式的应用开发非常的简单；在阻塞期间用户线程被挂起，基本上不会占用cpu资源
  - 缺点
    - 一般情况下会为每个连接配置一个独立的线程，在高并发的情况下，系统的性能开销会非常大。

  -------

##### 同步非阻塞NIO

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719104028179.png" alt="image-20210719104028179" style="zoom:50%;" />

​	同步非阻塞IO和BIO的最大区别就是，在操作系统内核区数据没有准备好的时候，可以让程序立即返回。

所以流程变成了这样：

1. 判断是否准备好，没有则直接返回，有则去把数据从内核缓冲区复制到用户缓冲区。
2. 用户线程处理数据



NIO的特点：

- 在开始复制到调用返回到阶段才会阻塞用户进程。
- 在查看操作系统内核是否准备好数据时可以直接返回。
  - 优点
    - 更加的灵活了，在等待数据的时候可以立即返回，用户线程不会阻塞，实时性较好。
  - 缺点
    - 需要不断的轮询内核，这将占用大量的cpu实际，效率低下。

在开发中我们依然还是不会使用这种模型，性能也是比较差劲的。

------

##### IO多路复用模型

​	为了解决同步非阻塞模型中的轮询等待问题，出现了IO多路复用模型。

​	在IO多路复用模型引入了一种新的系统调用，查询IO的就绪状态。在linux系统中，对应的是`selcet`、`epoll`系统调用，一个进程可以监视多个文件描述符（文件读取，socket连接），当某个描述符就绪，内核就将就绪的状态返回给应用程序。然后应用程序就根据应用程序根据就绪的状态，进行相应的IO系统调用。

​	<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719113010681.png" alt="image-20210719113010681" style="zoom:50%;" />

​	举个例子来说明IO多路复用模型的流程。发起一个多路复用IO的read读操作的系统调

用，流程如下：

（1）选择器注册。在这种模式中，首先，将需要read操作的目标文件描述符（socket连接），提前注册到Linux的select/epoll选择器中，在Java中所对应的选择器类是Selector类。然后，才可以开启整个IO多路复用模型的轮询流程。

（2）就绪状态的轮询。通过选择器的查询方法，查询所有的提前注册过的目标文件描述符（socket连接）的IO就绪状态。通过查询的系统调用，内核会返回一个就绪的socket列表。当任何一个注册过的socket中的数据准备好或者就绪了，就是内核缓冲区有数据了，内核就将该socket加入到就绪的列表中，并且返回就绪事件。 

（3）用户线程获得了就绪状态的列表后，根据其中的socket连接，发起read系统调用，用户线程阻塞。内核开始复制数据，将数据从内核缓冲区复制到用户缓冲区。

（4）复制完成后，内核返回结果，用户线程才会解除阻塞的状态，用户线程读取到了数据，继续执行。



IO多路复用模型的特点

- ​	设计两种系统调用，一种是IO操作系统的调用，一种是select/epoll的就绪查询系统调用。IO多路复用模型建立在操作系统的基础之上，操作系统必须是支持能够提供多路分离的系统调用select/epoll。
- 和NIO类似，多路复用IO也需要轮询。负责select/epoll状态查询调用的线程，需要不断的轮询。
  - 优点
    - 一个选择器查询线程，可以同时处理成千上万的网络连接，所以用户应用程序不需要大量的创建连接，也不必维护线程，从而大大的减少小了系统的开销。
  - 缺点
    - select/epoll本质上调用也是阻塞的，属于同步IO。都需要在读写事件就绪后，由系统本身负责进行读写，这个过程依然还是阻塞的。

为了彻底的解决线程阻塞的问题，就必须使用异步IO模型。

----

##### 异步IO模型 （AIO）

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20210719114148017.png" alt="image-20210719114148017" style="zoom:50%;" />

举个例子。发起一个异步IO的read读操作的系统调用，流程如下：

（1）当用户线程发起了read系统调用，立刻就可以开始去做其他的事，用户线程不阻塞。

（2）内核就开始了IO的第一个阶段：准备数据。等到数据准备好了，内核就会将数据从内核缓冲区复制到用户缓冲区。

（3）内核会给用户线程发送一个信号（Signal），或者回调用户线程注册的回调方法，告诉用户线程，read系统调用已经完成了，数据已经读入到了用户缓冲区。 

（4）用户线程读取用户缓冲区的数据，完成后续的业务操作。

异步IO模型的特点：在内核等待数据和复制数据的两个阶段，用户线程都不是阻塞的。用户线程需要接收内核的IO操作完成的事件，或者用户线程需要注册一个IO操作完成的回调函数。正因为如此，异步IO有的时候也被称为信号驱动IO。

​	异步IO异步模型的缺点：应用程序仅需要进行事件的注册与接收，其余的工作都留给了操作系统，也就是说，需要底层内核提供支持。

​	理论上来说，异步IO是真正的异步输入输出，它的吞吐量高于IO多路复用模型的吞吐量。就目前而言，Windows系统下通过IOCP实现了真正的异步IO。而在Linux系统下，异步IO模型在2.6版本才引入，JDK的对其的支持目前并不完善，因此异步IO在性能上没有明显的优势。

​	大多数的高并发服务器端的程序，一般都是基于Linux系统的。因而，目前这类高并发网络应用程序的开发，大多采用IO多路复用模型。大名鼎鼎的Netty框架，使用的就是IO多路复用模型，而不是异步IO模型。

------

#### 合理配置支持百万级并发连接

在Linux下，通过调用ulimit命令，可以看到一个进程能够打开的最大文件句柄数量，这个命令的具体使用方法是：

`ulimit -n`

`ulimit` 命令是用来显示和修改当前用户进程一些基础限制的命令，-n选项用于引用或设置当前的文件句柄数量的限制值，Linux的系统默认值为1024。

理论上1024个文件描述符，对绝大多数应用（例如Apache、桌面应用程序）来说已经足够了。但是，是对于一些用户基数很大的高并发应用，则是远远不够的。一个高并发的应用，面临的并发连接数往往是十万级、百万级、千万级、甚至像腾讯QQ一样的上亿级。

文件句柄数不够，会导致什么后果呢？当单个进程打开的文件句柄数量超过了系统配置的上限值时，就会发出“Socket/File:Can't open so many files”的错误提示。

所以，对于高并发、高负载的应用，就必须要调整这个系统参数，以适应处理并发处理大量连接的应用场景。可以通过ulimit来设置这两个参数。方法如下：

`ulimit -n 1000000`

在上面的命令中，n的设置值越大，可以打开的文件句柄数量就越大。建议以root用户来执行此命令。使用ulimit命令有一个缺陷，该命令仅仅只能修改当前用户环境的一些基础限制，仅在当前用户环境有效。也即是说，在当前的终端工具连接当前shell期间，修改是有效的；一旦断开用户会话，或者说用户退出Linux后，它的数值就又变回系统默认的1024了。并且，系统重启后，句柄数量又会恢复为默认值。

ulimit命令只能用于临时修改，如果想永久地把最大文件描述符数量值保存下来，可以编辑/etc/rc.local开机启动文件，在文件中添加如下内容：

`ulimit -SHn 1000000`

以上示例增加-S和-H两个命令选项。选项-S表示软性极限值，-H表示硬性极限值。硬性极限是实际的限制，就是最大可以是100万，不能再多了。软性极限值则是系统发出警告

（Warning）的极限值，超过这个极限值，内核会发出警告。

普通用户通过ulimit命令，可将软极限更改到硬极限的最大设置值。如果要更改硬极限，必须拥有root用户权限。

终极解除Linux系统的最大文件打开数量的限制，可以通过编辑Linux的极限配置文件

`/etc/security/limits.conf`来解决，修改此文件，加入如下内容：

```bash
soft nofile 1000000

hard nofile 1000000
```

soft nofile表示软性极限，hard nofile表示硬性极限。

举个实际例子，在使用和安装目前非常流行的分布式搜索引擎——ElasticSearch时，基本上就必须去修改这个文件，用于增加最大的文件描述符的极限值。当然，在生产环境运行Netty时，也需要修改`/etc/security/limits.conf`文件，增加文件描述符数量的限制。

### Reactor反应器模式

[疯狂创客园关于Reactor的介绍](https://www.cnblogs.com/crazymakercircle/p/9833847.html)

全宇宙最有名的、最高性能的Web服务器 Nginx，就是基于反应器模式的；Redis、Netty更是基于反应器模式的。

#### 反应器模式简介

反应器模式由 两个重要角色组成：

- **Reactor线程**

  - 负责相应IO事件，并且分发到`Handlers`处理器

- Handlers处理器

  - 可以非阻塞的执行业务处理逻辑

  这里可以看看一位大师：Doug Lea 的文章[《Scalable IO in java》](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)

理解一下一些基础知识。

#### 多线程OIO模式的致命缺陷

在最初的和最原始的网络服务器程序中，是使用一个while 循环，不断的监听端口是否有新的连接。如果有，那么就调用一个处理函数来处理，就像这样:

```java
while(true){
    socket = accept(); //阻塞在这里等待 连接
    handle(socket); //读取数据，处理数据，写入结果
}
```

这种问题存在的这样的一个问题：如果前面一个的任务没有处理完，那么就会阻塞住后面的任务和连接，这样我们的服务器的吞吐量太低了。于是乎就诞生了一种极为经典的模式：

> `Connection Per Thread`-为每个连接都创建一个线程。

对于每一个新的网络连接请求，都会分配给一个新的线程。每个线程都有独自处理自己负责的输入和输出。当然，服务器的监听线程也是独立的，任何一个socket连接不会阻塞到另外的新建立的socket连接。早期的`Tomcat`服务器就是这样实现的。

`Conncetion Per Thread` 为每个连接创建一个新的线程，从一定的程度上，极大的提高了服务器的吞吐量，但是诞生了一个新的问题。

如果连接太多了，创建线程就太耗费资源了，能不能存在这样的一种方式，把连接放在一个线程里面呢？

答案是：当然可以，但是有没有想过会产生这样的一个问题。我们的OIO方式会阻塞住后面的操作。即使我们将连接放到一个线程中，他们大多数还是在等待前面的连接先完成。所以无论怎么做还是只能一个线程在一个时间内处理一个连接。

有没有这样的可以解决掉我们`Connection Per Thread`模式的巨大缺陷的办法呢？这里就要使用我们提到一个 反应器模式了。

#### Reactor模式小结 

总体来看，Reactor模式有点像事件驱动模式。

在事件驱动模式中，当有事件触发的时候，事件源会将事件dispatch分发到handler处理器进行事件处理。

反应器模式中有` Reactor反应器`和`handler处理器`两个重要的组件

- Reactor: 负责查询IO事件，当检测到一个IO事件，将其发送给相应的`handler`去处理。这里的IO事件，就是NIO中选择器监控的 `通道IO事件`。
- Handler：与IO事件绑定，负责IO事件的处理。
  - 真正的建立连接
  - 通道的读取
  - 处理业务逻辑
  - 负责将结果写出到通道

--------

### 单线程Reactor反应器

![image-20210627142507469](https://myselfd.oss-cn-hangzhou.aliyuncs.com/img/image-20210627142507469.png)


 Java的NIO模式的Selector网络通讯，就是一个简单的Reactor模型，可以说是Reactor模型的朴素模型。

上面的图是最简单的单Reactor单线程模型。Reactor线程是一个多面手，负责多路分离套接字，Accept新连接，并分派请求到Handler处理器中。

> Scalable IO in Java 中 实现的单线程 Reactor的参考代码

**Reactor:**

```java
package com.crazymakercircle.ReactorModel;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

class Reactor implements Runnable
{
    final Selector selector;
    final ServerSocketChannel serverSocket;

    Reactor(int port) throws IOException
    { //Reactor初始化
        selector = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(port));
        //非阻塞
        serverSocket.configureBlocking(false);

        //分步处理,第一步,接收accept事件
        SelectionKey sk =
                serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        //attach callback object, Acceptor
        sk.attach(new Acceptor());
    }

    public void run()
    {
        try
        {
            while (!Thread.interrupted())
            {
                selector.select();
                Set selected = selector.selectedKeys();
                Iterator it = selected.iterator();
                while (it.hasNext())
                {
                    //Reactor负责dispatch收到的事件
                    dispatch((SelectionKey) (it.next()));
                }
                selected.clear();
            }
        } catch (IOException ex)
        { /* ... */ }
    }

    void dispatch(SelectionKey k)
    {
        Runnable r = (Runnable) (k.attachment());
        //调用之前注册的callback对象
        if (r != null)
        {
            r.run();
        }
    }

    // inner class
    class Acceptor implements Runnable
    {
        public void run()
        {
            try
            {
                SocketChannel channel = serverSocket.accept();
                if (channel != null)
                    new Handler(selector, channel);
            } catch (IOException ex)
            { /* ... */ }
        }
    }
}

```

**Handler:**

```java
package com.crazymakercircle.ReactorModel;


import com.crazymakercircle.config.SystemConfig;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;

class Handler implements Runnable
{
    final SocketChannel channel;
    final SelectionKey sk;
    ByteBuffer input = ByteBuffer.allocate(SystemConfig.INPUT_SIZE);
    ByteBuffer output = ByteBuffer.allocate(SystemConfig.SEND_SIZE);
    static final int READING = 0, SENDING = 1;
    int state = READING;

    Handler(Selector selector, SocketChannel c) throws IOException
    {
        channel = c;
        c.configureBlocking(false);
        // Optionally try first read now
        sk = channel.register(selector, 0);

        //将Handler作为callback对象
        sk.attach(this);

        //第二步,注册Read就绪事件
        sk.interestOps(SelectionKey.OP_READ);
        selector.wakeup();
    }

    boolean inputIsComplete()
    {
        /* ... */
        return false;
    }

    boolean outputIsComplete()
    {

        /* ... */
        return false;
    }

    void process()
    {
        /* ... */
        return;
    }

    public void run()
    {
        try
        {
            if (state == READING)
            {
                read();
            }
            else if (state == SENDING)
            {
                send();
            }
        } catch (IOException ex)
        { /* ... */ }
    }

    void read() throws IOException
    {
        channel.read(input);
        if (inputIsComplete())
        {

            process();

            state = SENDING;
            // Normally also do first write now

            //第三步,接收write就绪事件
            sk.interestOps(SelectionKey.OP_WRITE);
        }
    }

    void send() throws IOException
    {
        channel.write(output);

        //write完就结束了, 关闭select key
        if (outputIsComplete())
        {
            sk.cancel();
        }
    }
}


```

> 单线程模式的缺点

1. 当其中的一个hander阻塞时，会导致其他所有的handler得不到执行，更严重的是会导致整个服务不能接收到新的连接。
2. 不能利用多核处理器的。
3. 仅仅适用于handler中业务处理组件能快速完成的场景

------

### 多线程的Reactor

1. 当`Future`对象刚刚创建的时，处于非完成状态，调用者可以通过返回的`ChannelFuture`来获取操作执行的状态，注册监听函数来执行完成后的操作。
2. 常用操作如下
   - `isDone`判断当前操作是否完成
   - `isSuccess`判断已经完成的操作是否操作成功。
   - `getCause`获取已经完成的操作，失败的原因。
   - `isCancelled`判断当前操作是否被取消。
   - `addListener`注册监听器，当操作完成（`isDone`方法回）将会通知指定的监听器，如果Future对象已完成，则通知指定的监听器

