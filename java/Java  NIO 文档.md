# Java  NIO 文档

## 一、简介

​		Java NIO (New IO)是Java的另一种IO API(源自Java 1.4)，这意味着可以替代标准Java IO和Java网络API。Java NIO提供了一种不同于标准IO API的处理IO的方法。

​		Java NIO由以下核心组件组成:	

- 通道

- 缓冲区

- 选择器

  Java NIO有更多的类和组件，但是在我看来，通道、缓冲区和选择器构成了API的核心。其他组件，如Pipe和FileLock，只是与三个核心组件一起使用的实用程序类。因此，我将在NIO概述中重点介绍这三个组件。

### 1.1、Java NIO:通道和缓冲区

​		在标准的IO API中，可以使用字节流和字符流。在NIO中，您使用通道和缓冲区。数据总是从通道读入缓冲区，或者从缓冲区写入通道。

​		通常，NIO中的所有IO都从一个通道开始。通道有点像流。从通道数据可以读入缓冲区。数据也可以从缓冲区写入通道。下面是一个例子:

![1556343514969](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556343514969.png)

有几种通道和缓冲区类型。以下是Java NIO中主要通道实现的列表:

- FileChannel

- DatagramChannel

- SocketChannel

- ServerSocketChannel

可以看到，这些通道包括UDP + TCP网络IO和文件IO。

这些类还附带了一些有趣的接口，但是为了简单起见，我将不介绍这些接口。它们将在本Java NIO教程的其他文本中进行相关解释。

以下是Java NIO的核心缓冲区实现列表:

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些缓冲区涵盖了可以通过IO发送的基本数据类型:字节、短、整型、长、浮点型、双型和字符。

Java NIO还有一个MappedByteBuffer，它与内存映射文件一起使用。不过，我将把这个缓冲区排除在概述之外。

### 1.2、Java NIO:非阻塞IO

​		Java NIO使您能够执行非阻塞IO。例如，线程可以请求通道将数据读入缓冲区。当通道将数据读入缓冲区时，线程可以执行其他操作。一旦数据读入缓冲区，线程就可以继续处理它。将数据写入通道也是如此。

### 1.3、Java NIO:选择器

​		Java NIO包含“选择器”的概念。选择器是一个对象，它可以监视多个通道的事件(如:打开的连接、到达的数据等)。因此，一个线程可以监视多个通道的数据。

​	选择器允许一个线程处理多个通道。如果您的应用程序打开了许多连接(通道)，但每个连接上的流量都很低，那么这将非常方便。例如，在聊天服务器中。

下面是一个线程使用选择器处理3个通道的例子:

![1556344874727](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556344874727.png)



要使用选择器，您需要向它注册频道。然后调用它的select()方法。此方法将阻塞，直到为已注册的通道之一准备好事件为止。一旦方法返回，线程就可以处理这些事件。事件的例子有传入连接、接收的数据等。

## 二、Java NIO通道

​	Java NIO通道与流类似，但有一些区别:

- 您可以对通道进行读写。流通常是单向的(读或写)。
- 通道可以异步读取和写入。
- 通道总是读写缓冲区。

### 2.1、通道的实现

以下是Java NIO中最重要的通道实现:

- FileChannel

- DatagramChannel

- SocketChannel

- ServerSocketChannel

FileChannel从文件中读取数据并将数据读入文件。

DatagramChannel可以通过UDP在网络上读写数据。

SocketChannel可以通过TCP在网络上读写数据。

ServerSocketChannel允许您侦听传入的TCP连接，就像web服务器一样。为每个传入连接创建一个套接字通道。

### 2.2、基础的通道例子

```java
//创建一个文件读取 
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
//获取文件通道
FileChannel inChannel = aFile.getChannel();
//获取缓冲区
ByteBuffer buf = ByteBuffer.allocate(48);
//将数据从通道读取到缓存中，返回读取到的字节数
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
	
    System.out.println("Read " + bytesRead);
    //切换到读取模式
    buf.flip();
	//循环读取缓冲区的数据
    while(buf.hasRemaining()){
        System.out.print((char) buf.get());
    }
	//清空缓冲区，其实只是重置指针位置，缓冲区的数据依然存在，看下源码便可知
    buf.clear();
    //继续将通道中的数据读取到缓冲区中
    bytesRead = inChannel.read(buf);
}
//关闭流
aFile.close();
```

注意buf.flip()调用。首先读入缓冲区。然后把它翻转过来。然后你读出来。

## 三、Java NIO缓冲区

​		在与NIO通道交互时使用Java NIO缓冲区。如您所知，数据从通道读入缓冲区，然后从缓冲区写入通道。

​		缓冲区本质上是一块内存，您可以将数据写入其中，然后再读取数据。这个内存块被封装在NIO Buffer对象中，该对象提供了一组方法，使使用内存块变得更容易。

### 3.1、基本的缓冲区使用

使用缓冲区读取和写入数据通常遵循以下4个步骤:

- 将数据写入缓冲区

- 调用buffer.flip ()

- 从缓冲区中读取数据

- 调用buffer.clear()或buffer.compact()

当您将数据写入缓冲区时，缓冲区将跟踪您已经写入了多少数据。一旦需要读取数据，就需要使用flip()方法调用将缓冲区从写模式切换到读模式。在读取模式中，缓冲区允许您读取写入缓冲区的所有数据。

读取完所有数据后，需要清除缓冲区，以便再次编写。有两种方法可以做到这一点:调用clear()或调用compact()。方法的作用是:清除整个缓冲区。compact()方法只清除您已经读取的数据。任何未读数据都被移动到缓冲区的开头，现在数据将在未读数据之后写入缓冲区。

下面是一个简单的缓冲区使用示例，其中的写、翻转、读取和清除操作以粗体显示:

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

### 3.2、缓冲容量、位置和极限

缓冲区本质上是一块内存，您可以将数据写入其中，然后再读取数据。这个内存块被封装在NIO Buffer对象中，该对象提供了一组方法，使使用内存块变得更容易。

为了理解缓冲区的工作原理，您需要熟悉缓冲区的三个属性。这些都是:

- capacity
- position
- limit

position和limit的含义取决于缓冲区是处于读模式还是写模式。无论缓冲模式如何，容量总是相同的。

这是读写模式下容量、位置和限制的说明。解释在插图之后的部分。

![1556353050359](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556353050359.png)

#### 3.2.1、Capacity

缓冲区作为内存块，具有一定的固定大小，也称为“容量”。您只能将容量字节、长、字符等写入缓冲区。一旦缓冲区满了，就需要清空它(读取数据或清除数据)，然后才能将更多数据写入缓冲区。

#### 3.2.3、Position

当您将数据写入缓冲区时，您是在特定的位置这样做的。初始位置是0。当一个字节、long等被写入缓冲区时，该位置被提前指向缓冲区中的下一个单元格，以便将数据插入其中。位置可以最大限度地变成capacity - 1。

当从缓冲区读取数据时，也要从给定位置读取数据。当您将缓冲区从写模式切换到读模式时，位置将重置为0。当您从缓冲区读取数据时，您从position读取数据，position将被提前到下一个要读取的位置。

#### 3.2.3、Limit

在写模式下，缓冲区的limit是可以写入缓冲区的数据量的限制。在写模式下，limit等于缓冲区的容量。

当将缓冲区翻转到读取模式时，limit表示您可以从数据中读取多少数据的限制。因此，当将缓冲区切换到读取模式时，将limit设置为写入模式的写入位置。换句话说，您可以读取写入的字节数(限制设置为写入的字节数，由位置标记)。

### 3.2、缓冲类型

Java NIO附带以下缓冲区类型:

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

如您所见，这些缓冲区类型表示不同的数据类型。换句话说，它们让您以char、short、int、long、float或double的形式处理缓冲区中的字节。

MappedByteBuffer有点特殊，将在它自己的文本中介绍。

### 3.3、分配一个缓冲区

要获得缓冲区对象，必须首先分配它。每个缓冲区类都有一个allocate()方法来做这件事。下面是一个例子，展示了一个字节缓冲区的分配，其容量为48字节:

```
ByteBuffer buf = ByteBuffer.allocate(48);
```

下面是一个分配空间为1024个字符的CharBuffer的例子:

```
CharBuffer buf = CharBuffer.allocate(1024);
```

### 3.4、将数据写入缓冲区

你可以用两种方式将数据写入缓冲区:

1. 将数据从通道写入缓冲区
2. 自己通过缓冲区的put()方法将数据写入缓冲区。

下面是一个例子，展示了一个通道如何将数据写入缓冲区:

```
int bytesRead = inChannel.read(buf); //read into buffer.
```

下面是一个通过put()方法将数据写入缓冲区的例子:

```
buf.put(127);    
```

put()方法还有许多其他版本，允许以多种不同的方式将数据写入缓冲区。例如，在特定位置写入，或将字节数组写入缓冲区。有关具体缓冲区实现的详细信息，请参阅JavaDoc。

### 3.5、flip()

flip()方法将缓冲区从写模式切换到读模式。调用flip()将位置设置回0，并设置位置的限制。

换句话说，position现在标记读取位置，limit标记写入缓冲区的字节数、字符数等—可以读取的字节数、字符数等的限制。

### 3.6、从缓冲区读取数据

从缓冲区读取数据有两种方法。

