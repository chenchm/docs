# Netty NIO笔记

## Netty架构

Netty NIO对`Java NIO` 进行了封装，简化了`Java NIO`编码工作和存在的BUG。

### Reactor线程模型

在介绍Netty线程模型之前，先简单介绍Reactor线程模型，Reactor是基于事件驱动的模型，分为三个角色：

- Acceptor：负责接收Accept事件，然后将连接注册成读事件，传递给dispatch;
- Dispatch：负责分发事件，如果是接收到Accept事件,则分发给Acceptor。如果是Read事件，将请求交给ReadHandler处理；
- Handler：负责处理具体事件；

Reactor线程模型关注的是：任务接受之后，对处理过程继续进行切分，划分为多个不同的步骤，每个步骤用不同的线程来处理，也就是原本由一个线程处理的任务现在由多个线程来处理，每个线程在处理完自己的步骤之后，还需要将任务转发到下阶段线程继续进行处理。

Netty中的Reactor模型如下：

![reactor_model](../../images/reactor_model)

其中mainReactor线程池负责处理连接请求；当acceptor接受连接后会将连接注册到subReactor的线程上；由subReactor来处理IO请求；而ThreadPool可以用来处理耗时的业务逻辑。虽然 Netty 的线程模型基于主从 Reactor 多线程模型，但是做了部分调整和修改，将acceptor变成了mainReactor中的一个线程，并且默认使用subReactor线程池中的线程来处理compute工作，同时也支持使用独立的线程池处理compute工作，这需要进行手动配置。

### Netty线程模型

![netty_model](../../images/netty_model)

#### Netty组件

##### NioEventLoop和NioEventLoopGroup

NioEventLoopGroup实际上就是一个线程池，线程池中有多个NioEventLoop工作线程，如果不进行手动设置默认工作线程数量为物理机cpu核心数量的2倍。

每个NioEventLoop都绑定了一个Selector用于监听注册到Selector的Channel的IO事件，NioEventLoop的工作包括IO事件监听、IO事件处理、以及交给NioEventLoop的其他任务三个部分。由于Netty模型中有多个NioEventLoop，所以存在多组Selector在监听IO就绪事件。

一个NioEventLoop的Selector可以被多个Channel注册，也就是说多个Channel共享一个EventLoop。EventLoop的Selecctor对所以绑定到NioEventLoop的Channel的IO事件进行监听。但一个Channel只能绑定到一个NioEventLoop相当于一个连接绑定一个线程，这个连接所有的ChannelHandler都是在一个线程中执行的，避免了多线程干扰。更重要的是ChannelPipline链表必须严格按照顺序执行的。单线程的设计能够保证ChannelHandler的顺序执行。

> 在监听一个端口的情况下，一个NioEventLoop通过一个NioServerSocketChannel监听端口，处理TCP连接请求。后端多个NioEventLoop工作线程处理IO事件。每个Channel绑定一个NioEventLoop线程，一个NioEventLoop线程关联一个Selector来为多个注册到它的Channel监听IO就绪事件。NioEventLoop是单线程执行，保证ChannelPipline在单线程中执行，保证了ChannelHandler的执行顺序。

##### Channel

Netty中的通道是对Java原生网络API的封装，其顶层接口为Channel。

以TCP编程为例 ，在Java中，有两种方式：

- 基于BIO，JDK1.4之前，我们通常使用java.net包中的ServerSocket和Socket来代表服务端和客户端。

- 基于NIO，Jdk1.4引入nio编程之后，我们使用java.nio.channels包中的ServerSocketChannel和SocketChannel来代表服务端与客户端。

在Netty中，对java中的BIO、NIO编程api都进行了封装，分别为：

- 使用了OioServerSocketChannel，OioSocketChannel对java.net包中的ServerSocket与Socket进行了封装

- 使用NioServerSocketChannel和NioSocketChannel对java.nio.channels包中的ServerSocketChannel和SocketChannel进行了封装。

由于本文只对NIO进行说明，所以就不对BIO相关的内容进行深入，Netty中NIO相关的Channel类关系图如下：

![netty_nio_channel](..\images\netty_nio_channel.png)

NioServerSocketChannel和NioSocketChannel中都持有对应的Java NIO Channel的引用，所以NioServerSocketChannel注册到NioEventLoop，实际上就是Java NIO Channel注册到绑定在NioEventLoop上的Selector。二者都继承自AbstractNioChannel，其维护了Netty中的Channel与Java NIO中Channel的对应关系，并提供了`javaChannel()`方法获取对应的Java Channel。

```java
public abstract class AbstractNioChannel extends AbstractChannel {
    ...
    private final SelectableChannel ch;
    ...
    // 获取对应的java channel
    protected SelectableChannel javaChannel() {
        return ch;
    }
}
```

SelectableChannel是Java NIO中的类，我们前面提到nio包中的SocketChannel、ServerSocketChannel都是其子类，NioSocketChannel和NioServerSocketChannel对AbstractNioChannel的`javaChannel()`进行了重写，将SelectableChannel强转为对应Java Channel，如下：

- io.netty.channel.socket.nio.NioServerSocketChannel#javaChannel

```java
@Override
// 返回java.nio.channels.ServerSocketChannel
protected ServerSocketChannel javaChannel() {
  return (ServerSocketChannel) super.javaChannel();
}
```

- io.netty.channel.socket.nio.NioSocketChannel#javaChannel

```java
@Override
// 返回java.nio.channels.SocketChannel
protected SocketChannel javaChannel() {
    return (SocketChannel) super.javaChannel();
}
```

在创建NioServerSocketChannel和NioSocketChannel对象时，会在构造函数中为其指定对应的ServerSocket和Socket实例，以NioServerSocketChannel为例：

```java
public class NioServerSocketChannel extends AbstractNioMessageChannel
                             implements io.netty.channel.socket.ServerSocketChannel {
    private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();
    private static ServerSocketChannel newSocket(SelectorProvider provider) {
        try {
           // 生成一个新的ServerSocketChannel对象
           return provider.openServerSocketChannel();
        } catch (IOException e) {
            throw new ChannelException(
                    "Failed to open a server socket.", e);
        }
    }

    public NioServerSocketChannel() {
        this(newSocket(DEFAULT_SELECTOR_PROVIDER));
    }

    public NioServerSocketChannel(SelectorProvider provider) {
        this(newSocket(provider));
    }

  	public NioServerSocketChannel(ServerSocketChannel channel) {
        super(null, channel, SelectionKey.OP_ACCEPT);
        config = new NioServerSocketChannelConfig(this, javaChannel().socket());
    }
    
  	...
}
```

##### ChannelHandler

在NIO编程中，我们经常需要对Channel的输入和输出事件进行处理，Netty抽象出一个ChannelHandler概念，专门用于处理此类事件，因此编写Handler是我们进行Netty框架编程时主要编码工作。IO事件分为出站和入站，因此ChannelHandler又具体的分为ChannelInboundHandler和ChannelOutboundHandler ，分别用于出站和入站时某个阶段事件的处理。

