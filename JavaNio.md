### NIO概述

java的nio由以下几个核心部分组成：

- Channels
- Buffers
- Selectors



#### Channel和Buffer

基本上，所有的IO在NIO中都是从一个Channel开始。Channel优点像流。但是数据可以从Channel中读到Buffer中，也可以从Buffer写到Channel中。

<img src="http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers1.png" alt="img" style="zoom:75%;" />

> NIO中的Channel的实现

- FileChannel         【文件】
- DatagramChannel 【UDP】
- SocketChannel       【TCP】
- ServerSocketChannel 【像web服务器一样的监听TCP】

> Buffer的实现类

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

#### Selector

`selector`允许单线程处理多个`Channel`。如果你的应用打开了多个连接，但是每个连接的流量都很低，使用`Selector`就会很方便。例如在一个聊天服务器里面。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![img](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

要使用`Selector`，得向`Selector`注册`Channel`，然后调用的他的`select()`方法。这个方法会一直阻塞到某个注册的通道由事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有，新的连接进来了、数据接收等。

### **Channel**

javaNIO的通道有点像流，但是又有些不同。

1. 既可以从通道中读取数据，也可以写数据到通道里面。但是流的读写通常是单向的。
2. 通道可以异步的读写。
3. 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

> 基本的Channel示例

```java
//创建一个可以来回读写文件的FileIo流
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
//Buffer的初始化
ByteBuffer buf = ByteBuffer.allocate(48);
//读操作 RandomAccessFile每次读后都会将指针向后移动
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {

System.out.println("Read " + bytesRead);
    //切换读写模式，如果是写入模式就切换到读模式
buf.flip();
//获取到buffer里面已经有的内容
while(buf.hasRemaining()){
System.out.print((char) buf.get());
}
//读完之后清空buffer以变下一次写入
buf.clear();
    //继续读入我们文件中的内容
bytesRead = inChannel.read(buf);
}
aFile.close();
```

**首先将数据读取到Buffer里面(写操作)，然后我们再去Buffer里面读取出来(读操作)**。这不同与我们以前的之间从文件中读取出来。

### **Buffer**

#### **Buffer**的基本用法

---------

使用Buffer读写数据一般遵循四个步骤：

1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`或者`compact()`方法来清除

通过调用`flip()`方法我们可以将一个Buffer的状态在**读**或者**写**之间切换。

一旦读完成了，我们就有可能需要清空已经读过的数据，把我们的缓冲区空出来。

1. `clear()`方法的调用我们可以清空整个`Buffer`;
2. `compact()`方法我们会清除那些已经读取过的数据，然后将没有读取的数据移动到`Buffer`的起始处，新写入的数据将放到缓冲区未读数据的后面。



#### **Buffer**的**capacity**,**position**和**limit**

-----------

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

为了理解Buffer的工作原理，需要熟悉它的三个属性：

- **capacity**
- **position**
- **limit**

position和limit的含义取决于Buffer处在读模式还是写模式。

不管Buffer处在什么模式，capacity的含义总是一样的。

这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在插图后面。

![img](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)

> Capacity

作为一个内存块，`Buffer`有一个固定的大小值，也叫“**capacity**”.你只能往里写**capacity**个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

> Position

当你写数据到`Buffer`中时，**position**表示当前的位置。

初始的**position**值为0.

当一个byte、long等数据写到`Buffer`后， **position**会向前移动到下一个可插入数据的`Buffer`单元。

**position**最大可为`capacity – 1`.

当读取数据时，也是从某个特定位置读。当将`Buffer`从写模式切换到读模式，**position**会被重置为0. 当从`Buffer`的**position**处读取数据时，**position**向前移动到下一个可读的位置。

> Limit

在写模式下，`Buffer`的**limit**表示你最多能往`Buffer`里写多少数据。

 写模式下，**limit**等于`Buffer`的**capacity**。

当切换`Buffer`到读模式时，  **limit**表示你最多能读到多少数据。因此，当切换Buffer到读模式时，**limit**会被设置成写模式下的**position**值。

换句话说，你能读到之前写入的所有数据（**limit**被设置成已写数据的数量，这个值在写模式下就是**position**）



#### Buffer的类型

----------

Java NIO 有以下Buffer类型

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

如你所见，这些Buffer类型代表了不同的数据类型。

换句话说，就是可以通过`char`，`short`，`int`，`long`，`float` 或 `double`类型来操作缓冲区中的字节。

MappedByteBuffer 有些特别，在涉及它的专门章节中再讲。



#### Buffer 的分配

---------

要想获得一个Buffer对象首先要进行分配。 

每一个Buffer类都有一个`allocate()`方法。下面是一个分配48字节capacity的ByteBuffer的例子。

```java
ByteBuffer buf = ByteBuffer.allocate(48);
```

这是分配一个可存储1024个字符的CharBuffer：

```java
CharBuffer buf = CharBuffer.allocate(1024);
```



#### 向Buffer中写数据

----------

写数据到Buffer中有两种方式。

- 从Channel中写到Buffer中

  - ```java
    int bytesRead = inChannel.read(buf); //read into buffer.
    ```

- 通过Buffer里面的`put()`方法写到Buffer中

  - ```java
    buf.put(127);
    ```

    put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如， 写到一个指定的位置，或者把一个字节数组写入到Buffer。 更多Buffer实现的细节参考JavaDoc。

  `flip()`方法

  flip方法是把Buffer从写模式切换到读模式，并且将position设置为0，limit设置成之前的position值。

  从读模式转换为写模式，

  

#### 从Buffer读取数据

-----------

从Buffer中读取数据有两种方式：

1. 从Buffer读取数据到Channel。

   - ```java
     	int bytesWritten = inChannel.write(buf);
     ```

2. 使用get()方法从Buffer中读取数据。

   - ```java
     byte aByte = buf.get();
     ```

#### rewind()方法

--------

`Buffer.rewind()`将position设回0，所以你可以**重读**Buffer中的所有数据。

limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

#### clear()与compact()方法

----------

一旦**读完**Buffer中的数据，需要让Buffer准备好再次被写入，可以通过`clear()`或`compact()`方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。

换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法**将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。**limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。



#### mark()与reset()方法

-----------

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如：

```java
buffer.mark();

buffer.reset();  //set position back to mark.
```



#### equals()与compareTo()方法

------------

可以使用equals()和compareTo()方法两个Buffer。

> equals()

当满足下列条件时，表示两个Buffer相等：

1. 有相同的类型（byte、char、int等）。
2. Buffer中剩余的byte、char等的个数相等。
3. Buffer中所有剩余的byte、char等都相同。

如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。

>  compareTo()方法

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：

1. 第一个不相等的元素小于另一个Buffer中对应的元素 。
2. 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

*（译注：剩余元素是从 position到limit之间的元素）*

### Scatter/Gather  -Buffer与Channel之间

------------

Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel中读取或者写入到Channel的操作。

 **分散（`scatter`）**从`Channel`中读取是指在读操作时将读取的数据写入多个`buffer`中。因此，`Channel`将从`Channel`中读取的数据“**分散（`scatter`）**”到多个`Buffer`中。

 聚集（`gather`）写入`Channel`是指在写操作时将多个`buffer`的数据写入同一个`Channel`，因此，`Channel`将多个Buffer中的数据“聚集（`gather`）”后发送到`Channel`。

`scatter` / `gather`经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的`buffer`中，这样你可以方便的处理消息头和消息体。



#### **Scattering Reads**

----------

Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述：

[![Java NIO: Scattering Read](http://ifeve.com/wp-content/uploads/2013/06/scatter.png)](http://ifeve.com/java-nio-scattergather/scatter/)

代码实例如下：

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

**这里看到是从Channel中读入到Buffer中。**

注意buffer首先被插入到数组，然后再将数组作为channel.read()  的输入参数。

read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。



**Scattering  Reads在移动下一个buffer前，`必须填满`当前的buffer，这也意味着它不适用于动态消息(注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。**

#### **Gathering Writes**

-----------

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述：

[![Java NIO: Gathering Write](http://ifeve.com/wp-content/uploads/2013/06/gather.png)](http://ifeve.com/java-nio-scattergather/gather/)

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
//write data into buffer
//数组中的顺序很重要
ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

看起来和上面读入的时候几乎一模一样。

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到`channel`，注意只有position和limit之间的数据才会被写入。

因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到`channel`中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

### 通道之间的信息传输

-------------

**在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel传输到另外一个channel。**

#### **transferFrom()**

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中

（注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。

下面是一个简单的例子：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();
toChannel.transferFrom(position, count, fromChannel);
```

方法的输入参数**position**表示从position处开始向目标文件写入数据，**count**表示最多传输的字节数。如果源通道的剩余空间小于 **count** 个字节，**则所传输的字节数要小于请求的字节数**。
 此外要注意，在`SoketChannel`的实现中，`SocketChannel`只会传输此刻准备好的数据（可能不足count字节）。

因此，`SocketChannel`可能不会将请求的所有数据(count个字节)全部传输到`FileChannel`中。

#### **transferTo()**

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();
fromChannel.transferTo(position, count, toChannel);
```

是不是发现这个例子和前面那个例子特别相似？

除了调用方法的FileChannel对象不一样外，其他的都一样。上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。

SocketChannel会一直传输数据直到目标buffer被填满。

### Selector

-----------------

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。



#### 为什么使用Selector?

----------

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。

**事实上，可以只用一个线程处理所有的通道!**

对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。

下面是单线程使用一个Selector处理3个channel的示例图：

![Java NIO: Selectors](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

#### Selector的创建

----------

为了将**Channel**和**Selector**配合使用，必须将**channel**注册到selector上。

通过`SelectableChannel.register()`方法来实现，如下：

```java
//设置通道为非阻塞类型
channel.configureBlocking(false);
//注册channel到Selector上
SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

> 与Selector一起使用时，Channel必须处于非阻塞模式下。

这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。

而套接字通道都可以。



注意register()方法的第二个参数。

这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。

可以监听四种不同类型的事件：

1. Connect
2. Accept
3. Read
4. Write

通道触发了一个事件也就说明这个通道已经就绪。

1. 成功连接另一个服务器称为：“连接就绪”
2. 一个server socket channel 称为"接收就绪"
3. 一个 channel 已经准备好去读取，叫“读取就绪”
4. 一个channel已经准备好去写，叫做“写入就绪”

这四个事件在使用4个`SelectionKey`来表示:

1. `SelectionKey.OP_CONNECT`
2. `SelectionKey.OP_ACCEPT`
3. `SelectionKey.OP_READ`
4. `SelectionKey.OP_WRITE`

如果你想注册不止一个事件，那么你可以像下面这样写(使用位或连接符)：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;  
```



#### SelectionKey

-----------

在上一小节中，当向Selector注册Channel时，`register()`方法会返回一个`SelectionKey`对象。

这个对象包含了一些你感兴趣的属性：

- The interest set
- The ready set
- The Channel
- The Selector
- An attached object (optional)  附加选项

接下来介绍一下这些属性

> Interest Set

通过这个Interest Set属性我们可以知道这个你注册的时候的事件，通过下面这样的方式实现：

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;   
```

通过“位与操作” 我们就可以知道那些事件是被我们选择了的。

> Ready Set

这个属性是表明，我们的channel是否把我们注册的事件准备好了的。

我们可以通过下面这个方法来获取。

```java
int readySet = selectionKey.readyOps();
```

我们也可以通过下面的四个方法来判断selectionKey有那些已经成功准备好了。

```java
selectionKey.isAcceptable();//接收就绪
selectionKey.isConnectable();//连接就绪
selectionKey.isReadable();
selectionKey.isWritable();
```

> Channel + Selector

获取Channel和Selector从`SelectionKey`是很简单的，这样子做：

```java\
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();    
```

> ### Attaching Objects

你可以通过给`SelectionKey`来附加object对象来附加信息。这样可以帮助我们很方面的识别一个`Channel`或者添加其他什么信息到通道上。例如你可能附加一个`Buffer`到你使用的Channel上或者一个object包含集合信息的。只需要像下面这样做：

```java
selectionKey.attach(theObject);

Object attachedObj = selectionKey.attachment();
```

还可以在用`register()`方法向`Selector`注册`Channel`的时候附加对象。如：

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

这种比较方便把



#### 通过Selector选择通道

-------------

一旦向Selector注册了一个或多个通道，就可以调用几个重载的select()方法。

这些方法返回你所需要的事件（如连接、接受、读或写）已经准备就绪的那些通道。

换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

```java

    int select()
    int select(long timeout)
    int selectNow()

```

`select()`将会阻塞直到至少有一个Channel已经准备就绪好了你注册的事件。

`select(long timeout)`和`Select()`一样但是他会等到超过你所设置的毫秒之后就继续前进了。

`selectNow()不会阻塞，会立马返回，如果没有符合条件的通道则会返回0；

这个方法返回的int值告诉我们有多少通道准备好了你所注册的事件【**自上次调用select()方法后有多少通道变成就绪状态**】。例如你这一次调用的时候有一个就绪了，就会返回一，但是下一次调用的时候又有一个通道的就绪了，但是上一个通道的就绪你还没有处理，他还是只会返回1。



#### selectedKeys()

---------------

一旦调用了`select()`方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的`selectedKeys()`方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
//这里返回的是已经准备就绪的SelectionKey
Set<SelectionKey> selectedKeys = selector.selectedKeys();    
```

当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的`selectedKeySet()`方法访问这些对象。

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    //一般处理完成一个就绪的通道之后就直接移出
    keyIterator.remove();
}
```

通过迭代器循环遍历SelectionKey，然后测试每一个SelectionKey是否已经准备好了，如果准备好了就执行相应的操作。

注意到这里`KeyIterator.remove()`在每一次循环都被调用了。

`Selector`不会自己从已选择键集(Set)中移除`SelectionKey`实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会**再次**将其放入已选择键集中。

>  SelectionKey.channel()方法返回的通道需要转型成你要处理的类型
>
> 如ServerSocketChannel或SocketChannel等。



#### wakeUp()

-----------

某个线程调用`select()`方法后阻塞了，即使没有通道已经就绪，也有办法让其从`select()`方法返回。只要让**其它线程**在第一个线程调用`select()`方法的**那个对象上**调用`Selector.wakeup()`方法即可。阻塞在`select()`方法上的线程会立马返回。

如果有其它线程调用了`wakeup()`方法，但当前没有线程阻塞在`select()`方法上，下个调用`select()`方法的线程会**立即**“醒来（wake up）”。



#### close()

----------

用完`Selector`后调用其`close()`方法会关闭该Selector，且使注册到该Selector上的**所有**`SelectionKey`实例无效。

**通道本身并不会关闭。**



#### 完整的示例

-----------

这里有一个完整的示例，打开一个Selector，注册一个通道到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件（接受，连接，读，写）是否就绪。

```java
//打开一个Selector
Selector selector = Selector.open();
//设置通道未非阻塞
channel.configureBlocking(false);
//将通道注册到Select，并且是对读就绪感兴趣
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