1. 将数据从缓冲区读入通道。
2. 使用get()方法之一自己从缓冲区读取数据。

下面是一个如何从缓冲区读取数据到通道的例子:

```
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```

下面是一个使用get()方法从缓冲区读取数据的例子:

```
byte aByte = buf.get();    
```

get()方法还有许多其他版本，允许您以多种不同的方式从缓冲区读取数据。例如，读取特定位置，或从缓冲区读取字节数组。有关具体缓冲区实现的详细信息，请参阅JavaDoc。

### 3.7、rewind() 重置再次读取缓冲区方法

rewind()将位置设置回0，这样就可以重新读取缓冲区中的所有数据。这个限制保持不变，因此仍然标记可以从缓冲区读取多少元素(字节、字符等)。

### 3.8、clear() and compact() 清空缓冲区

一旦完成了从缓冲区中读取数据，就必须使缓冲区为再次写入做好准备。您可以通过调用clear()或调用compact()来实现这一点。

如果调用clear()，则`position` 将被设置为0，`limit` 设置为capacity值。换句话说，缓冲区被清除。缓冲区中的数据未被清除。只是指示可以将数据写入缓冲区的位置的标记。

如果在调用clear()时缓冲区中有任何未读数据，则该数据将被“遗忘”，这意味着不再有任何标记来表示哪些数据已被读取，哪些数据尚未被读取。

如果缓冲区中仍然有未读数据，并且您希望稍后读取它，但是需要先进行一些写入操作，那么调用compact()而不是clear()。

compact()将所有未读数据复制到缓冲区的开头。然后将position设置为最后一个未读元素之后的正确位置。与clear()一样，limit属性仍然被设置为capacity。现在缓冲区已经准备好进行写入，但是不会覆盖未读数据。

### 3.9、mark() and reset()

您可以通过调用Buffer.mark()方法来标记缓冲区中的给定位置。然后，您可以通过调用Buffer.reset()方法将位置重置回标记的位置。举个例子:

```
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.   
```

### 3.10、equals() and compareTo()

可以使用equals()和compareTo()比较两个缓冲区。

#### 3.10.1、equals()

两个缓冲区是相等的，如果:

1. 它们具有相同的类型(字节、字符、int等)
2. 它们在缓冲区中有相同数量的剩余字节、字符等。
3. 所有剩余的字节、字符等都是相等的。

如您所见，equals只比较缓冲区的一部分，而不是其中的每个元素。实际上，它只是比较缓冲区中的剩余元素。

#### 3.10.2、compareTo()

compareTo()方法比较两个缓冲区中剩余的元素(字节、字符等)，用于例如排序例程。如果满足以下条件则认为一个缓冲区比另一个缓冲区“更小”:

1. 同位置的元素小则更小。
2. 所有的元素都是相等的，但是第一个缓冲区会在第二个缓冲区之前耗尽元素(它的元素更少)。

## 四、Java NIO分散/聚集

Java NIO提供了内置的分散/聚集支持。分散/聚集是用于从通道中读取和从通道中写入的概念。

从通道散射读取是将数据读入多个缓冲区的读取操作。因此，通道将数据从通道“分散”到多个缓冲区。

对通道的收集写操作是将多个缓冲区中的数据写入到单个通道的写操作。因此，通道“收集”来自多个缓冲区的数据到一个通道中。

在处理多方面分散数据传输的情况下，分散/聚集非常有用。例如，如果消息由头和正文组成，则可以将头和正文保存在单独的缓冲区中。这样做可以使您更容易地分别处理header和body。

### 4.1、散射读取

“分散读取”将数据从单个通道读入多个缓冲区。下面是这一原则的一个例子:

下面是散点原理的一个例子:

![1556359252308](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556359252308.png)

下面是一个代码示例，演示了如何执行散射读取:

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

注意，首先将缓冲区插入数组，然后将数组作为参数传递给channel.read()方法。然后read()方法按照缓冲区在数组中出现的顺序从通道中写入数据。一旦缓冲区满了，通道将继续填充下一个缓冲区。

分散读取在进入下一个缓冲区之前会填满一个缓冲区，这意味着它不适合动态调整消息部分的大小。换句话说，如果您有一个头和一个主体，并且头的大小是固定的(例如128字节)，那么分散读取就可以正常工作。

### 4.1、聚集写

“聚集写”将数据从多个缓冲区写到一个通道。下面是这一原则的一个例子:

![1556374717749](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556374717749.png)

下面是一个代码示例，展示了如何执行一个聚集写:

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

缓冲区数组被传递到write()方法中，该方法按数组中遇到的顺序写入缓冲区的内容。只写入缓冲区的位置和限制之间的数据。因此，如果一个缓冲区的容量为128字节，但只包含58字节，则只有58字节从该缓冲区写到通道。因此，与分散读取相比，聚集写入可以很好地处理动态大小的消息部分。

## 五、Java NIO通道到通道的传输

在Java NIO中，如果其中一个通道是FileChannel，则可以直接将数据从一个通道传输到另一个通道。FileChannel类有一个transferTo()和一个transferFrom()方法，它们可以为您完成这一任务。

### 5.1、transferFrom()

transferfrom()方法将数据从源通道传输到FileChannel。下面是一个简单的例子:

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

参数position和count，告诉目标文件中从何处开始写入(position)，以及最大传输多少字节(count)。如果源通道的字节数少于count，那么传输的字节数就会减少。

此外，一些SocketChannel实现可能只传输SocketChannel在其内部缓冲区中已经准备好的数据——即使SocketChannel稍后可能有更多可用数据。因此，它可能不会将请求的全部数据(count)从SocketChannel传输到FileChannel。

### 5.2、transferTo()

transferTo()方法从一个FileChannel传输到另一个通道。下面是一个简单的例子:

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

请注意这个示例与前面的示例有多么相似。唯一真正的区别是方法调用的是哪个FileChannel对象。其余的都是一样的。

transferTo()方法也存在SocketChannel的问题。SocketChannel实现只能从FileChannel传输字节，直到发送缓冲区被填满，然后停止。

## 六、Java NIO选择器

Java NIO选择器是一个组件，它可以检查一个或多个Java NIO通道实例，并确定哪些通道已经准备好，例如读取或写入。通过这种方式，一个线程可以管理多个通道，从而管理多个网络连接。

### 6.1、为什么要使用选择器?

只使用一个线程来处理多个通道的好处是，您需要更少的线程来处理通道。实际上，您可以只使用一个线程来处理所有通道。对于操作系统来说，在线程之间切换非常昂贵，而且每个线程也会占用操作系统中的一些资源(内存)。因此，使用的线程越少越好。

但是请记住，现代操作系统和CPU在多任务处理方面变得越来越好，因此多线程的开销会随着时间的推移变得越来越小。事实上，如果一个CPU有多个内核，那么不进行多任务处理可能会浪费CPU的能量。无论如何，关于设计的讨论属于不同的文本。这里只需说明，您可以使用选择器使用一个线程处理多个通道。

下面是一个线程使用选择器处理3个通道的例子:

![1556375723184](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556375723184.png)

### 6.2、创建一个选择器

您可以通过调用select .open()方法来创建选择器，如下所示:

```
Selector selector = Selector.open();
```

### 6.3、使用选择器注册通道

要使用带有选择器的通道，必须用选择器注册通道。这是使用SelectableChannel.register()方法完成的，如下所示:

```
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

通道必须处于非阻塞模式，才能与选择器一起使用。这意味着您不能将FileChannel与选择器一起使用，因为FileChannel不能切换到非阻塞模式。不过套接字通道可以很好地工作。

注意register()方法的第二个参数。这是一个“interest set”，意思是通过选择器在通道中侦听您感兴趣的事件。你可以收听四个不同的事件:

1. Connect
2. Accept
3. Read
4. Write

一个“触发事件”的通道也被称为为该事件“准备好了”。因此，成功连接到另一个服务器的通道是“connect ready”。接受传入连接的服务器套接字通道是“accept”就绪的。准备读取数据的通道是“read”准备状态。准备好向其写入数据的通道是“write”准备状态。

这四个事件由四个SelectionKey常量表示:

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对不止一个事件感兴趣，或者把常数放在一起，就像这样:

```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;    
```

### 6.4、选择键（SelectionKey）

正如您在前一节中看到的，当您使用选择器注册通道时，register()方法将返回SelectionKey对象。这个SelectionKey对象包含一些有趣的属性:

- The interest set
- The ready set
- The Channel
- The Selector
- An attached object (optional)

我将在下面描述这些属性。

#### 6.4.1、Interest Set（兴趣集）

兴趣集是您对“选择”感兴趣的事件集，如“向选择器注册通道”一节所述。你可以像这样通过SelectionKey读写兴趣集:

```
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;    
```

如您所见，您可以使用给定的SelectionKey常量和兴趣集来确定某个事件是否在兴趣集中。

#### 6.4.2、Ready Set（就绪集）

就绪集是通道准备进行的一组操作。您将主要在选择之后访问就绪集。选择将在后面的部分进行解释。您可以这样访问就绪集:

```
int readySet = selectionKey.readyOps();
```

您可以使用与兴趣集相同的方法测试通道准备用于哪些事件/操作。但是，你也可以用这四种方法来代替，它们都是布尔值:

```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

#### 6.4.3、Channel + Selector（通道+选择器）