ChannelHanler的类关系图如下：

![netty_channel_hander](..\images\netty_channel_handler.png)

ChannelHandlerAdapter、ChannelInboundHandlerAdapter 、ChannelOutboundHandlerAdapter是Netty提供的Handler骨架类已经实现了大部分骨架方法，对于不同的需求你只要相应的Handler并重写自己感兴趣的方法即可。这里再简单介绍一下SimpleChannelInboundHandler和ChannelInitializer这两个类：

- SimpleChannelInboundHandler

	SimpleChannelInboundHandler继承自ChannelInboundHandlerAdapter用于处理入站信息，需要指定泛型即该Handler需要处理的对象类型，当传入对象类型不正确时不进行数据处理。使用时只需要实现`channelRead0`方法进行数据处理即可，使用该类好处在于SimpleChannelInboundHandler会自动为你释放ReferenceCounted类型对象（例如ByteBuf）.

- ChannelInitializer

	ChannelInitializer类的`initChannel`方法会在Channel初始化被调用，所以一般通过重写`initChannel`方法配置pipline中的ChannelHandler链。

在处理Channel的IO事件时，为了使逻辑更清晰和代码的简洁性通常会将数据处理过程分为几个阶段。以入站信息为例，通常的处理逻辑如下：

处理半包或者粘包问题-->数据的解码(或者说是反序列化)-->数据的业务处理

可以看到不同的阶段要执行不同的功能，因此通常我们会编写多个ChannelHandler，来实现不同的功能。而且多个ChannelHandler之间的顺序不能颠倒，例如我们必须先处理粘包解包问题，之后才能进行数据的业务处理。

##### ChannelPipline

Netty通过ChannelPipline来保证多个ChannelHandler之间的顺序调用，具体是通过一个双向链表实现的，下文将会进行介绍。在每个Channel被创建时，都会自动创建一个ChannelPipeline对象，我们可以通过io.netty.channel.Channel对象的pipeline()方法获取这个对象实例。

ChannelPipeline创建是在AbstractChannel类的构造方法中，所以Netty中每个Channel都会拥有和维护一个唯一的ChannelPipeline，如下：

```java
public abstract class AbstractChannel extends DefaultAttributeMap implements Channel {
    ....
    private final DefaultChannelPipeline pipeline;
    ....
    
    protected AbstractChannel(Channel parent) {
        this.parent = parent;
        id = newId();
        unsafe = newUnsafe();
        // 初始化pipline
        pipeline = newChannelPipeline();
    }
    ....
   
    // 创建一个DefaultChannelPipeline并把Channel自身作为引用传入
    protected DefaultChannelPipeline newChannelPipeline() {
        return new DefaultChannelPipeline(this);
    }
    ....
    
    @Override
    // 返回pipeline实例
    public ChannelPipeline pipeline() {
        return pipeline;
    }
}
```

从上述代码中，我们可以看到ChannelPipleLine的具体创建过程实际上是通过return new DefaultChannelPipeline(this)实现的。DefaultChannelPipeline是ChannelPipeline的默认实现类。

我们可以通过ChannelPipeline的`addXX`方法向pipline的双向链表中添加Handler来维护Handler之间的调用关系，常用的添加方法有`addBefore, addAfter, addLast`等。

> 1、默认情况下，一个ChannelPipeline实例中，同一个类型ChannelHandler只能被添加一次，如果添加多次，则会抛出异常，具体参见io.netty.channel.DefaultChannelPipeline#checkMultiplicity。如果需要多次添加同一个类型的ChannelHandler的话，则需要在该ChannelHandler实现类上添加@Sharable注解。
>
> 2、在ChannelPipeline中，每一个ChannelHandler都是有一个名字的，而且名字必须的是唯一的，如果名字重复了，则会抛出异常，参见io.netty.channel.DefaultChannelPipeline#checkDuplicateName。
>
> 3、如果添加ChannelHanler的时候没有显示的指定名字，则会按照规则其起一个默认的名字。具体规则如下，如果ChannelPipeline中只有某种类型的handler实例只有一个，如XXXHandler,YYYHandler，则其名字分别为XXXHandler#0，YYYHandler#0，如果同一类型的Handler有多个实例，则每次之后的编号加1。具体可参见io.netty.channel.DefaultChannelPipeline#generateName方法。

##### ChannelHandlerContext

前面提到可以通过ChannelPipeline的添加方法，按照顺序添加ChannelHandler，并在之后按照顺序进行调用。事实上，每个ChannelHandler会被先封装成ChannelHandlerContext，之后再封装进ChannelPipeline中，所以ChannelPipline维护的双向链表实际上是ChannelHandlerContext构成的链表。以DefaultChannelPipeline的addLast方法为例：

```java
@Override
public ChannelPipeline addLast(EventExecutorGroup group, final String name, ChannelHandler handler) {
    synchronized (this) {
        // 检查这种类型的handler实例是否允许被添加多次
        checkDuplicateName(name);
       	// 将handler包装成一个DefaultChannelHandlerContext类
        AbstractChannelHandlerContext newCtx = new DefaultChannelHandlerContext(this, group, name, handler);
        // 维护AbstractChannelHandlerContext的先后关系
        addLast0(name, newCtx);
    }

    return this;
}
```

可以看到的确是先将ChannelHandler当做参数构建成一个DefaultChannelHandlerContext实例之后，再调用addLast0方法维护ChannelHandlerContext的先后关系，从而确定了ChannelHandler的先后关系。可以注意到ChannelHandlerContext对象持有ChannelPipeline、EventExecutorGroup、ChannelHandler对象的引用，因此ChannelHandlerContext可以方便得获得和使用这些对象，当有方法的参数为ChannelHandlerContext时就可以很方便地使用其引用的对象，我觉得这应该是将Handler封装为ChannelHandlerContext的主要原因。

DefaultChannelPipeline内部是通过一个双向链表记录ChannelHandler的先后关系，内部通过两个哨兵节点HeadContext和TailContext作为链表的开始和结束，双向链表的节点是AbstractChannelHandlerContext类。以下是AbstractChannelHandlerContext类的部分源码(双向链表节点)：

```java
public class DefaultChannelPipeline implements ChannelPipeline {
    ...
    private static final String HEAD_NAME = generateName0(HeadContext.class);
    private static final String TAIL_NAME = generateName0(TailContext.class);
    ...
    final AbstractChannelHandlerContext head;
    final AbstractChannelHandlerContext tail;

    private final Channel channel;
    ....
    protected DefaultChannelPipeline(Channel channel) {
        this.channel = ObjectUtil.checkNotNull(channel, "channel");
        ....
        // 创建双向链表头部元素实例
        tail = new TailContext(this);
        head = new HeadContext(this);
        // 创建双向链表的尾部元素实例
        head.next = tail;
        tail.prev = head;
    }
    ....
    ....
    private void addLast0(AbstractChannelHandlerContext newCtx) {
       	// 设置ChannelHandler的先后顺序关系
        AbstractChannelHandlerContext prev = tail.prev;
        newCtx.prev = prev;
        newCtx.next = tail;
        prev.next = newCtx;
        tail.prev = newCtx;
       }
     }
}
```

