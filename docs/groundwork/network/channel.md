# 5.1.2 Channel

我们可以将Channel比作广播频道，这样会更易于理解。

定义Channel本来是一件很麻烦的事，不过好消息是，Forge为我们提供了一个很方便的实现——`SimpleChannel`

```java
public class Channel {
    public static SimpleChannel INSTANCE;
}
```

和以前不一样的是，接下来的内容我将会先提供代码再解释代码，不会解释它为什么会是这样，不然难度太过于大了。如果感兴趣的话可以去学习计算机网络相关的内容或者学习Netty。

```java
public class Channel {
    public static SimpleChannel INSTANCE;
    public static String VERSION = "1.0";

    private static int id = 0;

    public static int nextId() {
        return id++;
    }

    public static void register() {
        SimpleChannel net = NetworkRegistry.ChannelBuilder
                .named(new ResourceLocation(MODID, "water_channel"))
                .networkProtocolVersion(() -> VERSION)
                .clientAcceptedVersions(VERSION::equals)
                .serverAcceptedVersions(VERSION::equals)
                .simpleChannel();

        INSTANCE = net;

        net.messageBuilder(CisternUseLocPack.class, nextId())
                .encoder(CisternUseLocPack::toBytes)
                .decoder(CisternUseLocPack::new)
                .consumerMainThread(CisternUseLocPack::handle)
                .add();
    }

    public static <MSG> void sendToServer(MSG message) {
        INSTANCE.sendToServer(message);
    }

    public static <MSG> void sendToPlayer(MSG message, ServerPlayer player) {
        INSTANCE.send(PacketDistributor.PLAYER.with(() -> player), message);
    }
}
```

首先是ID，在同一个Channel中，我们需要分清我们接收到的是哪个包，那么就需要一个唯一标识符进行标记，这个标识符就是ID。我们使用一个静态变量，同时在创建一个包的时候就使它的值加一保证每个包的ID都不同。

VERSION顾名思义，是版本号的意思。在Channel中，我们需要给包一个版本号，然后在接受端会根据这个版本号来确定包是否可以被接收。

我们稍后会将register注册到Mod事件总线中，我们先来解释该方法的定义是什么意思。

- `SimpleChannel net`及其定义：
  - `named`是这个Channel的唯一标识符，在一个模组中，往往会有多个Channel来负责网络的传输。
  - 后三行表示给这个Channel提供怎样的版本号以及在客户端和服务端，这个Channel分别允许接受什么版本号的包。
  - 最后是构建这个Channel。
- `INSTANCE = net`将类中的成员绑定到该实例中
- 最后五行分别定义了
  - 数据包的类和ID
  - 序列化方法
  - 反序列化方法
  - 收到包后的处理方法
  - 将数据包添加到Channel中

另外最后的`sendToServer`和`sendToPlayer`是为了更方便我们使用Channel，你也可以选择直接在你要使用Channel的地方使用这段代码。

## 使用Channel

我们只需要在我们想要发送包的地方使用Channel即可，然后在接收到包的时候，Channel会自动调用我们的`handle`方法。

之前说的我将会生造一个客户端判定就是在这里，硬生生要求这个操作只能在客户端执行，那么服务端想要同步就必须使用网络发包。

```java
else if (pLevel.getBlockState(pos).getBlock() instanceof CisternBlock && pLevel.isClientSide) {
    // 如果右键到水箱的话，根据右键的位置发送网络包
    Vec3 hitVec = blockhitresult.getLocation().subtract(pos.getX(), pos.getY(), pos.getZ());
    // 0~0.125是第一个水位，0.125~0.375是第二个水位，0.375~0.625是第三个水位，0.625~0.875是第四个水位，0.875~1是第五个水位
    int targetWaterLevel = (int)((hitVec.y + 0.125) * 4);
    Channel.sendToServer(new CisternUseLocPack(pos, targetWaterLevel, pUsedHand));
}
```

当然，你也可以把这段代码做成一段函数，毕竟它是要在两个物品类里面都要添加的代码。

哦对了，如果你用的是`use`方法的话，你是不需要做的这么麻烦的，if的条件直接写

```java
else if (blockstate.getBlock() instanceof CisternBlock && pLevel.isClientSide)
```

至此，网络发包的部分结束。你会发现我讲的远没有之前详细了，两个原因，一是因为代码编辑的部分我认为在前面四章的实践下已经不需要讲那么细致了，二是如果讲太细致了反而容易看不懂，因为会涉及到网络相关的知识。也是因为这两个理由，后续除非遇到真的很难理解的点，否则我也不会讲得非常细致，还请读者理解。