从SelectionKey访问通道+选择器非常简单。具体做法如下:

```
Channel  channel  = selectionKey.channel();

Selector selector = selectionKey.selector();   
```

#### 6.4.4、Attaching Objects（附加对象）

您可以将对象附加到SelectionKey上——这是识别给定通道或将进一步信息附加到通道上的方便方法。例如，可以将正在使用的缓冲区附加到通道中，或者附加一个包含更多聚合数据的对象。下面是如何附加对象:

```
selectionKey.attach(theObject);

Object attachedObj = selectionKey.attachment();
```

还可以在register()方法中用选择器注册通道时附加一个对象。它是这样的:

```
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

### 6.5、Selecting Channels via a Selector（通过选择器选择通道）

一旦用选择器注册了一个或多个通道，就可以调用select()方法之一。这些方法返回为您感兴趣的事件(连接、接受、读取或写入)“准备好”的通道。换句话说，如果您对准备读取的通道感兴趣，那么您将从select()方法接收准备读取的通道。

以下是select()方法:

- int select()
- int select(long timeout)
- int selectNow()

select()会阻塞，直到至少有一个通道为您注册的事件准备好。

select(long timeout)与select()执行相同的操作，只是它阻塞超时毫秒(参数)的最大值。

selectNow()根本不会阻塞。它立即返回任何准备好的通道。

select()方法返回的int值告诉我们有多少通道已经准备好了。也就是说，自上次调用select()以来，已经准备好了多少通道。如果您调用select()，它返回1，因为一个通道已经就绪，您再次调用select()，并且一个通道已经就绪，它将再次返回1。如果您没有对第一个就绪的通道执行任何操作，那么现在就有了两个就绪通道，但是在每个select()调用之间只有一个通道已经就绪。

#### 6.5.1、selectedKeys()

一旦您调用了select()方法中的一个，并且它的返回值表明一个或多个通道已经就绪，您就可以通过“selected key set”通过调用selectors selectedKeys()方法来访问就绪通道。它是这样的:

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();   
```

当您使用选择器注册通道时，channel .register()方法将返回SelectionKey对象。此键表示使用该选择器进行通道注册。您可以从SelectionKey通过selectedKeySet()方法访问这些密钥。

您可以迭代这个选定的键集来访问就绪通道。它是这样的:

```
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

    keyIterator.remove();
}
```

这个循环遍历所选键集中的键。对于每个键，它都测试键，以确定键所引用的通道准备用于什么。

请注意在每次迭代结束时调用keyIterator.remove()。选择器不会从所选的键集本身删除SelectionKey实例。你必须这样做，当你处理完通道。下次通道“就绪”时，选择器将再次将其添加到所选的键集中。

channel()方法返回的通道应该转换为您需要使用的通道e。g一个ServerSocketChannel或SocketChannel等。

#### 6.5.2、wakeUp()

如果线程调用了阻塞的select()方法，则可以让select()方法保留，即使还没有通道。这是通过让另一个线程调用选择器上的select .wakeup()方法来实现的，第一个线程在这个选择器上调用select()。在select()中等待的线程将立即返回。

如果另一个线程调用了wakeup()，而select()中当前没有阻塞任何线程，则调用select()的下一个线程将立即“唤醒”。

#### 6.5.3、close()

当您完成选择器时，您将调用它的close()方法。这将关闭选择器并使所有使用该选择器注册的SelectionKey实例无效。通道本身并没有关闭。

### 6.6、完整的选择器的例子

下面是一个完整的示例，它打开一个选择器，用它注册一个通道(省略通道实例化)，并持续监视选择器，以确定四个事件(accept、connect、read、write)的“准备就绪”。

```java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.selectNow();

  if(readyChannels == 0) continue;


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

    keyIterator.remove();
  }
}
```

## 七、Java NIO FileChannel

Java NIO FileChannel是连接到文件的通道。使用文件通道，您可以从文件中读取数据，并将数据写入文件。Java NIO FileChannel类是NIO使用标准Java IO API读取文件的替代方法。

无法将FileChannel设置为非阻塞模式。它总是在阻塞模式下运行。

### 7.1、打开一个FileChannel

在使用FileChannel之前，必须先打开它。不能直接打开FileChannel。您需要通过InputStream、OutputStream或RandomAccessFile获得一个FileChannel。下面是如何通过RandomAccessFile打开FileChannel:

```java
RandomAccessFile aFile     = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel      inChannel = aFile.getChannel();
```

### 7.2、从FileChannel读取数据

要从FileChannel读取数据，可以调用read()方法之一。举个例子:

```
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

首先分配一个缓冲区。从FileChannel读取的数据被读入缓冲区。

其次，调用FileChannel.read()方法。此方法将数据从FileChannel读入缓冲区。read()方法返回的int值显示往缓冲区写入了多少字节。如果返回-1，则到达文件末尾。

### 7.3、将数据写入FileChannel

将数据写入FileChannel是使用FileChannel.write()方法完成的，该方法需要Buffer作为参数。举个例子:

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

注意，如何在while循环中调用FileChannel.write()方法。write()方法不能保证向FileChannel写入多少字节。因此，我们重复write()调用，直到缓冲区没有需要写入的字节为止。

### 7.4、关闭一个FileChannel

当您完成使用FileChannel时，必须关闭它。这是如何做到的:

```java
channel.close();    
```

### 7.5、FileChannel位置

当读取或写入一个FileChannel时，需要在特定的位置进行操作。您可以通过调用position()方法获得FileChannel对象的当前位置。

您还可以通过调用position(long pos)方法来设置FileChannel的位置。

这里有两个例子:

```
long pos = channel.position();

channel.position(pos +123);
```

如果您在文件结束后设置位置，并尝试从通道读取，您将得到-1-文件结束标记。

如果在文件结束后设置位置，并写入通道，文件将被展开以适应位置和写入的数据。这可能会导致“文件漏洞”，即磁盘上的物理文件在写入的数据中有间隙。

### 7.6、FileChannel大小

FileChannel对象的size()方法返回通道连接到的文件的文件大小。下面是一个简单的例子:

```java
long fileSize = channel.size();    
```

### 7.7、FileChannel截断

您可以通过调用FileChannel.truncate()方法来截断文件。当您截断一个文件时，您将它以给定的长度截断。举个例子:

```java
channel.truncate(1024);
```

这个示例以1024字节的长度截断文件。

### 7.8、FileChannel Force

force()方法将所有未写的数据从通道刷新到磁盘。由于性能原因，操作系统可能会在内存中缓存数据，所以在调用force()方法之前，不能保证写入通道的数据实际上是写入磁盘的。

force()方法接受一个布尔值作为参数，告诉是否也应该刷新文件元数据(权限等)。

下面是一个同时刷新数据和元数据的例子:

```java
channel.force(true);
```

## 八、Java NIO SocketChannel

Java NIO SocketChannel是连接到TCP网络套接字的通道。它相当于Java NIO的Java连网套接字。有两种方法可以创建SocketChannel:

1. 您打开SocketChannel并连接到internet上某处的服务器。
2. 可以在传入连接到达ServerSocketChannel时创建SocketChannel。

### 8.1、打开一个SocketChannel

下面是如何打开SocketChannel:

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

### 8.2、关闭SocketChannel

通过调用SocketChannel.close()方法，可以在使用后关闭SocketChannel。这是如何做到的:

```java
socketChannel.close();   
```

### 8.3、从套接字通道读取

要从SocketChannel读取数据，可以调用read()方法之一。举个例子:

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = socketChannel.read(buf);
```

首先分配一个缓冲区。从SocketChannel读取的数据被读入缓冲区。

其次，调用SocketChannel.read()方法。该方法将数据从SocketChannel读入缓冲区。read()方法返回的int值告诉往缓冲区中写入了多少字节。如果返回-1，则到达流的末端(连接已关闭)。

### 8.4、写入SocketChannel

将数据写入SocketChannel是使用SocketChannel.write()方法完成的，该方法将缓冲区作为参数。举个例子:

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

注意，如何在while循环中调用SocketChannel.write()方法。write()方法不能保证向SocketChannel写入多少字节。因此，我们重复write()调用，直到缓冲区没有需要写入的字节为止。

### 8.5、Non-blocking Mode

您可以将SocketChannel设置为非阻塞模式。这样做时，可以在异步模式下调用connect()、read()和write()。

#### 8.5.1、connect()

如果SocketChannel处于非阻塞模式，并且调用connect()，则该方法可能在建立连接之前返回。要确定是否建立连接，可以调用finishConnect()方法，如下所示:

```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...    
}
```

#### 8.5.2、write()

在非阻塞模式下，write()方法可以返回，而不需要编写任何内容。因此，您需要在循环中调用write()方法。但是，由于这已经在前面的写示例中完成了，所以这里不需要做任何不同的操作。

#### 8.5.3、read()

在非阻塞模式下，read()方法可能根本不读取任何数据就返回。因此，需要注意返回的int，它表示读取了多少字节。

#### 8.5.4、Non-blocking Mode with Selectors

套接字通道的非阻塞模式与选择器的工作得更好。通过向选择器注册一个或多个SocketChannel's，您可以向选择器询问准备读取、写入的通道。如何在SocketChannel中使用选择器将在本教程后面的文本中详细解释。

## 九、Java NIO ServerSocketChannel