头尾两个节点与其他Handler不同点在于，它们持有了Unsafe对象，从而能通过Unsafe对象对Java NIO组件进行操作。

## Netty源码分析

### 从服务端的初始化说起

让我们以一个简单的EchoServer为例子进行说明：

```java
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public void start() throws Exception {
        // boss线程池，用于接受连接请求
        EventLoopGroup boss = new NioEventLoopGroup(1);
        // workers线程池，用于执行IO请求
        EventLoopGroup workers = new NioEventLoopGroup();

        try {
            // 启动类，用于配置并辅助启动服务
            ServerBootstrap b = new ServerBootstrap();
            // 配置线程池
            b.group(boss, workers)
                // 设置要启动的Channel类型
                .channel(NioServerSocketChannel.class)
                // 配置NioServerSocketChannel的参数
                .option(ChannelOption.SO_BACKLOG, 128)
                // 配置子Channel的Handler链
                .childHandler(new EchoServerInitializer());
            
            // 线程绑定监听端口，启动Netty服务端
            ChannelFuture f = b.bind(port).sync();
            f.channel().closeFuture().sync();
        } finally {
            boss.shutdownGracefully();
            workers.shutdownGracefully();
        }
    }
}
```

可以看出在Netty服务启动之前，主要进行了Netty服务参数配置，其含义我们先按下不表，真正的开启服务的方法很显然是ServerBootstrap的`bind`方法，bind方法的调用链如下：

```java
public ChannelFuture bind(int inetPort) {
    // 使用端口号创建InetSocketAddress作为参数
    return bind(new InetSocketAddress(inetPort));
}

public ChannelFuture bind(SocketAddress localAddress) {
    // 校验工作
    validate();
    if (localAddress == null) {
        throw new NullPointerException("localAddress");
    }
    return doBind(localAddress);
}

// 核心方法
private ChannelFuture doBind(final SocketAddress localAddress) {
    // 创建接受连接请求的Channel并注册到NioEventLoop的Selector
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    if (regFuture.cause() != null) {
        return regFuture;
    }

    // 判断操作是否完成
    if (regFuture.isDone()) {
        ChannelPromise promise = channel.newPromise();
        // 如果操作完成调用doBind0方法
        doBind0(regFuture, channel, localAddress, promise);
        return promise;
    } else {
        final PendingRegistrationPromise promise = new PendingRegistrationPromise(channel);
        // 如果操作还未完成则添加监听器，在完成后判断操作是否失败，如果未失败则继续调用doBind0方法
        regFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                Throwable cause = future.cause();
                if (cause != null) {
                    promise.setFailure(cause);
                } else {
                    promise.registered();
                    doBind0(regFuture, channel, localAddress, promise);
                }
            }
        });
        return promise;
    }
}
```

通过bind方法的调用链，最终将调用ServerBootstrap的doBind方法，doBind主要有两个步骤一是通过`initAndRegister`方法创建Channel并注册注册到NioEventLoop的Selector上用于接受连接请求；二是通过`doBind0`方法将Java管道绑定到要监听的端口。

现在我们依次分析这两个方法，首先来看`initAndRegister`方法

```java
final ChannelFuture initAndRegister() {
    Channel channel = null;
    try {
        // 使用channelFactory创建NioSocketChannelServer
        channel = channelFactory.newChannel();
        // 初始化管道
        init(channel);
    } 
    // 如果创建或初始化管道失败了，返回包含错误原因的ChannelFuture
    catch (Throwable t) {
        if (channel != null) {
            channel.unsafe().closeForcibly();
            return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
        }
        return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
    }

    // 将管道注册到NioEventLoop的Selector上
    ChannelFuture regFuture = config().group().register(channel);
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) {
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    }

    return regFuture;
}
```

我们可以将`initAndRegister`方法的逻辑拆分成三个部分：

- 通过channelFactory.newChannel()，创建Channel(实际上是NioServerSocketChannel)
- 初始化创建的管道
- 通过config().group().register(channel)，将管道注册到NioEventLoop的Selector上

#### 使用ChannelFactory创建Channel

ChannelFactory是怎么来的呢，让我们回到调用EchoServe类调用bind方法启动之前的配置逻辑上来：

```java
// 配置线程池
b.group(boss, workers)
    // 设置要启动的Channel类型
    .channel(NioServerSocketChannel.class)
    // 配置子Channel的Handler链
    .childHandler(new EchoServerInitializer());
```

其中channel方法配置了管道类型为NioServerSocketChannel.class，让我们进入该方法的调用链：

```java
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    // 以channelClass为参数，这里是NioServerSocketChannel.class，创建ReflectiveChannelFactory
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}

public B channelFactory(io.netty.channel.ChannelFactory<? extends C> channelFactory) {
    return channelFactory((ChannelFactory<C>) channelFactory);
}

public B channelFactory(ChannelFactory<? extends C> channelFactory) {
    if (channelFactory == null) {
        throw new NullPointerException("channelFactory");
    }
    if (this.channelFactory != null) {
        throw new IllegalStateException("channelFactory set already");
    }

    // 将ServerBootstrap的channelFactory设置为ReflectiveChannelFactory
    this.channelFactory = channelFactory;
    return self();
}
```

可以看到`channel`为ServerBootstrap创建了ReflectiveChannelFactory对象，并用它初始化了ServerBootstrap的channelFactory域。

现在让我们回到`channelFactory.newChannel()`逻辑，进入newChannel方法，如下：

```java
public T newChannel() {
    try {
        return constructor.newInstance();
    } catch (Throwable t) {
        throw new ChannelException("Unable to create Channel from class " + constructor.getDeclaringClass(), t);
    }
}
```

newChannel方法通过反射调用创建了Channel对象，从上面的分析可以知道这里的Channel类为NioServerSocketChannel

所以让我们进入NioServerSocketChannel的无参构成函数的调用链，如下：