while(true) {
//不会阻塞的返回当前已经就绪的通道
  int readyChannels = selector.selectNow();
//如果没有则跳过
  if(readyChannels == 0) continue;

//通过selector获取SelectionKey的set集合
  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
//遍历集合
  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
      //在循环最后手动去移出SelectionKey，因为如果就绪会自己又被加入进来，而Selector不会自己去主动移出这个SelectionKey
    keyIterator.remove();
  }
}
```

### FileChannel

----------

Java NIO中的`FileChannel`是一个连接到文件的通道。可以通过文件通道读写文件。

> FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。



#### 打开FileChannel

-------

在使用FileChannel之前，必须先打开它。

但是，我们无法直接打开一个`FileChannel`，需要通过使用一个`InputStream`、`OutputStream`或`RandomAccessFile`来获取一个`FileChannel`实例。下面是通过`RandomAccessFile`打开`FileChannel`的示例：

```java
RandomAccessFile aFile  = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```



#### 从FileChannel读取数据

----------

调用多个`read()`方法之一从FileChannel中读取数据。如：

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

1. 首先，**分配**一个`Buffer`。从FileChannel中读取的数据将被**读到**Buffer中。
2. 然后，调用`FileChannel.read()`方法。

该方法将数据从FileChannel读取到Buffer中。

*`read()`方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1，表示到了文件末尾。*

#### 向FileChannel写数据

------------

使用`FileChannel.write()`方法向`FileChannel`写数据，该方法的**参数**是一个Buffer。如：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

*注意FileChannel.write()是在while循环中调用的。*

因为**无法保证**`write()`方法一次能向FileChannel写入多少字节，因此需要重复调用`write()`方法，直到`Buffer`中已经没有尚未写入通道的字节。

#### 关闭FileChannel

-------

用完FileChannel后必须将其关闭。如：

```java
channel.close();    
```

#### FileChannel的position方法

---------

有时可能需要在FileChannel的**某个特定位置**进行数据的读/写操作。

- 可以通过调用`position()`方法获取`FileChannel`的当前位置。

- 也可以通过调用`position(long pos)`方法设置`FileChannel`的当前位置。

```java
long pos channel.position();
channel.position(pos +123);
```

- 如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 (*文件结束标志*)。
- 如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“**文件空洞**”，磁盘上物理文件中写入的数据间有空隙。

#### FileChannel的size方法

----------

**FileChannel实例的size()方法将返回该实例所关联文件的大小。**如:

```java
long fileSize = channel.size();
```



#### FileChannel的truncate方法

------------

可以使用`FileChannel.truncate()`方法截取一个文件。

截取文件时，文件将中指定长度**后面的部分**将被删除。如：

```java
channel.truncate(1024);
```

*这个例子截取文件的前1024个字节.*

#### FileChannel的force方法

------------

`FileChannel.force()`方法将通道里尚未写入磁盘的数据强制写到磁盘上。

出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会**即时**写到磁盘上。

要保证这一点，需要调用`force()`方法。

`force()`方法有一个`boolean`类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上：

```java
channel.force(true);
```

### SocketChannel

----------

Java NIO中的`SocketChannel`是一个连接到**TCP**网络套接字的通道。

可以通过以下**2**种方式创建`SocketChannel`：

1. 打开一个`SocketChannel`并连接到互联网上的某台服务器。（发送）
2. 一个新连接到达`ServerSocketChannel`时，会创建一个`SocketChannel`。(接收)

#### 打开 SocketChannel

---------------

下面是SocketChannel的打开方式：

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

#### 关闭 SocketChannel

-------------

当用完SocketChannel之后调用`SocketChannel.close()`关闭SocketChannel：

```java
socketChannel.close();
```

#### 从 SocketChannel 读取数据

--------------

要从`SocketChannel`中读取数据，调用一个`read()`的方法之一。以下是例子：

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = socketChannel.read(buf);
```