Java NIO ServerSocketChannel是一个可以侦听传入TCP连接的通道，就像标准Java网络中的ServerSocket一样。ServerSocketChannel类位于java.nio中。渠道包。

下面是一个例子：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

### 9.1、打开一个ServerSocketChannel

通过调用ServerSocketChannel.open()方法打开ServerSocketChannel。它是这样的:

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

### 9.2、关闭一个ServerSocketChannel

关闭ServerSocketChannel是通过调用ServerSocketChannel.close()方法来完成的。它是这样的:

```java
serverSocketChannel.close();
```

### 9.3、监听传入连接

监听传入连接是通过调用ServerSocketChannel.accept()方法来完成的。当accept()方法返回时，它返回一个带有传入连接的套接字通道。因此，accept()方法阻塞，直到传入的连接到达。

由于您通常对侦听单个连接不感兴趣，所以您可以在while循环中调用accept()。它是这样的:

```java
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

当然，您将在while循环中使用一些其他的stop-criteria，而不是true。

### 9.4、non-blocking模式

可以将ServerSocketChannel设置为非阻塞模式。在非阻塞模式下，accept()方法立即返回，如果没有到达传入连接，则可能返回null。因此，您必须检查返回的SocketChannel是否为空。举个例子:

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

## 10、Java NIO: Non-blocking Server

即使您了解Java NIO非阻塞特性(选择器、通道、缓冲区等)是如何工作的，设计非阻塞服务器仍然很困难。与阻塞IO相比，非阻塞IO包含几个挑战。本非阻塞服务器教程将讨论非阻塞服务器的主要挑战，并描述一些可能的解决方案。

找到关于设计非阻塞服务器的良好信息是困难的。因此，本教程提供的解决方案是基于我自己的工作和想法。如果你有其他选择或者更好的主意，我很乐意听到!你可以在文章下面写一条评论，或者给我发一封电子邮件(见我们的About页面)，或者在Twitter上找到我。

本教程中描述的思想是围绕Java NIO设计的。但是，我相信这些思想可以在其他语言中重用，只要它们具有某种类似于选择器的构造。据我所知，这些结构是由底层OS提供的，所以很有可能您也可以用其他语言访问它。

### 10.1、Non-blocking Server - GitHub Repository

我在本教程中创建了一个简单的概念验证概念,并将其放入GitHub存储库中,让您查看。这是GitHub存储库:

<https://github.com/jjenkov/java-nio-server>

### 10.2、Non-blocking IO Pipelines

非阻塞IO管道是处理非阻塞IO的组件链。这包括以非阻塞方式读取和写入IO。下面是一个简化的非阻塞IO管道的例子:

![1556894082822](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556894082822.png)

组件使用选择器检查通道何时有数据要读取。然后组件读取输入数据并根据输入生成一些输出。再次将输出写入通道。

非阻塞IO管道不需要读写数据。有些管道可能只读取数据，有些管道可能只写入数据。

上面的图表只显示了一个组件。非阻塞IO管道可能有多个组件处理传入数据。非阻塞IO管道的长度取决于管道需要做什么。

非阻塞IO管道也可以同时从多个通道读取数据。例如，从多个套接字通道读取数据。

上图中的控制流程也得到了简化。它是启动通过选择器从通道读取数据的组件。并不是通道将数据推入选择器，然后从选择器推入组件，即使这是上面的图表所建议的。

### 10.3、Non-blocking vs. Blocking IO Pipelines

非阻塞IO管道和阻塞IO管道之间最大的区别是如何从底层通道(套接字或文件)读取数据。

IO管道通常从某些流(从套接字或文件)读取数据，并将数据拆分为一致的消息。这类似于将数据流分解为令牌，以便使用令牌器进行解析。相反，您将数据流分解为更大的消息。我将调用将流分解为消息阅读器的消息的组件。下面是一个消息阅读器将消息流分解为消息的例子:

![1556978643876](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556978643876.png)

阻塞IO管道可以使用一个类似inputstream的接口，在这个接口中，每次可以从底层通道读取一个字节，而类似inputstream的接口会阻塞该接口，直到有数据可以读取为止。这将导致阻塞消息读取器实现。

使用阻塞IO接口消息流简化了实现读者很多。一个阻塞消息读者从未处理的情况下没有从流读取数据,或者只有一个部分从流读取的消息和消息解析需要恢复。

类似地，阻塞消息编写器(向流编写消息的组件)永远不必处理只编写了消息的一部分以及稍后必须恢复消息编写的情况。

### 10.4、Blocking IO Pipeline Drawbacks（阻塞IO管道缺点）

虽然阻塞消息读取器更容易实现，但它有一个不幸的缺点，即需要为需要分割为消息的每个流分配一个单独的线程。之所以需要这样做，是因为每个流块的IO接口直到有一些数据可以从中读取为止。这意味着单个线程不能尝试从一个流读取数据，如果没有数据，则从另一个流读取。只要线程试图从流中读取数据，线程就会阻塞，直到实际有一些数据需要读取。

如果IO管道是服务器的一部分，服务器必须处理许多并发连接，那么服务器将需要每个活动输入连接一个线程。如果服务器在任何时候只有几百个并发连接，这可能不是问题。但是，如果服务器有数百万个并发连接，这种设计就不能很好地扩展。每个线程的堆栈将占用320K(32位JVM)和1024K(64位JVM)之间的内存。因此，1.000.000个线程将占用1 TB内存!这是在服务器使用任何内存来处理传入消息之前(例如内存分配器)。

为了减少线程数量，许多服务器使用这样的设计:服务器保存一个线程池(例如100)，每次从入站连接读取一条消息。入站连接保存在队列中，线程按照入站连接放入队列的顺序处理来自每个入站连接的消息。本设计如下图所示:

![1556979021847](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556979021847.png)

然而，这种设计要求入站连接合理地频繁地发送数据。如果入站连接可能处于非活动状态的时间更长，那么大量非活动连接实际上可能阻塞线程池中的所有线程。这意味着服务器响应变慢，甚至没有响应。

一些服务器设计试图通过在线程池中的线程数量上增加一些弹性来缓解这个问题。例如，如果线程池耗尽线程，线程池可能启动更多的线程来处理负载。这种解决方案意味着需要更多的慢速连接才能使服务器停止响应。但是请记住，您可以运行多少线程仍然有一个上限。因此，对于1.000.000个慢速连接，这将不能很好地扩展。

### 10.5、Basic Non-blocking IO Pipeline Design（基本的非阻塞IO管道设计）

非阻塞IO管道可以使用一个线程从多个流读取消息。这要求流可以切换到非阻塞模式。在非阻塞模式下，当您试图从流中读取数据时，流可能返回0或更多字节。如果流没有要读取的数据，则返回0字节。当流实际上有一些数据要读取时，返回1+字节。

为了避免检查需要读取0字节的流，我们使用Java NIO选择器。可以使用选择器注册一个或多个SelectableChannel实例。当您在选择器上调用select()或selectNow()时，它只给您一个SelectableChannel实例，该实例实际上具有要读取的数据。本设计如下图所示:

![1556979178715](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556979178715.png)

### 10.6、Reading Partial Messages（读取部分信息）

当我们从SelectableChannel读取数据块时，我们不知道该数据块包含的消息是少还是多。数据块可能包含部分消息(少于消息)、完整消息或多于消息，例如1.5或2.5 messsages。各种部分信息的可能性说明如下:

![1556979274964](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556979274964.png)

处理部分讯息有两个挑战:

1. 检测数据块中是否有完整的消息。
2. 如何处理部分消息，直到消息的其余部分到达。

检测完整的消息需要消息阅读器查看数据块中的数据，以查看数据是否包含至少一条完整的消息。如果数据块包含一个或多个完整的消息，则可以通过管道发送这些消息进行处理。寻找完整消息的过程会重复很多次，因此这个过程必须尽可能快。

当数据块中有部分消息时，无论是单独的还是在一个或多个完整消息之后，都需要存储该部分消息，直到该消息的其余部分从通道到达为止。

检测完整消息和存储部分消息都是消息阅读器的职责。为了避免混合来自不同通道实例的消息数据，我们将每个通道使用一个消息阅读器。设计是这样的:

![1556979455905](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556979455905.png)

在检索到要从选择器读取数据的通道实例之后，与该通道关联的消息读取器将读取数据并尝试将其分解为消息。如果这导致读取任何完整的消息，则可以将这些消息通过读取管道传递到需要处理它们的任何组件。

消息阅读器当然是特定于协议的。消息阅读器需要知道它要读取的消息的消息格式。如果我们的服务器实现要跨协议重用，它需要能够插入消息阅读器实现——可能通过以某种方式接受消息阅读器工厂作为配置参数。

### 10.7、Storing Partial Messages（存储部分信息）

既然我们已经确定了在接收到完整消息之前存储部分消息是消息阅读器的职责，那么我们需要弄清楚应该如何实现该部分消息存储。

我们需要考虑两个设计因素:

1. 我们希望尽可能少地复制消息数据。复制越多，性能越低。
2. 我们希望完整的消息以连续的字节序列存储，以便更容易地解析消息。

#### 10.7.1、A Buffer Per Message Reader（每个消息阅读器有一个缓冲区）

显然，部分消息需要存储在某种缓冲区中。简单的实现方法是在每个消息阅读器内部有一个缓冲区。然而，这个缓冲区应该有多大?它需要足够大，甚至能够存储允许的最大消息。因此，如果允许的最大消息是1MB，那么每个消息阅读器中的内部缓冲区至少需要1MB。

当我们到达数百万个连接时，每个连接使用1MB并不能真正工作。1.000.000 x 1MB仍然是1TB的内存!如果消息的最大大小是16MB呢?还是128 mb ?

#### 10.7.2、Resizable Buffers（可调整大小的缓冲区）

另一个选项是实现一个可调整大小的缓冲区，以便在每个消息阅读器中使用。可调整大小的缓冲区将从小处开始，如果消息对于缓冲区来说太大，则会展开缓冲区。这样，每个连接都不需要1MB的缓冲区。每个连接只占用存储下一条消息所需的内存。

有几种方法可以实现可调整大小的缓冲区。它们都有优点和缺点，所以我将在接下来的部分中讨论它们。

#### 10.7.3、Resize by Copy（调整大小的副本）

实现可调整大小的缓冲区的第一种方法是从4KB这样的小缓冲区开始。如果消息不能装入4KB缓冲区，则可以分配一个更大的缓冲区，例如8KB，并将4KB缓冲区中的数据复制到更大的缓冲区中

按大小复制缓冲区实现的优点是，消息的所有数据都保存在一个连续的字节数组中。这使得解析消息更加容易。

按大小复制缓冲区实现的缺点是，它会导致为更大的消息复制大量数据。

为了减少数据复制，您可以分析流经系统的消息的大小，以找到一些可以减少复制量的缓冲区大小。例如，您可能会看到大多数消息都小于4KB，因为它们只包含非常小的请求/响应。这意味着第一个缓冲区大小应该是4KB。

然后您可能会看到，如果消息大于4KB，通常是因为它包含一个文件。然后您可能会注意到，流经系统的大多数文件都小于128KB。然后，使第二个缓冲区大小为128KB是有意义的。

最后，您可能会看到，一旦消息超过128KB，消息的大小就没有真正的模式了，所以最后的缓冲区大小应该是最大的消息大小。

使用这3种基于流经系统的消息大小的缓冲区大小，您将在一定程度上减少数据复制。低于4KB的消息将永远不会被复制。对于1.000.000个并发连接，导致1.000.000 x 4KB = 4GB，这在今天(2015年)的大多数服务器中是可能的。4KB到128KB之间的消息将被复制一次，并且只需要将4KB数据复制到128KB缓冲区中。在128KB到最大消息大小之间的消息将被复制两次。第一次将复制4KB，第二次将复制128KB，因此对于最大的消息，总共复制132KB。

一旦消息被完全处理，分配的内存应该再次被释放。这样，从相同连接接收到的下一条消息又以最小的缓冲区大小开始。这对于确保在连接之间更有效地共享内存是必要的。大多数情况下，并非所有连接都需要同时使用大缓冲区。

我有一个完整的教程，介绍如何实现这样一个内存缓冲区，它支持可调整大小的数组:可调整大小的数组。本教程还包含一个到GitHub存储库的链接，其中的代码显示了一个工作实现。

#### 10.7.4、Resize by Append（追加调整）

另一种调整缓冲区大小的方法是使缓冲区由多个数组组成。当需要调整缓冲区大小时，只需分配另一个字节数组并将数据写入其中。

有两种方法可以增加这样的缓冲区。一种方法是分配单独的字节数组并保存这些字节数组的列表。另一种方法是分配较大的共享字节数组的片，然后保存分配给缓冲区的片的列表。就我个人而言，我觉得切片方法稍微好一些，但是差别很小。

通过附加单独的数组或片来增长缓冲区的优点是，在编写期间不需要复制任何数据。所有数据都可以直接从套接字(通道)复制到数组或片中。

以这种方式增长缓冲区的缺点是数据不是存储在一个连续的数组中。这使得消息解析更加困难，因为解析器需要同时查找每个单独数组的末尾和所有数组的末尾。由于您需要在书面数据中查找消息的结尾，因此使用这个模型并不太容易。

#### 10.7.5、TLV Encoded Messages

一些协议消息格式使用TLV格式(类型、长度、值)编码。这意味着，当消息到达时，消息的总长度存储在消息的开头。这样就可以立即知道要为整个消息分配多少内存。

TLV编码使内存管理更加容易。您立即知道要为消息分配多少内存。只有部分使用的缓冲区的末尾不会浪费内存。

TLV编码的一个缺点是在消息的所有数据到达之前为消息分配所有内存。一些发送大消息的慢速连接因此可以分配所有可用的内存，使服务器没有响应。

解决这个问题的方法是使用包含多个TLV字段的消息格式。因此，内存分配给每个字段，而不是整个消息，内存只在字段到达时分配。不过，大字段对内存管理的影响与大消息相同。

另一种解决方法是在10-15秒内暂停未收到的消息。这可以使您的服务器从许多大消息的同时到达中恢复过来，但是仍然会使服务器在一段时间内没有响应。此外，蓄意的DoS(拒绝服务)攻击仍然会导致服务器内存的完全分配。

TLV编码存在不同的变异。具体使用了多少字节，因此指定字段的类型和长度取决于每个TLV编码。还有一种TLV编码，它将字段的长度放在前面，然后是类型，最后是值(一种LTV编码)。虽然字段序列不同，但仍然是TLV变异。

TLV编码使内存管理更容易，这是HTTP 1.1是如此糟糕的协议的原因之一。这是他们在HTTP 2.0中试图解决的问题之一，其中数据以LTV编码帧传输。这也是为什么我们为VStack设计了自己的网络协议。使用TLV编码的co项目。

### 10.8、Writing Partial Messages

在非阻塞IO管道中，编写数据也是一个挑战。当您在非阻塞模式下调用通道上的write(ByteBuffer)时，无法保证ByteBuffer中有多少字节正在被写入。write(ByteBuffer)方法返回写入的字节数，因此可以跟踪写入的字节数。这就是挑战所在:跟踪部分已写的消息，以便最终发送消息的所有字节。

为了管理对通道的部分消息的写入，我们将创建一个消息写入器。就像使用消息阅读器一样，我们将需要为每个写入消息的通道设置一个消息写入器。在每个消息写入器中，我们精确地跟踪它当前正在写入的消息已经写入了多少字节。

如果消息写入器收到的消息比直接写到通道的消息多，则需要在消息写入器内部排队。然后，消息编写器将消息以尽可能快的速度写入通道。

下面的图表显示了到目前为止部分消息的编写是如何设计的:

![1556980170225](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556980170225.png)

要使消息编写器能够发送早期仅部分发送的消息，需要不时地调用消息编写器，以便它能够发送更多的数据。

如果有很多连接，就会有很多Message Writer实例。例如，检查一百万个Message Writer实例，看看它们是否能写数据是很慢的。首先，许多Message Writer实例没有任何消息要发送。我们不想检查那些Message Writer实例。其次，并非所有通道实例都可以编写数据。我们不想浪费时间试图将数据写入无法接受任何数据的通道。

要检查通道是否准备好写入，可以使用选择器注册通道。但是，我们不希望用选择器注册所有通道实例。想象一下，如果您有1.000.000个连接，这些连接大部分是空闲的，并且所有1.000.000个连接都已注册到选择器中。然后，当您调用select()时，这些通道实例中的大多数都是写就绪的(它们大部分是空闲的，还记得吗?)然后，您必须检查所有这些连接的消息编写器，以查看它们是否有要编写的数据。

为了避免检查所有消息编写器实例中的消息，以及所有通道实例中没有任何消息要发送给它们，我们使用以下两步方法:

1. 当将消息写入消息写入器时，消息写入器将向选择器注册其关联的通道(如果尚未注册)。
2. 当您的服务器有时间时，它将检查选择器，以查看哪些已注册的通道实例已准备好编写。对于每个准备写的通道，都要求其关联的消息写入器将数据写入通道。如果消息写入器将其所有消息写入其通道，则通道将再次从选择器注销。

这个小小的两步方法确保只有包含要写入的消息的通道实例才真正注册到选择器中。

### 10.9、Putting it All Together

如您所见，非阻塞服务器需要不时检查传入的数据，以查看是否收到了任何新的完整消息。服务器可能需要多次检查，直到收到一条或多条完整的消息。检查一次是不够的。

类似地，非阻塞服务器需要不时检查是否有需要写入的数据。如果是，服务器需要检查相应的连接是否准备好将数据写入它们。仅在消息第一次排队时进行检查是不够的，因为消息可能是部分编写的。

总而言之，一个非阻塞服务器最终需要定期执行三个“管道”:

- 从打开的连接中检查新传入数据的读管道。

- 处理接收到的任何完整消息的处理管道。

- 写入管道，检查是否可以将任何传出消息写入任何打开的连接。

这三个管道在一个循环中重复执行。您可能能够优化执行。例如，如果没有消息排队，可以跳过写管道。或者，如果没有收到新的、完整的消息，您可以跳过流程管道。

下面是一个图表，说明了整个服务器循环:

![1556980431288](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556980431288.png)

如果您仍然觉得这有点复杂，请记住查看GitHub存储库:

<https://github.com/jjenkov/java-nio-server>

也许看到实际运行的代码可以帮助您理解如何实现这一点。

### 10.10、Server Thread Model

GitHub存储库中的非阻塞服务器实现使用一个包含两个线程的线程模型。第一个线程接受来自ServerSocketChannel的传入连接。第二个线程处理接受的连接，这意味着读取消息、处理消息并将响应写回连接。这个2线程模型如下图所示:

![1556980505336](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556980505336.png)

上一节中解释的服务器处理循环由处理线程执行。

## 11、Java NIO DatagramChannel

Java NIO DatagramChannel是一个可以发送和接收UDP数据包的通道。由于UDP是一种无连接的网络协议，您不能像从其他通道读取和写入数据流通道一样，在默认情况下直接读取和写入数据流通道。而是发送和接收数据包。

### 11.1、Opening a DatagramChannel

下面是如何打开DatagramChannel:

```java
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