```java
public NioServerSocketChannel() {
    // 创建ServerSocketChannel作为构造函数参数
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
}

private static ServerSocketChannel newSocket(SelectorProvider provider) {
    try {
        // 通过Java NIO的SelectorProvider来构建Java Channel
        return provider.openServerSocketChannel();
    } catch (IOException e) {
        throw new ChannelException(
            "Failed to open a server socket.", e);
    }
}

public NioServerSocketChannel(ServerSocketChannel channel) {
    // 设置readInterestOp的值为SelectionKey.OP_ACCEPT，即监听连接请求
    super(null, channel, SelectionKey.OP_ACCEPT);
    // 初始化NettyNio配置对象，用于快速获得channel参数信息
    config = new NioServerSocketChannelConfig(this, javaChannel().socket());
}

// 中间对象AbstractNioMessageChannel
protected AbstractNioMessageChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent, ch, readInterestOp);
}

protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    // 设置Netty NioChannel持有的Java Channel引用
    this.ch = ch;
    // 设置Channel监听的操作类型
    this.readInterestOp = readInterestOp;
    try {
        // 设置Java Channel为非阻塞的
        ch.configureBlocking(false);
    } catch (IOException e) {
        try {
            ch.close();
        } catch (IOException e2) {
            if (logger.isWarnEnabled()) {
                logger.warn(
                    "Failed to close a partially initialized socket.", e2);
            }
        }

        throw new ChannelException("Failed to enter non-blocking mode.", e);
    }
}

protected AbstractChannel(Channel parent) {
    this.parent = parent;
    // 生成Channel的全局Id
    id = newId();
    // 初始化Unsafe对象
    unsafe = newUnsafe();
    // 初始化ChannelPipeline
    pipeline = newChannelPipeline();
}
```

从NioServerSocketChannel构造函数的调用链可以看出，在创建Netty NioChannel时会初始对应的Java Channel并且将Java Channel配置为非阻塞的。之后进入AbstractChannel的逻辑，初始化Netty NioChannel的相关属性：

- 调用newId()，生成一个Channel的全局Id
- 调用newUnsafe()，生成Unsafe对象，Unsafe能够处理Java NIO相关的逻辑，例如在NioEventLoop的Selector上注册Java Channel，或者Java Channel绑定到对应的端口。
- 调用newChannelPipeline()，生成DefaultChannelPipeline对象，这个对象在[Netty组件]()中介绍过，是用来维护Netty Handler处理链的。

[Netty组件]()中对DefaultChannelPipeline对象的分析可以知道，在DefaultChannelPipeline初始化完成时，pipeline中只有头节点（head）和尾节点（tail）两个节点，其中头结点同时实现了`ChannelOutboundHandler`和`ChannelInboundHandler`这表示头结点即可以处理入站信息也可以处理出站信息；而尾节点只实现了`ChannelInboundHandler`所以只能处理入站信息，如下：

```java
// 头结点实现的接口
final class HeadContext extends AbstractChannelHandlerContext
            implements ChannelOutboundHandler, ChannelInboundHandler {

    private final Unsafe unsafe;

    HeadContext(DefaultChannelPipeline pipeline) {
        super(pipeline, null, HEAD_NAME, true, true);
        unsafe = pipeline.channel().unsafe();
        setAddComplete();
    }
    ...
}

// 尾节点实现的接口
final class TailContext extends AbstractChannelHandlerContext implements ChannelInboundHandler {
    TailContext(DefaultChannelPipeline pipeline) {
        super(pipeline, null, TAIL_NAME, true, false);
        setAddComplete();
    }
}
```

此时pipeline的链表结构如下：

![netty_ctx](..\images\netty_ctx.png)

至此，整个NioServerSocketChannel的创建工作就完成了，接下将进行NioServerSocketChannel初始化工作。

#### 初始化NioServerSocketChannel

完成管道的创建工作后，继续从ServerBootstrap的`initAndRegister`方法往下走，接下来将调用`init`方法来进行管道的初始化工作，`init`方法如下：

```java
void init(Channel channel) throws Exception {
    // 获得通过Bootstrap对象的option方法配置的参数
    final Map<ChannelOption<?>, Object> options = options0();
    synchronized (options) {
        // 将配置应用到相应的对象上，如java channel或java socket
        setChannelOptions(channel, options, logger);
    }

    // 获得通过Bootstrap对象的attr方法配置的属性
    final Map<AttributeKey<?>, Object> attrs = attrs0();
    synchronized (attrs) {
        // 使用for循环修改属性对应的值
        for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
            @SuppressWarnings("unchecked")
            AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
            channel.attr(key).set(e.getValue());
        }
    }

    // 获得NioServerSocketChannel的pipline对象
    ChannelPipeline p = channel.pipeline();

    // 获得子Channel参数信息，用于创建ServerBootstrapAcceptor
    // ServerBootstrapAcceptor是一个处理连接请求的Handler，会为连接请求创建对应的Channel
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(0));
    }
    synchronized (childAttrs) {
        currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(0));
    }

    // 在pipline中添加一个ChannelInitializer
    // 当ChannelInitializer的initChannel被调用时，可以对管道进行操作
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(final Channel ch) throws Exception {
            final ChannelPipeline pipeline = ch.pipeline();
            // 在pipline中添加通过Bootstrap对象的handler配置的Handler
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }

            // 向EventLoop线程的任务队列提交创建ServerBootstrapAcceptor对象的任务
            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    pipeline.addLast(new ServerBootstrapAcceptor(
                        ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```

可以看出，`init`方法主要做了以下工作：

- 将Bootstrap对象的option方法配置的参数配置到Config对象上
- 将Bootstrap对象的attr方法配置的属性应用到channel对应的attr上
- 调用addLast方法在NioServerSocketChannel的pipline尾部添加了一个ChannelInitializer（ChannelInboundHandlerAdapter的一个子类），ChannelInitialize的initChannel方法会在后续的管道初始化过程中被调用，所以这里先不对initChannel方法进行深入

让我们进入pipline的`addLast`方法的调用链，看看做了哪些操作：

```java
public final ChannelPipeline addLast(ChannelHandler... handlers) {
    // 这里的第一个参数为EventExecutorGroup，作用是指定的执行Handler逻辑的线程池
    // 设为null则表示使用BootStrap中配置的NioEeventLoopGroup线程池
    return addLast(null, handlers);
}

public final ChannelPipeline addLast(EventExecutorGroup executor, ChannelHandler... handlers) {
    if (handlers == null) {
        throw new NullPointerException("handlers");
    }

    // 通过for循环中的addLast方法将可变参数中的handler加入pipline
    for (ChannelHandler h: handlers) {
        if (h == null) {
            break;
        }
        addLast(executor, null, h);
    }

    return this;
}

public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        // 校验handler信息
        checkMultiplicity(handler);

        // 构建HandlerContext对象，并且根据name是否为空校验或者生成handler名字
        newCtx = newContext(group, filterName(name, handler), handler);

        // 将HandlerContext对象添加到pipeline末尾
        addLast0(newCtx);

        // 判断管道是否注册到EventLoop
        if (!registered) {
            // 将handler的阶段从INIT设置为PENDING
            newCtx.setAddPending();
            // 创建一个稍后执行的callHandlerAdded0方法的回调任务
            callHandlerCallbackLater(newCtx, true);
            return this;
        }

        // 如果管道已注册到EventLoop
        // 因为ctx中executor对象为空，所以这里的exector是Channel持有的EventLoop对象
        EventExecutor executor = newCtx.executor();
        // 如果当前线程不是EventLoop的工作线程
        if (!executor.inEventLoop()) {
            // 将callHandlerAdded0的调用封装为task提交到指定的工作线程池
            callHandlerAddedInEventLoop(newCtx, executor);
            return this;
        }
    }
    
    // 如果当前线程是EventLoop的工作线程
    // 则立刻执行callHandlerAdded0方法
    callHandlerAdded0(newCtx);
    return this;
}
```

