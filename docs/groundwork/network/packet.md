# 5.1.1 创建数据包

首先，我们先把该删的都删掉，比如我们之前监听的左键的事件，还有方块的`use`方法。

然后我们开始创建数据包。

## 新建数据包

和以往不同，我们的数据包不需要继承任何类，里面可以添加任何我们想要的东西。

但是请注意，它是网络传输用的包，所以它所存储的内容不宜过大。

我们来想想我们最少需要传递哪些数据给服务器。

已知，玩家作为发送方，我们可以通过某种方法获取玩家信息（在后续实现中会看到）。

那么我们需要告诉服务器的就是“我们右键方块的哪里”、”我们用什么右键的“和”我们右键的方块状态“。

真的吗？能不能再少点？

我们是不是可以只发送这些东西：“我们右键方块的坐标”、“我们想要将水箱的水位改变到多少”和“我们的水壶在哪只手”三项？

当然，你如果想要发送之前的那三项也可以，但是一来对于一个网络包来说太大了，二来Forge也没有方便的处理方案。

那我们先把定义做出来吧。

```java
public class CisternUseLocPack {
    private final BlockPos pos;
    private final int waterLevel;
    private final InteractionHand hand;
}
```

然后我们定义一个构造函数，让它可以被我们手动创建。

```java
public CisternUseLocPack(BlockPos pos, int level, InteractionHand hand) {
    this.pos = pos;
    this.waterLevel = level;
    this.hand = hand;
}
```

## 序列化和反序列化

根据我们之前的模型，我们还需要定义序列化成字节流和反序列化的方案，很幸运的是，Forge已经为我们封装好了对应的方法，我们只需要调用即可。

```java
public CisternUseLocPack(FriendlyByteBuf buf) {
    // 反序列化
    pos = buf.readBlockPos();
    waterLevel = buf.readInt();
    hand = buf.readEnum(InteractionHand.class);
}

public void toBytes(FriendlyByteBuf buf) {
    // 序列化
    buf.writeBlockPos(pos);
    buf.writeInt(waterLevel);
    buf.writeEnum(hand);
}
```

## 接收到包后的处理

最后和数据包相关的就是接收到包后的处理方法了，逻辑也已经讲过好多了，我在想，这次我只提供框架，由读者自己来实现相关的内容。

```java
@SuppressWarnings("resource")
public boolean handle(Supplier<NetworkEvent.Context> supplier) {
    NetworkEvent.Context context = supplier.get();
    ServerPlayer player = context.getSender();
    if (player != null) {
        BlockState state = player.level().getBlockState(pos);
        if (state.getBlock() instanceof CisternBlock) {
            adjustWaterLevel(player, state, waterLevel, hand); // 实现这个方法
        }
    }
    return true;
}
```

如果你真的不会了，就去看看[Github](https://github.com/MineCraftGensoukyo/Thirst/tree/chapter5)上的实现吧。