本例打开一个DatagramChannel，它可以在UDP端口9999上接收数据包。

### 11.2、Receiving Data

通过调用DatagramChannel的receive()方法接收数据，如下所示:

```java
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

channel.receive(buf);
```

receive()方法将把接收到的数据包的内容复制到给定的缓冲区中。如果接收到的数据包包含的数据超过缓冲区所能包含的数据，那么剩余的数据将被无声地丢弃。

### 11.3、Sending Data

您可以通过调用DatagramChannel的send()方法来发送数据，如下所示:

```java
String newData = "New String to write to file..."
                    + System.currentTimeMillis();
    
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

本例将字符串发送到UDP端口80上的“jenkov.com”服务器。但是没有监听端口，所以什么也不会发生。您将不会被通知是否收到发送包，因为UDP不做任何保证，关于数据的交付。

### 11.4、Connecting to a Specific Address

可以将DatagramChannel“连接”到网络上的特定地址。由于UDP是无连接的，这种连接到地址的方式不会创建真正的连接，就像使用TCP通道一样。相反，它锁定您的DatagramChannel，因此您只能从一个特定的地址发送和接收数据包。

下面是一个例子：

```java
channel.connect(new InetSocketAddress("jenkov.com", 80));  
```

连接时，还可以使用read()和write()方法，就像使用传统通道一样。对于发送的数据，您没有任何保证。这里有几个例子:

```java
int bytesRead = channel.read(buf); 
int bytesWritten = channel.write(buf);
```

## 12、Java NIO Pipe

Java NIO管道是两个线程之间的单向数据连接。管道有源信道和接收信道。将数据写入接收器通道。然后可以从源通道读取这些数据。

下面是管道原理的说明:

![1556981223177](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1556981223177.png)

### 12.1、Creating a Pipe

您可以通过调用Pipe.open()方法来打开管道。它是这样的:

```java
Pipe pipe = Pipe.open();
```

### 12.2、Writing to a Pipe

要写入管道，您需要访问sink通道。这是如何做到的:

```
Pipe.SinkChannel sinkChannel = pipe.sink();
```

您可以通过调用它的write()方法来写入一个SinkChannel，如下所示:

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

### 12.3、Reading from a Pipe

要从管道中读取数据，需要访问源通道。这是如何做到的:

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```