可以看到addLast对handler主要做了以下工作：

- 对handler信息进行校验

```java
private static void checkMultiplicity(ChannelHandler handler) {
    // 如果handler是ChannelHandlerAdapter的子类
    if (handler instanceof ChannelHandlerAdapter) {
        ChannelHandlerAdapter h = (ChannelHandlerAdapter) handler;
        
        // 判断handler是不是已经被添加到管道了
        // 如果已经添加到管道并且handler没有被@shareable注解标识，抛出异常
        if (!h.isSharable() && h.added) {
            throw new ChannelPipelineException(
                h.getClass().getName() +
                " is not a @Sharable handler, so can't be added or removed multiple times.");
        }
        // 将handler已经被添加到管道的标志设置为true
        h.added = true;
    }
}
```

- 调用newContext方法，将handler包装成AbstractChannelHandlerContext对象
- 调用addLast0将构建的AbstractChannelHandlerContext加入pipline双向链表尾部

```java
private void addLast0(AbstractChannelHandlerContext newCtx) {
    AbstractChannelHandlerContext prev = tail.prev;
    newCtx.prev = prev;
    newCtx.next = tail;
    prev.next = newCtx;
    tail.prev = newCtx;
}
```

- 接下来就是分情况进行callHandlerAdded0方法的调用，如果NioServerSocketChannel未注册到EventLoop则调用callHandlerCallbackLater方法创建一个稍后被执行的Task；如果NioServerSocketChannel已经注册到EventLoop，并且如果当前线程不是EventLoop的工作线程，则callHandlerAdded0方法会提交到线程池中被执行；如果NioServerSocketChannel已经注册到EventLoop并且当前线程是EventLoop的工作线程，则使用当前线程执行callHandlerAdded0方法。显然这时候NioServerSocketChannel还未注册到EventLoop，所以让我们接着进入callHandlerCallbackLater方法。

````java
private void callHandlerCallbackLater(AbstractChannelHandlerContext ctx, boolean added) {
    assert !registered;

    // 根据调用传入的参数，这里显然是新建一个PendingHandlerAddedTask对象
    PendingHandlerCallback task = added ? new PendingHandlerAddedTask(ctx) : new PendingHandlerRemovedTask(ctx);
    // 获得PendingHandlerCallback链表的头结点，PendingHandlerCallback链表是一个单向链表
    PendingHandlerCallback pending = pendingHandlerCallbackHead;
    // 如果链表为空则将新的Task作为链表头，否则将Task插入链表尾部
    if (pending == null) {
        pendingHandlerCallbackHead = task;
    } else {
        // Find the tail of the linked-list.
        while (pending.next != null) {
            pending = pending.next;
        }
        pending.next = task;
    }
}

// PendingHandlerAddedTask实现了Runnable和PendingHandlerCallback类的execute抽象方法
// 两个方法的目的都是执行callHandlerAdded0方法
private final class PendingHandlerAddedTask extends PendingHandlerCallback {
    PendingHandlerAddedTask(AbstractChannelHandlerContext ctx) {
        super(ctx);
    }

    @Override
    public void run() {
        callHandlerAdded0(ctx);
    }

    @Override
    void execute() {
        EventExecutor executor = ctx.executor();
        if (executor.inEventLoop()) {
            callHandlerAdded0(ctx);
        } else {
            try {
                    executor.execute(this);
                } catch (RejectedExecutionException e) {
                    ...
                }
        }
    }
}
````

该方法只是在PendingHandlerCallback链表中加入了一个新的PendingHandlerAddedTask而没有执行callHandlerAdded0方法，该方法调用的时机会在下文阐述。

自此addLast方法的调用链分析完成，这时候NioServerSocketChannel的pipline的链表结构应该如下：

![](..\images\netty_ctx_link2.png)

#### 将NioServerSocketChannel注册到EventLoop

初始化管道的工作完成后，接下来就是调用`config().group().register(channel)`进行Channel的注册工作了。这里的group方法返回的是MultithreadEventLoopGroup对象，该对象的register方法如下：

```java
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
```

MultithreadEventLoopGroup对象的register可以被分为两个部分：

- 调用next方法会使用选择器返回一个EventLoop对象

```java
// MultithreadEventLoopGroup.java
public EventLoop next() {
    return (EventLoop) super.next();
}

// MultithreadEventExecutorGroup.java
public EventExecutor next() {
    return chooser.next();
}

// GenericEventExecutorChooser
public EventExecutor next() {
    // 使用idx对线程池长度进行取模，这样Channel被平均分配到线程池的线程上
    return executors[Math.abs(idx.getAndIncrement() % executors.length)];
}
```

- 调用SingleThreadEventLoop的register方法来注册Channel

```java
public ChannelFuture register(Channel channel) {
    return register(new DefaultChannelPromise(channel, this));
}

public ChannelFuture register(final ChannelPromise promise) {
    ObjectUtil.checkNotNull(promise, "promise");
    // 调用Channel的AbstractUnsafe对象的register方法来注册管道
    promise.channel().unsafe().register(this, promise);
    return promise;
}
```

