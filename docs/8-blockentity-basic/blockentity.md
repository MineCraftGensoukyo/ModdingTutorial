# 8.3 方块实体的基础

终于要到这一激动人心的部分了，很多复杂的逻辑都是建立在方块实体上的，这一个方块实体实际上是逻辑控制的中枢，下面我们来一步步介绍一个看起来呆呆的方块是如何成为真正的核心。

## 数据持久化
`
方块实体的数据持久化是通过覆写`load`和`saveAdditional`方法，通过他们，可以把方块实体上的各种字段保存和加载

load：当方块实体从 NBT 数据加载时，load 方法会被调用。在这个方法中，我们从 NBT 数据中读取键为 “new_int” 的整数值，并将其赋值给实体的 yourInt 变量。`super.load(pTag)` 调用父类的 `load`方法，允许父类处理其他可能存在的数据。

saveAdditional：当方块实体被保存到 NBT 数据时，`saveAdditional` 方法会被调用。在这个方法中，我们将实体的 yourInt 变量的值写入 NBT 数据，以键 “counter” 存储。`super.saveAdditional(pTag)` 调用父类的 `saveAdditional` 方法，允许父类保存其他可能存在的数据。

和网络包类似，我们可以需要准备数据的储存和加载

``` java
    public void load(CompoundTag pTag) {
        super.load(pTag);
        this.yourInt = getPersistentData().getInt("new_int");
        this.playerUUID = getPersistentData().getUUID("player_uuid");
    }

    protected void saveAdditional(CompoundTag pTag) {
        getPersistentData().putInt("new_int")
        getPersistentData().putUUID("player_uuid", this.playerUUID);
        super.saveAdditional(pTag);
    }
```

像这样，就能把基础数据持久化保存

## 数据同步

这里要注意一下，mc并不会自动帮你把数据在客户端与服务端间同步，你得告诉mc，我这里有数据，需要交给客户端/服务端。这里有两种方式可以用来同步数据

- `Synchronizing on Block Update` 方块更新

- `Synchronizing Using a Custom Network Message` 自定义网络包发包

### 方块更新

为此，你需要覆写

``` java
    @Override
    public @NotNull CompoundTag getUpdateTag() {
        return this.saveWithoutMetadata();
    }

    @Nullable
    @Override
    public Packet<ClientGamePacketListener> getUpdatePacket() {
        return ClientboundBlockEntityDataPacket.create(this);
    }
```

第一种方法收集应发送到客户端的数据，而第二种方法处理该数据。如果你的 `BlockEntity` 不包含太多数据，你也许可以使用这个方法。

在服务端时，可以调用这个方法触发方块更新，告诉mc，这个数据需要同步到客户端
``` java
    pLevel.sendBlockUpdated(pBlockEntity.getBlockPos(), pBlockEntity.getBlockState(), pBlockEntity.getBlockState(), Block.UPDATE_CLIENTS);
```

### 网络包同步

网络包的编写就比较自由，可以把需要的数据从客户端发往服务端或者反向，只需要在特定的端，把发过来的数据写入即可。

这种同步方式可能是最复杂的，但通常是最优化的，因为您可以确保实际上只有需要同步的数据是同步的。

当然，发送网络包时，也需要小心，进行安全检查很重要，当消息到达玩家时，BlockEntity 可能已经被销毁/替换了！您还应该检查块是否已加载 （Level#hasChunkAt（BlockPos））。