1. 首先，分配一个Buffer。从SocketChannel读取到的数据将会放到这个Buffer中。
2. 然后，调用SocketChannel.read()。

该方法将数据从SocketChannel 读到Buffer中。read()方法返回的int值表示读了多少字节进Buffer里。如果返回的是-1，表示已经读到了流的末尾（**连接关闭了**）。

#### 写入 SocketChannel

-------------

写数据到SocketChannel用的是`SocketChannel.write()`方法，该方法以一个**Buffer作为参数**。示例如下：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

Notice how the `SocketChannel.write()` method is called inside a while-loop.    There is no guarantee of how many bytes the `write()` method writes to the `SocketChannel`.    Therefore we repeat the `write()` call until the `Buffer` has no further bytes to write.

#### 非阻塞模式

------------

可以设置 `SocketChannel` 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用`connect()`, `read()` 和`write()`了。

> connect()

如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用`finishConnect()`的方法。像这样：

```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...    
}
```

> write()

非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在**循环中**调用write()。前面已经有例子了，这里就不赘述了。

> read()

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。

#### 非阻塞模式与Selector

-------------

非阻塞模式与选择器搭配会工作的更好，通过将一或多个`SocketChannel`注册到`Selector`，可以询问选择器哪个通道已经准备好了读取，写入等。`Selector`与`SocketChannel`的搭配使用会在后面详讲。