可以看出SingleThreadEventLoop的register方法最构建一个DefaultChannelPromise对象，DefaultChannelPromise对象是一个用来存储异步任务结果并作为异步任务观察者的实例；终会调用Channel的Unsafe对象的register方法来注册管道，所以让我们继续进入AbstractUnsafe对象的register方法：

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    // 一些校验工作
    ...

    // AbstractUnsafe是AbstractChannel的一个内部类
    // 所以用AbstractChannel.this.eventLoop来指定当前的eventLoop
    AbstractChannel.this.eventLoop = eventLoop;

    // 判断当前线程是不是传入的EventLoop对象的工作线程
    if (eventLoop.inEventLoop()) {
        // 如果是就直接执行register0
        register0(promise);
    } else {
        try {
            // 如果不是则将register0作为一个Task放入eventLoop的任务队列
            eventLoop.execute(new Runnable() {
                @Override
                public void run() {
                    register0(promise);
                }
            });
        } catch (Throwable t) {
            logger.warn(
                "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                AbstractChannel.this, t);
            closeForcibly();
            closeFuture.setClosed();
            safeSetFailure(promise, t);
        }
    }
}
```

可以看到register方法主要做了两件事，首先设置当前Channel的EventLoop，然后根据当前线程是否是EventLoop中的工作线程来决定如何执行register0方法，毋庸置疑在整个Service服务起来的过程中执行线程是主线程，所以这里的register0方法会在EventLoop的工作队列中等待执行，也就是说register0方法中所有的逻辑都是由EventLoop的工作线程来完成的。任务在何时开始执行与Netty线程模型有关，这里我们暂时不进行深入，先进入异步执行的register0方法：

```java
private void register0(ChannelPromise promise) {
    try {
        if (!promise.setUncancellable() || !ensureOpen(promise)) {
            return;
        }
        boolean firstRegistration = neverRegistered;
        // 将Java Channel注册到Selector
        doRegister();
        // 将从未注册字段设置为false
        neverRegistered = false;
        // 将已经注册变量设置为true
        registered = true;

        // 在通知监听器之前执行PendingHandlerCallback链中callback的逻辑
        pipeline.invokeHandlerAddedIfNeeded();

        // 将Promise的中执行结果标记为SUCCESS，并通知所有的监听器执行任务
        safeSetSuccess(promise);
        // 调用pipline的fireChannelRegistered，执行管道注册完成的逻辑
        pipeline.fireChannelRegistered();
       	
        // 如果Java Channel处于绑定状态
        if (isActive()) {
            // 如果是第一次进行注册
            if (firstRegistration) {
                // 调用pipline的fireChannelActive，执行管道激活完成的完成的逻辑
                pipeline.fireChannelActive();
            } else if (config().isAutoRead()) {
                beginRead();
            }
        }
    } catch (Throwable t) {
        // Close the channel directly to avoid FD leak.
        closeForcibly();
        closeFuture.setClosed();
        safeSetFailure(promise, t);
    }
}
```

register0方法是Netty初始过程中比较核心的代码，主要可以分成以下几块逻辑：

- 调用doRegister将Java Channel注册到Selector

```java
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                eventLoop().selectNow();
                selected = true;
            } else {
                throw e;
            }
        }
    }
}
```

将doRegister方法中将Java Channel（这里是ServerSocketChannel）注册到了EventLoop的Selector上，可以注意到这里注册的事件是0，即不监听任何事件，监听的事件将在后续进行注册，下文将会提到；为什么要用死循环的原因目前我并不太明白。。。

- 将registered标记字段设置为true，表明该Channel已经完成了注册
- 调用` pipeline.invokeHandlerAddedIfNeeded()`，该方法会执行之前在pipline的addLast方法中进行过说明的PendingHandlerCallback链中所有的Task的execute方法，也就是说callHandlerAdded0方法将在这里进行调用。

```java
final void invokeHandlerAddedIfNeeded() {
    // 由于register0方法是由EventLoop中的工作线程执行的
    // 所以当前线程必须是EventLoop中的工作线程，这里进行一次判断校验
    assert channel.eventLoop().inEventLoop();
    if (firstRegistration) {
        // firstRegistration属性的值修改为false
        firstRegistration = false;
        // 执行回调逻辑
        callHandlerAddedForAllHandlers();
    }
}

private void callHandlerAddedForAllHandlers() {
    final PendingHandlerCallback pendingHandlerCallbackHead;
    synchronized (this) {
        assert !registered;
	   	// 修改pipline的registered属性的值为true
        registered = true;

        // 获得链表的头结点
        pendingHandlerCallbackHead = this.pendingHandlerCallbackHead;
        this.pendingHandlerCallbackHead = null;
    }

    // 从头结点开始沿着回调链执行回调的execute方法
    PendingHandlerCallback task = pendingHandlerCallbackHead;
    while (task != null) {
        task.execute();
        task = task.next;
    }
}
```

由上述流程可以知道invokeHandlerAddedIfNeeded最终将会执行PendingHandlerCallback链中所有的task的execute方法，所以之前通过pipline的addLast方法最终添加到链中的PendingHandlerAddedTask对象的execute方法也在此时被执行，让我们回到PendingHandlerAddedTask的execute方法：

```java
void execute() {
    // 因为ctx中executor对象为空，所以这里的exector是Channel持有的EventLoop对象
    EventExecutor executor = ctx.executor();
    // 判断当前线程是不是executor的工作线程
    if (executor.inEventLoop()) {
        // 直接执行callHandlerAdded0方法
        callHandlerAdded0(ctx);
    } else {
        try {
            // 把自己作为task提交，也就是会在稍后执行自身的run方法
            // run方法中也是执行的callHandlerAdded0方法
            executor.execute(this);
        } catch (RejectedExecutionException e) {
            if (logger.isWarnEnabled()) {
                logger.warn(
                    "Can't invoke handlerAdded() as the EventExecutor {} rejected it, removing handler {}.",
                    executor, ctx.name(), e);
            }
            remove0(ctx);
            ctx.setRemoved();
        }
    }
}

public void run() {
    callHandlerAdded0(ctx);
}
```

由之前对AbstractUnsafe对象的register方法分析可知，AbstractUnsafe对象的register0方法是由EventLoop中的工作线程去执行的，所以register0中所有的逻辑都由EventLoop中的工作线程来完成，因此`executor.inEventLoop()`得到的结果为true，所以接下将直接执行DefaultChannelPipline的callHandlerAdded0方法：

```java
private void callHandlerAdded0(final AbstractChannelHandlerContext ctx) {
    try {
        ctx.callHandlerAdded();
    } catch (Throwable t) {
        // 错误处理逻辑
        ...
    }
}
```

目前我们只在ServerBootstrap的`init`方法中在链表末尾添加了一个添加了一个ChannelInitializer，所以这里的HandlerContext就是封装ChannelInitializer的HandlerContext。接着进入HandlerContext的callHandlerAdded方法：

```java
final void callHandlerAdded() throws Exception {
   	// 将Handler的状态修改为ADD_COMPLETE
    if (setAddComplete()) {
        // 通过handler方法获得context中的handler，并且调用handler的handlerAdded方法
        handler().handlerAdded(this);
    }
}
```

很显然这里的handler方法获得的就是ChannelInitializer对象，接着进入ChannelInitializer的handlerAdded方法：

```java
public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
    // 如果管道已经完成注册，即registered标记为true
    if (ctx.channel().isRegistered()) {
        if (initChannel(ctx)) {
		   // 做一些后续的处理
            removeState(ctx);
        }
    }
}

