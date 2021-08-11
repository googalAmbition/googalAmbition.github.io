# rpc

## BOI
1. server
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
public class IOServer {

    public static void main(String[] args) throws Exception {

        //首先创建了一个serverSocket
        ServerSocket serverSocket = new ServerSocket();
        serverSocket.bind(new InetSocketAddress("127.0.0.1", 8081));
        while (true) {
            //同步阻塞
            Socket socket = serverSocket.accept();
            new Thread(() -> {
                try {
                    byte[] bytes = new byte[1024];
                    int len = socket.getInputStream().read(bytes);  //同步阻塞
                    System.out.println(new String(bytes, 0, len));
                    socket.getOutputStream().write(bytes, 0, len);
                    socket.getOutputStream().flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

2. client

```java
import java.io.IOException;
import java.net.Socket;
public class IOClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8081);
        socket.getOutputStream().write("hello".getBytes());
        socket.getOutputStream().flush();
        System.out.println("server send back data =====");
        byte[] bytes = new byte[1024];
        int len = socket.getInputStream().read(bytes);
        System.out.println(new String(bytes, 0, len));
        socket.close();
    }
}
```
## NIO
1. server
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.CancelledKeyException;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;
public class NIOServer extends Thread {

    //1.声明多路复用器
    private Selector selector;

    //2.定义读写缓冲区
    private ByteBuffer readBuffer = ByteBuffer.allocate(1024);
    private ByteBuffer writeBuffer = ByteBuffer.allocate(1024);

    //3.定义构造方法初始化端口
    public NIOServer(int port) {
        init(port);
    }

    //4.main方法启动线程
    public static void main(String[] args) {
        new Thread(new NIOServer(8888)).start();
    }

    //5.初始化
    private void init(int port) {

        try {
            System.out.println("服务器正在启动......");
            //1)开启多路复用器
            this.selector = Selector.open();
            //2) 开启服务通道
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            //3)设置为非阻塞
            serverSocketChannel.configureBlocking(false);
            //4)绑定端口
            serverSocketChannel.bind(new InetSocketAddress(port));
            /**
             * SelectionKey.OP_ACCEPT   —— 接收连接继续事件，表示服务器监听到了客户连接，服务器可以接收这个连接了
             * SelectionKey.OP_CONNECT  —— 连接就绪事件，表示客户与服务器的连接已经建立成功
             * SelectionKey.OP_READ     —— 读就绪事件，表示通道中已经有了可读的数据，可以执行读操作了（通道目前有数据，可以进行读操作了）
             * SelectionKey.OP_WRITE    —— 写就绪事件，表示已经可以向通道写数据了（通道目前可以用于写操作）
             */
            //5)注册,标记服务连接状态为ACCEPT状态
            serverSocketChannel.register(this.selector, SelectionKey.OP_ACCEPT);
            System.out.println("服务器启动完毕");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        while (true) {
            try {
                //1.当有至少一个通道被选中,执行此方法
                this.selector.select();
                //2.获取选中的通道编号集合
                Iterator<SelectionKey> keys = this.selector.selectedKeys().iterator();
                //3.遍历keys
                while (keys.hasNext()) {
                    SelectionKey key = keys.next();
                    //4.当前key需要从动刀集合中移出,如果不移出,下次循环会执行对应的逻辑,造成业务错乱
                    keys.remove();
                    //5.判断通道是否有效
                    if (key.isValid()) {
                        try {
                            //6.判断是否可以连接
                            if (key.isAcceptable()) {
                                accept(key);
                            }
                        } catch (CancelledKeyException e) {
                            //出现异常断开连接
                            key.cancel();
                        }

                        try {
                            //7.判断是否可读
                            if (key.isReadable()) {
                                read(key);
                            }
                        } catch (CancelledKeyException e) {
                            //出现异常断开连接
                            key.cancel();
                        }

                        try {
                            //8.判断是否可写
                            if (key.isWritable()) {
                                write(key);
                            }
                        } catch (CancelledKeyException e) {
                            //出现异常断开连接
                            key.cancel();
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void accept(SelectionKey key) {
        try {
            //1.当前通道在init方法中注册到了selector中的ServerSocketChannel
            ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
            //2.阻塞方法, 客户端发起后请求返回.
            SocketChannel channel = serverSocketChannel.accept();
            ///3.serverSocketChannel设置为非阻塞
            channel.configureBlocking(false);
            //4.设置对应客户端的通道标记,设置次通道为可读时使用
            channel.register(this.selector, SelectionKey.OP_READ);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //使用通道读取数据
    private void read(SelectionKey key) {
        try {
            //清空缓存
            this.readBuffer.clear();
            //获取当前通道对象
            SocketChannel channel = (SocketChannel) key.channel();
            //将通道的数据(客户发送的data)读到缓存中.
            int readLen = channel.read(readBuffer);
            //如果通道中没有数据
            if (readLen == -1) {
                //关闭通道
                key.channel().close();
                //关闭连接
                key.cancel();
                return;
            }
            //Buffer中有游标,游标不会重置,需要我们调用flip重置. 否则读取不一致
            this.readBuffer.flip();
            //创建有效字节长度数组
            byte[] bytes = new byte[readBuffer.remaining()];
            //读取buffer中数据保存在字节数组
            readBuffer.get(bytes);
            System.out.println("收到了从客户端 " + channel.getRemoteAddress() + " :  " + new String(bytes, "UTF-8"));
            //注册通道,标记为写操作
            channel.register(this.selector, SelectionKey.OP_WRITE);
        } catch (Exception e) {

        }
    }

    //给通道中写操作
    private void write(SelectionKey key) {
        //清空缓存
        this.readBuffer.clear();
        //获取当前通道对象
        SocketChannel channel = (SocketChannel) key.channel();
        //录入数据
        Scanner scanner = new Scanner(System.in);

        try {
            System.out.println("即将发送数据到客户端..");
            String line = scanner.nextLine();
            //把录入的数据写到Buffer中
            writeBuffer.put(line.getBytes("UTF-8"));
            //重置缓存游标
            writeBuffer.flip();
            channel.write(writeBuffer);
            channel.register(this.selector, SelectionKey.OP_READ);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

2. client

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.util.Scanner;
public class NIOClient {

    public static void main(String[] args) {
        //创建远程地址
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        SocketChannel channel = null;
        //定义缓存
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        try {
            //开启通道
            channel = SocketChannel.open();
            //连接远程远程服务器
            channel.connect(address);
            Scanner sc = new Scanner(System.in);
            while (true) {
                System.out.println("客户端即将给 服务器发送数据..");
                String line = sc.nextLine();
                if (line.equals("exit")) {
                    break;
                }
                //控制台输入数据写到缓存
                buffer.put(line.getBytes("UTF-8"));
                //重置buffer 游标
                buffer.flip();
                //数据发送到数据
                channel.write(buffer);
                //清空缓存数据
                buffer.clear();

                //读取服务器返回的数据
                int readLen = channel.read(buffer);
                if (readLen == -1) {
                    break;
                }
                //重置buffer游标
                buffer.flip();
                byte[] bytes = new byte[buffer.remaining()];
                //读取数据到字节数组
                buffer.get(bytes);
                System.out.println("收到了服务器发送的数据 : " + new String(bytes, "UTF-8"));
                buffer.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != channel) {
                try {
                    channel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
## RMI
1. interface
```java
import java.rmi.Remote;
import java.rmi.RemoteException;
public interface IHelloService extends Remote {

    //1.定义一个sayHello方法
    String sayHello(User user) throws RemoteException;
}
```

2. server


impl

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
public class HelloServiceImpl extends UnicastRemoteObject implements IHelloService {

    //手动实现父类的构造方法
    public HelloServiceImpl() throws RemoteException {
        super();
    }

    //我们自定义的sayHello
    public String sayHello(User user) throws RemoteException {
        System.out.println("this is server , say hello to " + user.getUsername());
        return "success";
    }
}
```

```java
import java.net.MalformedURLException;
import java.rmi.AlreadyBoundException;
import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
public class RMIServer {

    public static void main(String[] args) throws RemoteException, AlreadyBoundException, MalformedURLException {
        //1.创建HelloService实例
        IHelloService service = new HelloServiceImpl();

        //2.获取注册表
        LocateRegistry.createRegistry(8888);

        //3.对象的绑定
        //bind方法的参数1:   rmi://ip地址:端口/服务名   参数2:绑定的对象
        Naming.bind("//127.0.0.1:8888/rmiserver", service);
    }
}
```
3. client
```java
import java.net.MalformedURLException;
import java.rmi.Naming;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;
public class RMIClient {

    public static void main(String[] args) throws RemoteException, NotBoundException, MalformedURLException {
        //1.从注册表中获取远程对象 , 强转
        IHelloService service = (IHelloService) Naming.lookup("//127.0.0.1:8888/rmiserver");

        //2.准备参数
        User user = new User("laowang", 18);

        //3.调用远程方法sayHello
        String message = service.sayHello(user);
        System.out.println(message);
    }
}
```
## Netty
1. server
```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
// 接收客户端请求,打印在控制台
public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
        //1.创建2个线程池对象
        //bossGroup 负责接收用户连接
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        //workGroup 负责处理用户的io读写操作
        NioEventLoopGroup workGroup = new NioEventLoopGroup();

        //2.创建启动引导类
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        //3.设置启动引导类
        //添加到组中,两个线程池,第一个位置的线程池就负责接收,第二个参数就负责读写
        serverBootstrap.group(bossGroup, workGroup)
            //给我们当前设置一个通道类型
            .channel(NioServerSocketChannel.class)
            //绑定一个初始化监听
            .childHandler(new ChannelInitializer<NioSocketChannel>() {
                //事件监听Channel通道
                @Override
                protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                    //获取pipeLine
                    ChannelPipeline pipeline = nioSocketChannel.pipeline();
                    //绑定编码
                    pipeline.addFirst(new StringEncoder());
                    pipeline.addLast(new StringDecoder());
                    //绑定我们的业务逻辑
                    pipeline.addLast(new SimpleChannelInboundHandler<String>() {
                        @Override
                        protected void channelRead0(ChannelHandlerContext channelHandlerContext, String msg) throws Exception {
                            //获取入栈信息,打印客户端传递的数据
                            System.out.println(msg);
                        }
                    });
                }
            });

        //4.启动引导类绑定端口
        ChannelFuture future = serverBootstrap.bind(9999).sync();

        //5.关闭通道
        future.channel().closeFuture().sync();
    }
}
```
2. client
```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
//客户端给服务器发送数据
public class NettyClient {

    public static void main(String[] args) throws InterruptedException {
        //1.创建连接池对象
        NioEventLoopGroup group = new NioEventLoopGroup();

        //2.创建客户端的启动引导类 BootStrap
        Bootstrap bootstrap = new Bootstrap();

        //3.配置启动引导类
        bootstrap.group(group)
            //设置通道为Nio
            .channel(NioSocketChannel.class)
            //设置Channel初始化监听
            .handler(new ChannelInitializer<Channel>() {
                //当前该方法监听channel是否初始化
                protected void initChannel(Channel channel) throws Exception {
                    //设置编码
                    channel.pipeline().addLast(new StringEncoder());
                }
            });

        //4.使用启动引导类连接服务器 , 获取一个channel
        Channel channel = bootstrap.connect("127.0.0.1", 9999).channel();

        //5.循环写数据给服务器
        while (true) {
            //给服务器写数据
            channel.writeAndFlush("hello server .. this is client ...");
            Thread.sleep(2000);
        }
    }
}
```

## [代码地址](https://github.com/googalAmbition/rpc-demo)