### ServerSocketChannel

---------------

Java NIO中的 ServerSocketChannel 是一个可以**监听新进来的TCP连接**的通道, 就像标准IO中的ServerSocket一样。

ServerSocketChannel类在 java.nio.channels包中。

这里有个例子：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```



#### 打开 ServerSocketChannel

----------

通过调用 `ServerSocketChannel.open()` 方法来打开ServerSocketChannel.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

#### 关闭 ServerSocketChannel

----------

通过调用`ServerSocketChannel.close() `方法来关闭ServerSocketChannel. 如：

```java
serverSocketChannel.close();
```

#### 监听新进来的连接

-----------

通过 `ServerSocketChannel.accept()` 方法监听新进来的连接。

- 当 `accept()`方法返回的时候,它返回一个包含新进来的连接的 `SocketChannel`。
- 因此, accept()方法会**一直阻塞**到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 `accept()`方法. 如下面的例子：

```java
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

*当然,也可以在while循环中使用除了true以外的其它退出准则。*

#### 非阻塞模式

**ServerSocketChannel可以设置成非阻塞模式。**

在非阻塞模式下，`accept() `方法会立刻返回，如果还没有新进来的连接,返回的将是`null`。

因此，需要检查返回的`SocketChannel`是否是`null`.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
        }
}

