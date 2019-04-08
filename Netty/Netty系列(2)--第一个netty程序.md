# Netty系列(2)--第一个netty程序
---
# 简介
在了解了Netty基本概念和框架的基础上，参考《Netty实战》，编写了第一个程序--Echo客户端和服务器。

那么，什么是Echo呢？Echo就是经常说的**回显**程序，即服务器收到一个客户端的请求，就会对客户端进行响应。更简单的Echo例子是，客户端发送给服务器一个字符串，服务器将这个字符串原封不动地再发送给客户端。

下图显示了这个回显程序的一个架子。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/echonetty.PNG">
</center>

# 实现
首先启动服务器，这时服务器就处在监听状态。当客户端连接成功后，客户端程序会调用`channelActive`方法，在这个方法中，将`Netty rocks`这个字符串写入缓冲区并刷出去。

服务端在接受到消息之后，会调用`channelRead`方法，将`Netty rocks`保存到一个缓冲区数组中，并打印`Server received: Netty rocks`，然后再将这个字符串返还给客户端。

客户端接受到服务反馈回来的字符串后，会打印`Client received: Netty rocks`消息。

## 客户端程序的实现
```java
//EchoClientHandler
public class EchoClientHander extends SimpleChannelInboundHandler<ByteBuf> {
    //在连接建立的时候被调用，将信息写入缓冲区
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks", CharsetUtil.UTF_8));
    }
    //每次接收数据是都会被调用
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) {
        System.out.println("Client received: "+byteBuf.toString(CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

```java
//EchoClient
public class EchoClient {
    private final String host;
    private final int port;

    EchoClient(String host, int port){
        this.port = port;
        this.host = host;
    }

    public void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)
                    .remoteAddress(new InetSocketAddress(host, port))
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) {
                            socketChannel.pipeline().addLast(new EchoClientHander());
                        }
                    });
            ChannelFuture f = b.connect().sync();
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }

    public static void main(String[] args) throws Exception {
        String host;
        int port;
        if(args.length == 2){
            host = args[0];
            port = Integer.parseInt(args[1]);
        }else {
            host = "127.0.0.1";
            port = 8080;
        }
        new EchoClient(host, port).start();
    }
}
```

## 服务器程序的实现
```java
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    //每个传入的消息都要调用这个方法
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf)msg;         //将接收到的消息存入ByteBuf中
        System.out.println("Server received: "+in.toString(CharsetUtil.UTF_8));
        ctx.write(in);
    }

    //通知ChannelInbountHandler消息已经读完
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }

    //异常处理
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();    //打印异常日志
        ctx.close();                //关闭channel
    }
}
```

```java
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        int port;
        //如果调用函数的时候传进来端口值，就使用端口值，否则使用8080
        if(args.length > 0){
            port = Integer.parseInt(args[0]);
        }else{
            port = 8080;
        }
        new EchoServer(port).start();
    }

    public void start() throws Exception {
        final EchoServerHandler echoServerHandler = new EchoServerHandler();
        //NioEventLoopGroup 是用来处理I/O操作的多线程事件循环器，
        //Netty提供了许多不同的EventLoopGroup的实现用来处理不同传输协议。
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            //ServerBootstrap 是一个启动NIO服务的辅助启动类 可以在这个服务中直接使用Channel
            ServerBootstrap b = new ServerBootstrap();
            //ServerSocketChannel以NIO的selector为基础进行实现的，用来接收新的连接
            //ChannelInitializer是一个特殊的处理类，他的目的是帮助使用者配置一个新的Channel。
            //并将新的channel实例添加到ChannelPipeline中
            b.group(group).
                    channel(NioServerSocketChannel.class).
                    localAddress(new InetSocketAddress(port)).
                    childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) {
                            socketChannel.pipeline().addLast(echoServerHandler);
                        }
                    });

            //绑定服务器，阻塞直到绑定完成。
            ChannelFuture f = b.bind().sync();
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }
}
```

## 程序运行结果
客户端结果：
```java
Client received: Netty rocks
```

服务器结果：
```java
Server received: Netty rocks
```

---
参考：

Netty实战

---