private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
    // 保证一个context只会被执行一次
    if (initMap.add(ctx)) { 
        try {
            // 执行由子类实现的initChannel方法
            initChannel((C) ctx.channel());
        } catch (Throwable cause) {
            // 调用Handler的异常处理方法
            exceptionCaught(ctx, cause);
        } finally {
            // 最后将ChannelInitializer从pipline中删除
            ChannelPipeline pipeline = ctx.pipeline();
            if (pipeline.context(this) != null) {
                pipeline.remove(this);
            }
        }
        return true;
    }
    return false;
}
```

最终handlerAdded会调用子类实现的initChannel方法，现在我们回过头来看被添加到pipeline的ChannelInitializer实现的initChannel方法：

```java
p.addLast(new ChannelInitializer<Channel>() {
    @Override
    public void initChannel(final Channel ch) throws Exception {
        final ChannelPipeline pipeline = ch.pipeline();
        // 在pipline中添加通过Bootstrap对象的handler配置的Handler
        ChannelHandler handler = config.handler();
        if (handler != null) {
            pipeline.addLast(handler);
        }

        // 向EventLoop线程的任务队列提交创建ServerBootstrapAcceptor对象的任务
        ch.eventLoop().execute(new Runnable() {
            @Override
            public void run() {
                pipeline.addLast(new ServerBootstrapAcceptor(
                    ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
            }
        });
    }
});
```

initChannel方法只做了两件事，一是将在Bootstrap对象的handler方法中配置的Handler加入到pipeline中，由于我们并没有配置Handler所以这一步没有添加Handler；二是将添加ServerBootstrapAcceptor对象到pipeline的工作交给Channel的EventLoop来处理。这里这么做的原因是，要保证如果config中Handler也是ChannelInitializer时，ChannelInitializer配置的Handler会在ServerBootstrapAcceptor之前，那为什么能保证呢？

我是这样认为的，之前在addLast方法中分析过，addLast执行callHandlerAdded0方法，就是最终执行ChannelInitializer的initChannel方法的可能性有三种，其中一种的条件是pipeline的register属性为false，此时register已经被设置为true所以不考虑这种可能性；但是存在当前线程与EventLoop的工作线程不是同一个线程的情况，callHandlerAdded0方法可能被直接执行，这种情况是可以保证先后顺序的，但也可能被放入EventLoop的任务队列中去执行，所以为了保证ServerBootstrapAcceptor在pipeline的末尾，需要将添加ServerBootstrapAcceptor工作加入到任务队列中去。

这里对ServerBootstrapAcceptor进行一个简单的说明，ServerBootstrapAcceptor实现了ChannelInboundHandlerAdapter接口负责监听连接请求，在完成连接以后会创建一个NioSocketChannel来表示客户端连接，NioSocketChanne会被l注册到workerGroup，用于监听IO读写请求。

在调用完子类的initChannel方法之后，在finally块中最后会从piplien中将ChannelInitializer移除，至此invokeHandlerAddedIfNeeded()方法的整个调用流程就结束了，pipeline链表的结构如下：

![ctx_link3](..\images\netty_ctx_link3.png)

- 调用safeSetSuccess方法，调用链如下：

```java
protected final void safeSetSuccess(ChannelPromise promise) {
    if (!(promise instanceof VoidChannelPromise) && !promise.trySuccess()) {
        logger.warn("Failed to mark a promise as success because it is done already: {}", promise);
    }
}

public boolean trySuccess() {
    return trySuccess(null);
}

public boolean trySuccess(V result) {
    if (setSuccess0(result)) {
        notifyListeners();
        return true;
    }
    return false;
}
```

该方法的主要逻辑就是将DefaultPromise对象的结果设置为SUCCESS并且通知所有的监听器执行operationComplete方法，这里就不深入具体的逻辑了。

- pipeline的fireChannelRegistered方法会依次调用pipleline中的双向链表中的InboundHandler的channelRegistered()方法，执行管道注册完成的逻辑，这里也不继续进行展开。
- 最后，如果Java Channel处于绑定状态，通过pipeline.fireChannelActive()方法依次调用pipleline中的双向链表中的InboundHandler的channelActive()方法，执行管道激活完成的逻辑，但是这时候管道还未绑定所以不会执行这里的逻辑。

这样register0方法的调用就结束了，然而因为register0是一个异步调用过程所以实际上调用register0的register方法早就返回了，随着register方法早就返回initAndRegister方法的调用也结束了并且返回了存放register0这个异步调用结果的DefaultPromise对象，让我们回到ServerBootstrap的doBind方法中：

```java
private ChannelFuture doBind(final SocketAddress localAddress) {
    // 创建接受连接请求的Channel并注册到NioEventLoop的Selector
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    if (regFuture.cause() != null) {
        return regFuture;
    }

    // 判断操作是否完成
    if (regFuture.isDone()) {
        ChannelPromise promise = channel.newPromise();
        // 如果操作完成调用doBind0方法
        doBind0(regFuture, channel, localAddress, promise);
        return promise;
    } else {
        final PendingRegistrationPromise promise = new PendingRegistrationPromise(channel);
        // 如果操作还未完成则添加监听器，在完成后判断操作是否失败，如果未失败则继续调用doBind0方法
        regFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                Throwable cause = future.cause();
                if (cause != null) {
                    promise.setFailure(cause);
                } else {
                    promise.registered();
                    doBind0(regFuture, channel, localAddress, promise);
                }
            }
        });
        return promise;
    }
}
```

在initAndRegister方法返回后，通过DefaultPromise的isDone方法判断一次register0方法是否完成了调用，如果完成则直接调用doBind0方法，如果没有完成添加一个监听器去等待任务完成后继续执行doBind0方法，之前分析过监听器会在safeSetSuccess中被通知。因此无论如何，最终都会调用doBind0方法：

```java
private static void doBind0(
    final ChannelFuture regFuture, final Channel channel,
    final SocketAddress localAddress, final ChannelPromise promise) {
	// 在触发channelRegistered之前调用此方法
    // 使用户处理程序有机会在其channelRegistered实现中设置管道
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (regFuture.isSuccess()) {
                channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            } else {
                promise.setFailure(regFuture.cause());
            }
        }
    });
}
```

该方法直接调用了AbstractChannel的bind方法，调用链如下：

```java
public ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    return pipeline.bind(localAddress, promise);
}

public final ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    // 调用了TailContext的bind方法
    return tail.bind(localAddress, promise);
}

public ChannelFuture bind(final SocketAddress localAddress, final ChannelPromise promise) {
    if (localAddress == null) {
        throw new NullPointerException("localAddress");
    }
    if (isNotValidPromise(promise, false)) {
        return promise;
    }

    // 找到上一个outbound
    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    // 执行AbstractChannelHandlerContext的invokeBind
    if (executor.inEventLoop()) {
        next.invokeBind(localAddress, promise);
    } else {
        safeExecute(executor, new Runnable() {
            @Override
            public void run() {
                next.invokeBind(localAddress, promise);
            }
        }, promise, null);
    }
    return promise;
}
```

通过findContextOutbound找到pipeline链表中的上一个持有实现了ChannelOutboundHandler接口的Handler对象的AbstractChannelHandlerContext对象，findContextOutbound方法如下：

```java
private AbstractChannelHandlerContext findContextOutbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.prev;
    } while (!ctx.outbound);
    return ctx;
}
```

目前的pipeline链表如下：

![ctx_link3](..\images\netty_ctx_link3.png)

所以这里找到的AbstractChannelHandlerContext为HeadContext，最终会进入TailContext的bind方法，调用链如下：

```java
public void bind(
    ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) {
    // 由于TailContext是pipeline的内部类，所以这里调用了pipeline的unsafe的bind方法
    unsafe.bind(localAddress, promise);
}