```

### 非阻塞服务器设计

-------------

即使你知道Java NIO  非阻塞的工作特性（如Selector,Channel,Buffer等组件），但是想要设计一个非阻塞的服务器仍然是一件**很困难**的事。

一个非阻塞式IO管道是由各个处理非阻塞式IO组件组成的链。其中包括读/写IO。下图就是一个简单的非阻塞式IO管道组成：

![non-blocking-server-1](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-1.png)

一个组件使用 `Selector `监控 `Channel `什么时候有可读数据。然后这个组件读取输入并且根据输入生成相应的输出。最后输出将会再次写入到一个`Channel`中。

一个非阻塞式IO管道不需要将读数据和写数据都包含，有一些管道可能只会读数据，另一些可能只会写数据。

上图仅显示了一个单一的组件。一个非阻塞式IO管道可能拥有超过一个以上的组件去处理输入数据。一个非阻塞式管道的长度是由他的所要完成的任务决定。

一个非阻塞IO管道可能同时读取多个Channel里的数据。举个例子：从多个SocketChannel管道读取数据。

#### **非阻塞式**vs. **阻塞式管道**

-----------

非阻塞和阻塞IO管道两者之间最大的区别在于他们如何从底层`Channel`(Socket或者file)读取数据。

- IO管道通常从流中读取数据（来自socket或者file）并且将这些数据拆分为一系列连贯的消息。
- 这和使用tokenizer（这里估计是解析器之类的意思）将数据流解析为token（这里应该是数据包的意思）类似。

相反你只是将数据流分解为更大的消息体。

我将拆分数据流成消息这一组件称为“消息读取器”（Message Reader）下面是Message Reader拆分流为消息的示意图：

![A Message Reader breaking a stream into messages.](http://tutorials.jenkov.com/images/java-nio/non-blocking-server-2.png)

一个**阻塞IO**管道可以使用类似`InputStream`的接口每次一个字节地从底层`Channel`读取数据，并且这个接口**阻塞**直到有数据可以读取。这就是阻塞式Message Reader的实现过程。

使用阻塞式IO接口简化了Message Reader的实现。阻塞式Message Reader从不用处理在流没有数据可读的情况，或者是只从流中读取部分消息，稍后需要恢复消息解析的情况。

同样，阻塞式Message Writer(一个将数据写入流中组件)也从不用处理只有部分数据被写入和写入消息要延迟恢复的情况。

#### **阻塞式IO管道的缺陷**

-----------

虽然阻塞式Message Reader容易实现，但是也有一个不幸的缺点：

- **每一个要分解成消息的流都需要一个独立的线程。**

必须要这样做的理由是每一个流的IO接口会阻塞，直到它有数据读取。这就意味着一个单独的线程是无法尝试从一个没有数据的流中读取数据转去读另一个流。一旦一个线程尝试从一个流中读取数据，那么这个线程将会阻塞直到有数据可以读取。

如果IO管道是必须要处理大量并发链接服务器的一部分的话，那么服务器就需要为每一个链接维护一个线程。对于任何时间都只有几百条并发链接的服务器这确实不是什么问题。但是如果服务器拥有`百万级别`的并发链接量，这种设计方式就没有良好收放。每个线程都会占用栈32bit-64bit的内存。所以`一百万`个线程占用的内存将会达到**1TB**！不过在此之前服务器将会把所有的内存用以处理传经来的消息（例如：分配给消息处理期间使用对象的内存）

为了将线程数量降下来，许多服务器使用了服务器维持线程池（例如：常用线程为100）的设计，从而一次一个地从入站链接（inbound  connections）地读取。入站链接保存在一个**队列**中，线程按照进入队列的顺序处理入站链接。这一设计如下图所示：(*注：`Tomcat`就是这样的*)

![non-blocking-server-3](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-3.png)

然而，这一设计需要入站链接合理地发送数据。

如果入站链接长时间不活跃，那么大量的不活跃链接实际上就造成了线程池中所有线程阻塞。这意味着服务器响应变慢甚至是没有反应。

一些服务器尝试通过**弹性控制线程池的核心线程数量**这一设计减轻这一问题。

例如，如果线程池线程不足时，线程池可能开启更多的线程处理请求。这一方案意味着需要大量的长时链接才能使服务器不响应。但是记住，对于并发线程数仍然是有一个上限的。因此，这一方案仍然无法很好地解决一百万个长时链接。

#### **基础非阻塞式IO管道设计**

-------------



一个非阻塞式IO管道可以使用一个单独的线程向**多个流**读取数据。

这需要流可以被切换到`非阻塞模式`。

在非阻塞模式下，当你读取流信息时可能会返回0个字节或更多字节的信息。如果流中没有数据可读就返回0字节，如果流中有数据可读就返回1+字节。

为了避免检查没有可读数据的流我们可以使用`Java NIO Selector`. 一个或多个`SelectableChannel `实例可以同时被一个`Selector`注册。简单的来说就是一个`Selector`可以有多个`Channel`。

当你调用Selector的`select()`或者`selectNow() `方法它只会返回**有数据读取**的`SelectableChannel`的实例。下图是该设计的示意图：

![A component selecting channels with data to read.](http://tutorials.jenkov.com/images/java-nio/non-blocking-server-4.png)

#### **读取部分消息**

-----------

当我们从一个`SelectableChannel`读取一个数据包时，我们不知道这个数据包相比于源文件是否有丢失或者重复数据。

一个数据包可能的情况有：

1. 缺失数据（比原有消息的数据少）
2. 与原有一致
3. 比原来的消息的数据更多（例如：是原来的1.5或者2.5倍）

数据包可能出现的情况如下图所示：

![A data block can contain less than or more than a single message.](http://tutorials.jenkov.com/images/java-nio/non-blocking-server-5.png)

在处理类似上面这样部分信息时，有两个问题:

1. **判断**你是否能在数据包中获取完整的消息。
2. 在其余消息到达之前如何**处理已到达的部分消息**。

判断消息的完整性需要消息读取器（Message Reader）在数据包中寻找是否存在至少一个完整消息体的数据。如果一个数据包包含一个或多个完整消息体，这些消息就能够被发送到管道进行处理。寻找完整消息体这一处理可能会重复多次，因此这一操作应该尽可能的快。

**判断消息完整性**和**存储部分消息**都是消息读取器(Message Reader)的责任。

为了避免混合来自不同`Channel`的消息，我们将对每一个`Channel`使用一个Message Reader。设计如下图所示:

![non-blocking-server-6](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-6.png)

在从`Selector`得到可从中读取数据的`Channel`实例之后, 与该`Channel`相关联的`Message Reader`读取数据并尝试将他们分解为消息。这样读出的任何完整消息可以被传到读取通道(read pipeline)任何需要处理这些消息的组件中。

一个`Message Reader`一定满足特定的协议。`Message  Reader`需要知道它尝试读取的消息的消息格式。如果我们的服务器可以通过协议来复用，那它需要有能够插入`Message Reader`实现的功能 – 可能通过接收一个**Message Reader工厂**作为配置参数。

#### **存储部分消息**

----------

现在我们已经确定Message Reader有责任存储部分消息，直到收到完整的消息，我们需要弄清楚这些部分消息的存储应该如何实现。

有两个设计因素我们要考虑：

1. 我们想**尽可能少**地复制消息数据。复制越多，性能越低。
2. 我们希望将完整的消息存储在**连续的字节序列**中，使解析消息更容易。

> **每个Message Reader的缓冲区**

很显然部分消息需要存储某些缓冲区中。

简单的实现方式可以是每一个Message  Reader内部简单地有一个缓冲区。

但是这个缓冲区应该多大？

它要大到足够储存最大允许储存消息。

因此，如果**最大允许储存消息**是1MB，那么Message Reader内部缓冲区将至少需要1MB。

当我们的链接达到百万数量级，每个链接都使用1MB并没有什么作用。1,000,000 * 1MB仍然是1TB的内存！那如果最大的消息是16MB甚至是128MB呢？

> 大小可调的缓冲区

另一个选择是在Message Reader内部实现一个大小可调的缓冲区。大小可调的缓冲区开始的时候很小，如果它获取的消息过大，那缓冲区会扩大。这样每一条链接就不一定需要如1MB的缓冲区。每条链接的缓冲区只要需要足够储存下一条消息的内存就行了。

> 通过复制调整大小

实现可调大小缓冲区的第一种方式是从一个大小(例如:4KB)的缓冲区开始。如果4KB的缓冲区装不下一个消息，则会分配一个更大的缓冲区(如:8KB),并将大小为4KB的缓冲区数据复制到这个更大的缓冲区中去。

- 通过复制实现大小可调缓冲区的优点在于消息的所有数据被保存在一个连续的字节数组中，这就使得消息的解析更加容易。
- 它的缺点就是在复制更大消息的时候会导致大量的数据。

为了减少消息的复制，你可以分析流进你系统的消息的大小，并找出尽量减少复制量的缓冲区的大小。例如，你可能看到大多数消息都小于4KB，这是因为它们都仅包含很小的 request/responses。这意味着缓冲区的初始值应该设为4KB。

然后你可能有一个消息大于4KB，这通常是因为它里面包含一个文件。你可能注意到大多数流进系统的文件都是小于128KB的。这样第二个缓冲区的大小设置为128KB就较为合理。

最后你可能会发现一旦消息超过128KB之后，消息的大小就没有什么固定的模式，因此缓冲区最终的大小可能就是最大消息的大小。

上面三种缓冲区大小可以减少数据的复制。小于4KB的消息将不会复制。对于一百万个并发链接其结果是：1,000,000 * 4KB = 4GB，对于目前大多数服务器还是有可能的。

介于4KB –  128KB的消息将只会复制一次，并且只有4KB的数据复制进128KB的缓冲区中。介于128KB至最大消息大小的消息将会复制两次。第一次复制4KB，第二次复制128KB，所以最大的消息总共复制了132KB。假设没有那么多超过128KB大小的消息那还是可以接受的。

一旦消息处理完毕，那么分配的内存将会被清空。这样在同一链接接收到的下一条消息将会再次从最小缓冲区大小开始算。这样做的必要性是确保了不同连接间内存的有效共享。所有的连接很有可能在同一时间并不需要大的缓冲区。

> 通过追加调整大小

调整缓冲区大小的另一种方法是使缓冲区由多个数组组成。当你需要调整缓冲区大小时，你只需要另一个字节数组并将数据写进去就行了。

这里有两种方法扩张一个缓冲区。一个方法是分配单独的字节数组，并将这些数组保存在一个列表中。另一个方法是分配较大的共享字节数组的片段，然后保留分配给缓冲区的片段的列表。就个人而言，我觉得片段的方式会好些，但是差别不大。

通过追加单独的数组或片段来扩展缓冲区的优点在于写入过程中不需要复制数据。所有的数据可以直接从socket (Channel)复制到一个数组或片段中。

以这种方式扩展缓冲区的缺点是在于数据不是存储在单独且连续的数组中。这将使得消息的解析更困难，因为解析器需要同时查找每个单独数组的结尾处和所有数组的结尾处。由于你需要在写入的数据中查找消息的结尾，所以该模型并不容易使用。



#### **写部分数据**

------------

在非阻塞IO管道中写数据仍然是一个挑战。当你调用一个处于非阻塞式`Channel`对象的`write(ByteBuffer)`方法时，ByteBuffer写入多少数据是无法保证的。write（ByteBuffer）方法会返回写入的字节数，因此可以跟踪写入的字节数。这就是挑战：跟踪部分写入的消息，以便最终可以发送一条消息的所有字节。

为了管理部分消息写入Channel，我们将创建一个消息写入器`（Message Writer）`。就像Message Reader一样，每一个要写入消息的Channel我们都需要一个Message Writer。在每个Message Writer中，我们跟踪正在写入的消息的字节数。

如果达到的消息量超过Message Writer可直接写入Channel的消息量，消息就需要在Message Writer排队。然后Message Writer尽快地将消息写入到Channel中。

下图是部分消息如何写入的设计图：

![non-blocking-server-8](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-8.png)

为了使Message Writer能够尽快发送数据，Message Writer需要能够不时被调用，这样就能发送更多的消息。

如果你又大量的连接那你将需要大量的Message Writer实例。检查Message  Writer实例(如:一百万个)看写任何数据时是否缓慢。 首先，许多Message  Writer实例都没有任何消息要发送，我们并不想检查那些Message  Writer实例。其次，并不是所有的Channel实例都可以准备好写入数据。 我们不想浪费时间尝试将数据写入无法接受任何数据的Channel。

为了检查Channel是否准备好进行写入，您可以使用Selector注册Channel。然而我们并不想将所有的Channel实例注册到Selector中去。想象一下，如果你有1,000,000个连接且其中大多是空闲的，并且所有的连接已经与Selector注册。然后当你调用select()时，这些Channel实例的大部分将被写入就绪（它们大都是空闲的，记得吗？）然后你必须检查所有这些连接的Message Writer，以查看他们是否有任何数据要写入。

为了避免检查所有消息的Message Writer实例和所有不可能被写入任何信息的Channel实例，我们使用这两步的方法：

1. **当一个消息被写入Message Writer，Message Writer向Selector注册其相关Channel（如果尚未注册）。**
2. 当你的服务器有时间时，它检查Selector以查看哪些注册的Channel实例已准备好进行写入。  对于每个写就绪Channel，请求其关联的Message Writer将数据写入Channel。 如果Message  Writer将其所有消息写入其Channel，则Channel将再次从Selector注册。

这两个小步骤确保了有消息写入的Channel实际上已经被Selector注册了。

#### **汇总**

-----------

正如你所见，一个非阻塞式服务器需要时不时检查输入的消息来判断是否有任何的新的完整的消息发送过来。服务器可能会在一个或多个完整消息发来之前就检查了多次。检查一次是不够的。

同样，一个非阻塞式服务器需要时不时检查是否有任何数据需要写入。如果有，服务器需要检查是否有任何相应的连接准备好将该数据写入它们。只有在第一次排队消息时才检查是不够的，因为消息可能被部分写入。

所有这些非阻塞服务器最终都需要定期执行的三个“管道”（pipelines）：:

- 读取管道（The read pipeline），用于检查是否有新数据从开放连接进来的。
- 处理管道(The process pipeline)，用于所有任何完整消息。
- 写入管道（The write pipeline），用于检查是否可以将任何传出的消息写入任何打开的连接。

这三条管道在循环中重复执行。你可能可以稍微优化执行。例如，如果没有排队的消息可以跳过写入管道。 或者，如果我们没有收到新的，完整的消息，也许您可以跳过处理管道。

以下是说明完整服务器循环的图：

![non-blocking-server-9](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-9.png)

#### **服务器线程模型**

----------

GitHub资源库里面的非阻塞式服务器实现使用了两个线程的线程模式。第一个线程用来接收来自ServerSocketChannel的传入连接。第二个线程处理接受的连接，意思是读取消息，处理消息并将响应写回连接。这两个线程模型的图解如下：

![non-blocking-server-10](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-10.png)

上一节中说到的服务器循环处理是由处理线程（Processor Thread）执行。

### NIO中的零拷贝

零拷贝的意思是 没有cpu参与的拷贝的意思。

当前零拷贝的实现方法有两种

- MemeryMap（mmap）
- sendFile

>  传统IO流程

![1575855435947](https://images.cnblogs.com/cnblogs_com/ronnieyuan/1607877/o_1912090242471575855435947.png)

- 传统IO存在 4次拷贝 、3次状态转换。
- cpu参与了拷贝的过程。

> mmap内存映射方式

mmap是内存映射的方式来实现的。

**通过直接在操作系统内核里面，用户空间共享内核空间来减少拷贝。**

![1575855870723](https://images.cnblogs.com/cnblogs_com/ronnieyuan/1607877/o_1912090242591575855870723.png)

- 3次拷贝，3次状态转换。
- 不用从系统内核区复制到用户内核区，直接拷贝到socketBuffer里面
- 不是真正意义上面的零拷贝，依然存在cpu参与的 拷贝

> linux 2.1 里面 的sendFile

![1575856449573](https://images.cnblogs.com/cnblogs_com/ronnieyuan/1607877/o_1912090243091575856449573.png)

- 2次状态的转换，3次拷贝
- 数据不经过用户态，直接从内核缓冲区进入Socket Buffer ，同时，由于和用户态完全无关就减少了一次上下文切换。
- 但是依然存在一次cpu拷贝。

> linux2.4 优化sendFile过后

![1575856952257](https://images.cnblogs.com/cnblogs_com/ronnieyuan/1607877/o_1912090243191575856952257.png)

- 2次状态转换 基本上去除了cpu拷贝的过程。
- 仅仅有少量的信息需要cpu拷贝一些 length，offset，消耗很低可忽略不计。
- 2次拷贝。

> 什么时候使用mmap，什么时候使用 sendfile

sendfile 在nio中使用transforTo方法来实现。可以在两个通道直接直接进行拷贝。

但是限制是一次只能传递 8m大小的文件。如果传递大文件需要计算传递次数，和每次发送的偏移量。适合传输大文件。

mmap适合小文件的快速改写。