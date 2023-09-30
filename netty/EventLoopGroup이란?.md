- netty에서는 EventLoopGroup 이라는 인터페이스(`io.netty.channel.``EventLoopGroup`)가 있으며 EventLoopGroup은 여러개의 EventLoop를 갖고 풀방식을 취한다.
- `EventLoopGroup` 는 하나의 Runnable task를 수행하면 EventLoop에 할당한 뒤 수행을 맡기게 된다.
    - 새로운 쓰레드에 위임하는 방식과 비슷한 원리이다.

![image](https://github.com/eastperson/TIL/assets/66561524/31a0f54e-8a83-4d32-89b4-fb0842f01c55)

NioEventLoopGroup은 NioServerSocketChannel을 갖고 있으며 맨 앞단에서 Selector와 비슷한 역할을 한다.

![image](https://github.com/eastperson/TIL/assets/66561524/4307cf2f-7cbb-4132-843f-c110dd98994f)

- 내부적으로 EventLooprk Selector를 사용한다.
- NioServerSocketChannel은 java의 ServerSocketChannel과 유사하며 비동기적으로 처리하는 netty에서 제공하는 클래스이다.

```kotlin
fun main() {
    val eventLoopGroup = NioEventLoopGroup()
    val nioServerSocketChannel = NioServerSocketChannel()
    eventLoopGroup.register(nioServerSocketChannel)

    val bind = nioServerSocketChannel.bind(InetSocketAddress(8080))
    bind.addListener { future -> println("bind completed : $future") }
}

..
public interface ChannelFuture extends Future<Void> {

	...

	ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> listener);
..
```

- ChannelFuture은 비동기적인 결과를 받는 Java의 Future와 유사한 기능을 한다.

### Pipeline

channel에는 pipeLine을 추가할 수 있다.

- channel에 어떤 데이터가 왔을 때의 수행을 명시할 수 있다.
- pipeline에는 여러개의 핸들러를 추가할 수 있으며 순차적으로 진행된다.
    - ChannelInboundHandler, ChannelOutBoundHandler

![image](https://github.com/eastperson/TIL/assets/66561524/c2e3fb12-73c8-46f0-ac82-31859003aebb)

```kotlin
fun main() {
    val eventLoopGroup = NioEventLoopGroup()
    val nioServerSocketChannel = NioServerSocketChannel()
    eventLoopGroup.register(nioServerSocketChannel)

    val bind = nioServerSocketChannel.bind(InetSocketAddress(8080))
    bind.addListener {
        ChannelFutureListener {
            println(it.channel())
            it.channel().pipeline()
                .addLast(object : ChannelInboundHandlerAdapter() {
                    override fun channelRead(ctx: ChannelHandlerContext, msg: Any) {
                        val channel = msg as Channel
                        println("channel : $channel")
                        super.channelRead(ctx, msg)
                    }
                })
        }
    }
}
```

- 채널을 만들고 파이프 라인에 행동을 정의합니다.
- 앞서봤던 ChannelInboundHandlerAdapter를 익명 클래스로 구현했습니다.

### BootStrap
![image](https://github.com/eastperson/TIL/assets/66561524/a015ba0a-bb8f-49da-ad7c-81610080497e)
위와 같은 로직으로 돌아가고 있다.

하지만 ServerBootStrap을 사용하면 EventLoopGroup 2개를 사용해야 한다.

- EventLoopGroup1
    - NioServerSocketChannel이 갖고 있는 eventLoopGroup을 가진다.
    - 요청이 오면 eventLoop를 한개 꺼내온 뒤 NioSocketChannel을 생성하는 역할을 한다.
- EventLoopGroup2
    - 요청이 온 뒤 (channelRead 이벤트) 에 생성된 NioSocketChannel 을 등록할 수 있는 eventLoopGroup를 갖고 있다.
    - 이벤트에 대해서는 등록한 pipeLine에 따른 순서로 처리한다.
![image](https://github.com/eastperson/TIL/assets/66561524/88992b4f-8ce1-42c5-b5a2-3c8e400b8b79)
- 첫번째 EventLoopGroup 은 자식(NioSocketChannel) 생성을 담당
- 두번째 EventLoopGroup 은 생성된 자식으로 이벤트처리를 담당

```kotlin
fun main() {
    val serverBootstrap = ServerBootstrap()

    val parent = NioEventLoopGroup()
    val child = NioEventLoopGroup()
    parent.execute { println("hello parent") }
    child.execute { println("hello child") }

    serverBootstrap.group(parent, child)
        .channel(NioServerSocketChannel::class.java)
        .localAddress(8080) // 포트 바인딩
        .childHandler(object : SimpleChannelInboundHandler<ByteBuf>() { // child 핸들러 추가
            override fun channelRead0(ctx: ChannelHandlerContext?, msg: ByteBuf?) {
								// child가 처리할 파이프라인의 핸들러 메소드들
                println("msg : ${msg?.toString(CharsetUtil.UTF_8)}")
            }
        })

    val channelFuture = serverBootstrap.bind()
}
```

![Sep-30-2023 11-53-37](https://github.com/eastperson/TIL/assets/66561524/d136a4f4-a1e8-4b46-b741-7bab9d022e2a)