要从源通道读取，可以像这样调用它的read()方法:

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

read()方法返回的int值表示读入缓冲区的字节数。

## 13、Java NIO vs. IO

在学习Java NIO和IO API时，一个问题很快就会浮现在脑海中:

我应该何时使用IO,何时使用NIO ?

在本文中,我将尝试阐明Java NIO和IO之间的差异,它们的用例,以及它们如何影响代码的设计。

### 13.1、Main Differences Betwen Java NIO and IO（NIO和IO的主要区别）

下表总结了Java NIO和IO之间的主要区别。我将在下面的几节中更详细地介绍每个差异。

| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
|        | 选择器     |

### 13.2、Stream Oriented vs. Buffer Oriented

Java NIO和IO之间的第一个主要区别是IO是面向流的，而NIO是面向缓冲区的。这是什么意思呢?

面向流的Java IO意味着每次从流中读取一个或多个字节。如何处理读取的字节由您决定。它们不会被缓存到任何地方。此外，您不能在流中的数据中来回移动。如果需要来回移动从流读取的数据，则需要首先将其缓存到缓冲区中。

Java NIO的面向缓冲区方法略有不同。数据被读入一个缓冲区，然后从中进行处理。您可以在缓冲区中来回移动。这使您在处理过程中更具灵活性。但是，您还需要检查缓冲区是否包含了完整处理它所需的所有数据。而且，您需要确保在将更多数据读入缓冲区时，不会覆盖尚未处理的缓冲区中的数据。

### 13.3、Blocking vs. Non-blocking IO

Java IO的各种流正在阻塞。这意味着，当线程调用read()或write()时，该线程将被阻塞，直到有一些数据需要读取，或者数据被完全写入。线程在此期间不能做任何其他事情。

Java NIO的非阻塞模式允许线程请求从通道读取数据，并且只获取当前可用的数据，如果当前没有可用的数据，则什么也得不到。线程可以继续执行其他操作，而不是在数据可用之前一直被阻塞。

不阻塞写作也是如此。一个线程可以请求一些数据写入一个通道,但不要等待它被完全写入。然后,线程可以继续进行,并在平均时间内做其他的事情。

当没有在IO调用时,线程会花在空闲时间,通常在其他通道上执行IO。也就是说,一个线程现在可以管理多个输入和输出通道。

### 13.4、Selectors

Java NIO的选择器允许一个线程监视多个输入通道。您可以使用选择器注册多个通道，然后使用一个线程“选择”具有可用于处理的输入的通道，或者选择准备写入的通道。这种选择器机制使单个线程可以轻松地管理多个通道。

1. API调用NIO或IO类。
2. 数据处理。
3. 用于处理数据的线程数。

#### 13.4.1、The API Calls

当然，使用NIO时的API调用与使用IO时看起来是不同的。这并不奇怪。与其从InputStream中一个字节一个字节地读取数据，还不如先将数据读入缓冲区，然后再从那里进行处理。

#### 13.4.2、The Processing of Data

在使用纯NIO设计时,数据的处理也会受到影响,而IO设计。

在IO设计中，您从InputStream或读取器中逐字节读取数据。假设您正在处理基于行的文本数据流。例如:

```java
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```

这些文本行可以这样处理:

```java
InputStream input = ... ; // get the InputStream from the client socket

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

注意处理状态是如何由程序执行的距离决定的。换句话说，一旦第一个reader.readLine()方法返回，您就可以确定已经读取了一整行文本。readLine()会阻塞，直到读取完整行为止，这就是原因。您还知道这一行包含名称。类似地，当第二个readLine()调用返回时，您知道这一行包含年龄等。

正如您所看到的，程序只有在需要读取新数据时才会继续运行，并且对于每个步骤，您都知道该数据是什么。一旦执行的线程通过读取代码中的某段数据，线程就不会在数据中后退(大多数情况下不会)。这个原理也在这个图中得到了说明:

![1557019289076](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1557019289076.png)

NIO实现看起来会有所不同。下面是一个简单的例子:

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);
```

注意第二行，它将字节从通道读入ByteBuffer。当该方法调用返回时，您不知道是否需要的所有数据都在缓冲区中。您只知道缓冲区包含一些字节。这使得处理有些困难。

想象一下，在第一次读取(缓冲区)调用之后，所有读取到缓冲区的内容都是半行。例如，“Name: An”。你能处理这些数据吗?不是真的。您至少需要等到一行完整的数据进入缓冲区之后，才能处理任何数据。