public final void bind(final SocketAddress localAddress, final ChannelPromise promise) {
    assertEventLoop();

    if (!promise.setUncancellable() || !ensureOpen(promise)) {
        return;
    }

    if (Boolean.TRUE.equals(config().getOption(ChannelOption.SO_BROADCAST)) &&
        localAddress instanceof InetSocketAddress &&
        !((InetSocketAddress) localAddress).getAddress().isAnyLocalAddress() &&
        !PlatformDependent.isWindows() && !PlatformDependent.maybeSuperUser()) {
        logger.warn(
            "A non-root user can't receive a broadcast packet if the socket " +
            "is not bound to a wildcard address; binding to a non-wildcard " +
            "address (" + localAddress + ") anyway as requested.");
    }

    boolean wasActive = isActive();
    try {
        // 绑定Java Channel到指定的端口
        doBind(localAddress);
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        closeIfClosed();
        return;
    }

    // 如果Java Channel之前是未绑定的，现在完成了绑定则执行fireChannelActive逻辑
    if (!wasActive && isActive()) {
        // 在Channel的EventLoop的任务队列将pipeline.fireChannelActive()方法作为任务提交
        invokeLater(new Runnable() {
            @Override
            public void run() {
                pipeline.fireChannelActive();
            }
        });
    }

    safeSetSuccess(promise);
}
```

这里有很重要的两步逻辑，一是通过doBind方法将Java Channel绑定到对应的端口；然后在绑定之后接着调用fireChannelActive方法，开始管道激活的逻辑。

由于这里的Channel类是NioServerSocketChannel，所以进入NioServerSocketChannel的doBind方法：

```java
protected void doBind(SocketAddress localAddress) throws Exception {
    if (PlatformDependent.javaVersion() >= 7) {
        javaChannel().bind(localAddress, config.getBacklog());
    } else {
        javaChannel().socket().bind(localAddress, config.getBacklog());
    }
}
```

这里就回到了Java NIO的API，根据JDK版本实现Java Channel的绑定：

- `>=`7，调用ServerSocketChannel.bind(SocketAddress，config.getBacklog())
- 否则，调用ServerSocketChannel.socket().bind(SocketAddress，config.getBacklog())

完成绑定后，稍后调用`pipeline.fireChannelActive()`方法触发管道激活事件：

```java
public final ChannelPipeline fireChannelActive() {
    // 以HeadContext作为链表的开端触发pipeline的激活方法
    AbstractChannelHandlerContext.invokeChannelActive(head);
    return this;
}

static void invokeChannelActive(final AbstractChannelHandlerContext next) {
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelActive();
    } else {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                next.invokeChannelActive();
            }
        });
    }
}
```

fireChannelActive方法将HeadContext作为参数调用了invokeChannelActive方法，因此最终将调用HeadContext的invokeChannelActive方法：

```java
private void invokeChannelActive() {
    if (invokeHandler()) {
        try {
            // HeadContext的handler方法返回的HeadContext自身
            // 所以将继续调用HeadContext的channelActive方法
            ((ChannelInboundHandler) handler()).channelActive(this);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelActive();
    }
}
```

由于HeadContext的handler方法返回的HeadContext自身，所以将继续调用HeadContext的channelActive方法：

```java
public void channelActive(ChannelHandlerContext ctx) {
    // 将激活事件传递给链表中的其他HandlerContext
    ctx.fireChannelActive();

    readIfIsAutoRead();
}
```

HeadContext会首先将激活事件传递给链表中的其他HandlerContext，由于其他Context中没有特别重要的激活事件的逻辑，这里就不进行深入了。接下来继续执行readIfIsAutoRead方法，逻辑如下：

```java
private void readIfIsAutoRead() {
    if (channel.config().isAutoRead()) {
        channel.read();
    }
}

public Channel read() {
    pipeline.read();
    return this;
}

public final ChannelPipeline read() {
    tail.read();
    return this;
}

public ChannelHandlerContext read() {
    final AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeRead();
    } else {
        Runnable task = next.invokeReadTask;
        if (task == null) {
            next.invokeReadTask = task = new Runnable() {
                @Override
                public void run() {
                    next.invokeRead();
                }
            };
        }
        executor.execute(task);
    }

    return this;
}
```

可以看出，read方法的调用链和bind方法的调用链大同小异，也是通过TailContext往前寻找持有实现了ChannelOutboundHandler接口的Handler对象的AbstractChannelHandlerContext对象，在当前链表中只能找到HeadContext，因此接下来将执行HeadContext的read方法，如下：

```java
public void read(ChannelHandlerContext ctx) {
    unsafe.beginRead();
}

// AbstractUnsafe
public final void beginRead() {
    assertEventLoop();

    if (!isActive()) {
        return;
    }

    try {
        doBeginRead();
    } catch (final Exception e) {
        invokeLater(new Runnable() {
            @Override
            public void run() {
                pipeline.fireExceptionCaught(e);
            }
        });
        close(voidPromise());
    }
}

// AbstractNioMessageChannel
protected void doBeginRead() throws Exception {
    if (inputShutdown) {
        return;
    }
    super.doBeginRead();
}

// AbstractNioChannel
protected void doBeginRead() throws Exception {
    // 这里的selectionKey就是之前初始化NioServerSocketChannel时设置的SelectionKey.OP_ACCEPT
    final SelectionKey selectionKey = this.selectionKey;
    if (!selectionKey.isValid()) {
        return;
    }

    readPending = true;

    final int interestOps = selectionKey.interestOps();
    if ((interestOps & readInterestOp) == 0) {
        selectionKey.interestOps(interestOps | readInterestOp);
    }
}
```

根据调用调用链可知最后将调用AbstractNioChannel的doBeginRead方法，其中的readInterestOp就是之前的SelectionKey.OP_ACCEPT的值，又回到了Java NIO的API，通过selectionKey .interestOps()注册一个ACCEPT事件。到这里整个`ServerBootstrap.bind`的逻辑就完成了，接受连接请求的管道也初始化和创建了。

## IO读写管道的创建和数据的出站入站

要开始做交接工作了，后续有时间在把这边的分析补上。

<====To be continue...

## Ref

https://blog.csdn.net/u013857458/article/details/82527722

https://blog.csdn.net/dk243553650/article/details/83062573