那么，如何知道缓冲区中是否包含足够的数据，以便处理它呢?好吧,你不喜欢。找出答案的唯一方法是查看缓冲区中的数据。结果是，在知道是否所有数据都在缓冲区中之前，您可能必须检查缓冲区中的数据几次。这是低效的，并且在程序设计方面可能变得混乱。例如:

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```

bufferFull()方法必须跟踪有多少数据被读入缓冲区，并根据缓冲区是否满返回true或false。换句话说，如果缓冲区已准备好进行处理，则认为缓冲区已满。

bufferFull()方法扫描缓冲区，但必须使缓冲区保持与调用bufferFull()方法之前相同的状态。否则，下一个读入缓冲区的数据可能无法在正确的位置读入。这并非不可能，但这又是一个需要注意的问题。

如果缓冲区已满，则可以处理它。如果它不是满的，您可能能够部分处理任何数据，如果这对您的特定情况有意义的话。在很多情况下并不是这样。

is-data-in-buffer-ready循环如下图所示:

![1557019432931](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1557019432931.png)

### 13.5、Summary（总结）

NIO允许您仅使用一个(或几个)线程管理多个通道(网络连接或文件)，但代价是解析数据可能比从阻塞流读取数据要复杂一些。

如果需要同时管理数千个打开的连接(每个连接只发送少量数据，例如聊天服务器)，那么在NIO中实现服务器可能是一个优势。类似地，如果您需要保持与其他计算机的大量开放连接，例如在P2P网络中，使用一个线程管理所有出站连接可能是一个优势。这一个线程，多个连接的设计如下图所示:

![1557019799956](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1557019799956.png)

如果您有非常高带宽的少量连接，一次发送大量数据，那么经典的IO服务器实现可能是最佳选择。这个图表说明了一个经典的IO服务器设计:

![1557019819728](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1557019819728.png)

## 14、Java NIO Path

Java Path接口是Java NIO 2更新的一部分，Java NIO在Java 6和Java 7中接收到该更新。Java 7将Java Path接口添加到Java NIO中。Path接口位于java.nio中。因此，Java Path接口的完全限定名是Java .nio.file.Path。

Java Path实例表示文件系统中的路径。路径可以指向文件或目录。路径可以是绝对的，也可以是相对的。绝对路径包含从文件系统的根目录到它所指向的文件或目录的完整路径。相对路径包含相对于其他路径的文件或目录的路径。相对路径可能听起来有点混乱。别担心。稍后，我将在Java NIO Path教程中更详细地解释相关路径。

在某些操作系统中，不要混淆文件系统路径和路径环境变量。java.nio.file。Path接口与Path环境变量无关。

在许多方面，java.nio.文件。Path接口类似于java.io。文件类，但有一些细微的区别。不过，在许多情况下，您可以使用Path接口代替File类。

### 14.1、Creating a Path Instance

以便使用java.nio.文件。您必须创建一个Path实例。使用名为Paths.get()的Paths类(java.nio.file.Paths)中的静态方法创建Path实例。下面是一个Java path .get()例子:

```java
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {

    public static void main(String[] args) {

        Path path = Paths.get("c:\\data\\myfile.txt");

    }
}
```

注意示例顶部的两个import语句。要使用Path接口和Paths类，我们必须首先导入它们。

其次，注意path .get(“c:\\data\\myfile.txt”)方法调用。创建Path实例的是对Paths.get()方法的调用。换句话说，Path .get()方法是Path实例的工厂方法。

#### 14.1.1、Creating an Absolute Path（创建绝对路径）

创建绝对路径是通过调用path .get() factory方法来完成的，该方法的参数是绝对文件。下面是创建一个表示绝对路径的路径实例的例子:

```java
Path path = Paths.get("c:\\data\\myfile.txt");
```

绝对路径是c:\data\myfile.txt。在Java字符串中，双\字符是必需的，因为\是转义字符，这意味着下面的字符告诉我们在字符串的这个位置应该使用什么字符。通过编写\\告诉Java编译器在字符串中写入一个\字符。

上面的路径是Windows文件系统路径。在Unix系统(Linux、MacOS、FreeBSD等)上，上面的绝对路径可以是这样的:

```java
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

绝对路径现在是/home/jakobjenkov/myfile.txt。

如果您在Windows机器上使用这种路径(以/开头的路径)，该路径将被解释为相对于当前驱动器的路径。例如，路径

```java
/home/jakobjenkov/myfile.txt
```

可以解释为位于C驱动器上。那么这个路径就对应于这个完整的路径:

```java
C:/home/jakobjenkov/myfile.txt
```

#### 14.1.2、Creating a Relative Path（创建相对路径）

相对路径是从一个路径(基本路径)指向目录或文件的路径。相对路径的完整路径(绝对路径)是通过将基本路径与相对路径相结合而得到的。

Java NIO Path类还可以用于处理相对路径。使用这些路径创建一个相对路径。得到(basePath relativePath)方法。以下是Java中的两个相对路径示例:

```java
Path projects = Paths.get("d:\\data", "projects");

Path file     = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一个示例创建一个指向路径(目录)d:\data\projects的Java Path实例。第二个示例创建一个Path实例，该实例指向路径(文件)d:\data\projects\a-project\myfile.txt。

在处理相对路径时，可以在路径字符串中使用两个特殊代码。这些代码是:

- .
- ..

.代码表示“当前目录”。例如，如果您创建一个像这样的相对路径:

```java
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

然后，Java path实例所对应的绝对路径将是执行上述代码的应用程序所在的目录。

如果.在路径字符串的中间使用，它只表示与该点所指向的路径相同的目录。下面是一个路径例子:

```java
Path currentDir = Paths.get("d:\\data\\projects\.\a-project");
```

这条路径将对应这条路径:

```java
d:\data\projects\a-project
```

. .代码表示“父目录”或“向上一个目录”。下面是一个Path Java例子:

```
Path parentDir = Paths.get("..");
```

本例创建的Path实例将对应于运行此代码的应用程序启动时所在目录的父目录。

如果你用。位于路径字符串中间的代码将对应于在路径字符串的该点上更改一个目录。例如:

```java
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```

这个例子创建的Java Path实例将对应于这个绝对路径:

```
d:\data\projects\another-project
```

. .代码在a-project目录将目录向上更改为父目录projects，然后将路径引用从那里向下更改为另一个-project目录。

.和. .代码还可以与双字符串path .get()方法结合使用。下面是两个Java path .get()例子，展示了一些简单的例子:

```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project",
                       "..\\another-project");
```

可以使用Java NIO Path类处理相对路径的方法有很多。您将在本教程的后面了解更多相关信息。

### 14.2、Path.normalize()

Path接口的normalize()方法可以对路径进行标准化。归一化意味着它删除所有。和. .在路径字符串的中间编码，并解析路径字符串所指的路径。下面是一个Java Path.normalize()示例:

```java
String originalPath =
        "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

这个路径示例首先创建一个带有..的路径字符串。代码在中间。然后，该示例从这个Path字符串创建一个Path实例，并输出该Path实例(实际上它输出Path. tostring())。

然后，该示例在创建的Path实例上调用normalize()，该实例返回一个新的Path实例。然后还打印出这个新的规范化路径实例。

下面是上面例子打印的输出:

```
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```

如您所见，规范化路径不包含a-project\\..部分，因为这是多余的。删除的部分不向最终的绝对路径添加任何内容。

## 15、Java NIO Files

Java NIO Files类(Java . NIO .file.Files)提供了几种操作文件系统中的文件的方法。本Java NIO文件教程将介绍这些方法中最常用的方法。Files类包含许多方法，所以如果需要这里没有描述的方法，也请检查JavaDoc。Files类可能仍然有一个方法。

java.nio.file。Files类与java.nio.file一起工作。因此，在使用Files类之前，需要了解Path类。

### 15.1、Files.exists()

Files.exists()方法检查文件系统中是否存在给定的路径。

可以创建文件系统中不存在的Path实例。例如，如果您计划创建一个新目录，您将首先创建相应的Path实例，然后创建目录。

由于Path实例可能指向文件系统中存在的路径，也可能不指向，所以可以使用Files.exists()方法来确定它们是否指向(如果需要检查的话)。

下面是一个Java Files.exists()示例:

```java
Path path = Paths.get("data/logging.properties");

boolean pathExists =
        Files.exists(path,
            new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

这个例子首先创建一个Path实例，指向我们想要检查是否存在的路径。其次，该示例调用Files.exists()方法，其中Path实例作为第一个参数。

注意Files.exists()方法的第二个参数。这个参数是一个选项数组，它影响Files.exists()确定路径是否存在。在上面的这个例子中，数组包含LinkOption.NOFOLLOW_LINKS，这意味着Files.exists()方法不应该跟随文件系统中的符号链接来确定路径是否存在。

### 15.2、Files.createDirectory()

方法的作用是:从Path实例创建一个新目录。下面是一个Java Files.createDirectory()示例:

```java
Path path = Paths.get("data/subdir");

try {
    Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
    // the directory already exists.
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```

第一行创建表示要创建目录的Path实例。在try-catch块中，以path作为参数调用Files.createDirectory()方法。如果创建目录成功，则返回一个Path实例，该实例指向新创建的路径。

如果目录已经存在，将抛出java.nio.file.FileAlreadyExistsException。如果其他地方出错，可能会抛出IOException。例如，如果所需的新目录的父目录不存在，则可能引发IOException。父目录是要在其中创建新目录的目录。因此，它表示新目录的父目录。

### 15.3、Files.copy()

copy()方法将文件从一个路径复制到另一个路径。下面是一个Java NIO Files.copy()示例:

```java
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```

首先，该示例创建源和目标路径实例。然后该示例调用Files.copy()，将两个Path实例作为参数传递。这将导致源路径引用的文件被复制到目标路径引用的文件中。

如果目标文件已经存在，则使用java.nio.file.FileAlreadyExistsException抛出。如果出现其他错误，将抛出IOException。例如，如果要复制文件到的目录不存在，将抛出IOException。

#### 15.3.1、Overwriting Existing Files

可以强制Files.copy()覆盖现有文件。下面是一个示例，演示如何使用Files.copy()覆盖现有文件:

```java
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```

注意Files.copy()方法的第三个参数。如果目标文件已经存在，此参数指示copy()方法重写现有文件。

#### 15.3.2、Files.move()

Java NIO Files类还包含一个函数，用于将文件从一个路径移动到另一个路径。移动一个文件与重命名它是一样的，只是移动一个文件既可以将它移动到另一个目录，也可以在同一操作中更改它的名称。java.io.File类也可以使用它的renameTo()方法来实现这一点，但是现在您在java.nio.file.Files中有了文件移动功能。	

下面是一个Java Files.move()例子:

```java
Path sourcePath      = Paths.get("data/logging-copy.properties");
Path destinationPath = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.move(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    //moving file failed.
    e.printStackTrace();
}
```

首先创建源路径和目标路径。源路径指向要移动的文件，目标路径指向应该移动文件的位置。然后调用Files.move()方法。这将导致文件被移动。

注意传递给Files.move()的第三个参数。此参数告诉Files.move()方法覆盖目标路径上的任何现有文件。这个参数实际上是可选的。

move()方法可能会在移动文件失败时抛出IOException。例如，如果目标路径上已经存在一个文件，并且您忽略了StandardCopyOption.REPLACE_EXISTING选项，或者如果要移动的文件不存在，等等。

#### 15.3.3、Files.delete()

delete()方法可以删除文件或目录。下面是一个Java Files.delete()示例:

```
Path path = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.delete(path);
} catch (IOException e) {
    //deleting file failed
    e.printStackTrace();
}
```

首先创建指向要删除的文件的路径。其次，调用Files.delete()方法。如果Files.delete()由于某种原因(例如文件或目录不存在)未能删除文件，则抛出IOException。

#### 15.3.4、Files.walkFileTree()

walkfiletree()方法包含递归遍历目录树的功能。walkFileTree()方法接受Path实例和FileVisitor作为参数。Path实例指向要遍历的目录。在traversion期间调用FileVisitor。

在我解释遍历的工作原理之前，首先介绍一下FileVisitor接口:

```java
public interface FileVisitor {

    public FileVisitResult preVisitDirectory(
        Path dir, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFile(
        Path file, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFileFailed(
        Path file, IOException exc) throws IOException;

    public FileVisitResult postVisitDirectory(
        Path dir, IOException exc) throws IOException {

}
```

您必须自己实现FileVisitor接口，并将实现的实例传递给walkFileTree()方法。在遍历目录期间，将在不同的时间调用FileVisitor实现的每个方法。如果不需要连接到所有这些方法，可以扩展SimpleFileVisitor类，该类包含FileVisitor接口中所有方法的默认实现。

下面是一个walkFileTree()示例:

```java
Files.walkFileTree(path, new FileVisitor<Path>() {
  @Override
  public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
    System.out.println("pre visit dir:" + dir);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
    System.out.println("visit file: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
    System.out.println("visit file failed: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
    System.out.println("post visit directory: " + dir);
    return FileVisitResult.CONTINUE;
  }
});
```

在遍历过程中，FileVisitor实现中的每个方法都在不同的时间被调用:

在访问任何目录之前都会调用preVisitDirectory()方法。postVisitDirectory()方法仅在访问目录之后调用。

在文件遍历期间访问的每个文件都会调用visitFile() 方法。它不用于目录—只用于文件。如果访问文件失败，则调用visitFileFailed()方法。例如，如果您没有正确的权限，或者出现了其他问题。

这四个方法都返回一个FileVisitResult 枚举实例。FileVisitResult枚举包含以下四个选项:

- `CONTINUE`
- `TERMINATE`
- `SKIP_SIBLINGS`
- `SKIP_SUBTREE`

通过返回这些值中的一个，被调用的方法可以决定应该如何继续文件遍历。

CONTINUE表示文件的遍历应该像往常一样继续。

TERMINATE意味着文件遍历现在应该终止。

SKIP_SIBLINGS表示文件遍历应该继续，但是不访问该文件或目录的任何兄弟姐妹。

SKIP_SUBTREE表示文件遍历应该继续，但不访问该目录中的条目。这个值只有在从preVisitDirectory()返回时才具有函数。如果从任何其他方法返回，它将被解释为一个CONTINUE。

##### 15.3.4.1、Searching For Files（搜索文件）

这是一个walkFileTree()，它扩展了SimpleFileVisitor来查找名为README.txt的文件:

```java
Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      String fileString = file.toAbsolutePath().toString();
      //System.out.println("pathString = " + fileString);

      if(fileString.endsWith(fileToFind)){
        System.out.println("file found at path: " + file.toAbsolutePath());
        return FileVisitResult.TERMINATE;
      }
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
    e.printStackTrace();
}
```

##### 15.3.4.2、Deleting Directories Recursively（递归删除目录）

walkfiletree()还可以用来删除包含所有文件和子目录的目录。 Files.delete()方法只在目录为空时才删除该目录。通过遍历所有目录并删除每个目录中的所有文件(在visitFile()中)，然后删除目录本身(在postVisitDirectory()中)，您可以删除包含所有子目录和文件的目录。下面是一个递归目录删除的例子:

```java
Path rootPath = Paths.get("data/to-delete");

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      System.out.println("delete file: " + file.toString());
      Files.delete(file);
      return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
      Files.delete(dir);
      System.out.println("delete dir: " + dir.toString());
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
  e.printStackTrace();
}
```

### 15.4、Additional Methods in the Files Class

java.nio.file.Files类包含许多其他有用的函数，如用于创建符号链接、确定文件大小、设置文件权限等的函数。查看JavaDoc以获得java.nio.file.Files类获取有关这些方法的更多信息。

## 16、Java NIO AsynchronousFileChannel（异步文件通道）

在Java 7中，AsynchronousFileChannel被添加到Java NIO中。AsynchronousFileChannel使异步地读取数据和将数据写入文件成为可能。本教程将解释如何使用AsynchronousFileChannel。

### 16.1、Creating an AsynchronousFileChannel

通过其静态方法open()创建AsynchronousFileChannel。下面是一个创建AsynchronousFileChannel的例子:

```java
Path path = Paths.get("data/test.xml");

AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```

open()方法的第一个参数是指向要与AsynchronousFileChannel关联的文件的路径实例。

第二个参数是一个或多个open选项，它告诉AsynchronousFileChannel要对基础文件执行哪些操作。在本例中，我们使用了StandardOpenOption.READ。这意味着文件将被打开以供读取。

### 16.2、Reading Data

可以通过两种方式从AsynchronousFileChannel读取数据。每种读取数据的方法都调用AsynchronousFileChannel的read()方法之一。下面几节将介绍这两种读取数据的方法。

### 16.3、Reading Data Via a Future

从AsynchronousFileChannel读取数据的第一种方法是调用read()方法，该方法返回一个Future。下面是调用read()方法的样子:

```java
Future<Integer> operation = fileChannel.read(buffer, 0);
```

read()方法的这个版本将ByteBuffer作为第一个参数。从AsynchronousFileChannel读取的数据被读入这个ByteBuffer。第二个参数是要开始读取的文件中的字节位置。

即使read操作尚未完成，read()方法也会立即返回。您可以通过调用read()方法返回的将来实例的isDone()方法来检查何时完成读取操作。

下面是一个更长的例子，展示了如何使用read()方法的这个版本:

```java
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

Future<Integer> operation = fileChannel.read(buffer, position);

while(!operation.isDone());

buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();
```

这个例子创建了一个AsynchronousFileChannel，然后创建了一个ByteBuffer，它作为参数传递给read()方法，位置为0。在调用read()之后，示例循环，直到返回的Future的isDone()方法返回true。当然，这不是对CPU的非常有效的使用——但是您需要等待read操作完成。

读取操作完成后，将数据读入ByteBuffer，然后读入字符串并打印到System.out。

### 16.4、Reading Data Via a CompletionHandler

从AsynchronousFileChannel读取数据的第二种方法是调用read()方法版本，该版本以CompletionHandler作为参数。下面是如何调用read()方法:

```java
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("result = " + result);

        attachment.flip();
        byte[] data = new byte[attachment.limit()];
        attachment.get(data);
        System.out.println(new String(data));
        attachment.clear();
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {

    }
});
```

一旦读取操作完成，CompletionHandler的completed()方法将被调用。作为completed()方法的参数，传递一个整数，该整数表示读取了多少字节，以及传递给read()方法的“附件”。“attachment”是read()方法的第三个参数。在本例中，也将数据读入ByteBuffer。您可以自由选择要附加什么对象

如果read操作失败，则会调用CompletionHandler的failed()方法。

### 16.5、Writing Data

与读取一样，可以用两种方式将数据写入AsynchronousFileChannel 。每种编写数据的方法都调用AsynchronousFileChannel 的write()方法之一。这两种编写数据的方法将在下面几节中讨论。

### 16.6、Writing Data Via a Future

AsynchronousFileChannel 还允许您异步地编写数据。下面是一个完整的Java AsynchronousFileChannel 写例子:

```java
Path path = Paths.get("data/test-write.txt");
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();

while(!operation.isDone());

System.out.println("Write done");
```

首先，以写模式打开AsynchronousFileChannel 。然后创建一个ByteBuffer并将一些数据写入其中。然后将ByteBuffer中的数据写入文件。最后，示例检查返回的Future，以查看写入操作何时完成。

注意，在此代码运行之前，文件必须已经存在。如果文件不存在，write()方法将抛出`java.nio.file.NoSuchFileException` 。

您可以使用以下代码确保路径指向的文件存在:

```
if(!Files.exists(path)){
    Files.createFile(path);
}
```

### 16.7、Writing Data Via a CompletionHandler

您还可以使用CompletionHandler将数据写入到asynousfilechannel，以告诉您何时完成写入，而不是将来完成。下面是一个使用CompletionHandler向异步filechannel写入数据的例子:

```
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
    Files.createFile(path);
}
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

fileChannel.write(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {

    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("bytes written: " + result);
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.out.println("Write failed");
        exc.printStackTrace();
    }
});
```

当写操作完成时，CompletionHandler的completed()方法将被调用。如果写操作由于某种原因失败，则会调用failed()方法。

注意ByteBuffer是如何作为附件使用的——它是传递给CompletionHandler方法的对象。