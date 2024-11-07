# 10.1 自己的能力

首先，我们要有自己的Capability，用来储存数据和对应的序列化，反序列化

``` java
public class LevelCapability {
    private float level;
    private boolean isChanged;
    public float getLevel() {
        return this.level;
    }

    public void setLevel(float level) {
        this.level = level;
    }

    public void markChanged() {
        this.isChanged = true;
    }

    public void copyFrom(LevelCapability source){
        this.level = source.getLevel();
    }

    public void saveNBTData(CompoundTag nbt){
        nbt.putFloat("level",this.level);
    }
    public void loadNBTData(CompoundTag nbt){
        this.level = nbt.getFloat("level");
    }
}

```

可以看到这里只保存了一个int类型的值level和对应的一些操作方法，这个int，就是我们预备给玩家附加的自己的数据。

接下来就是对应的`CapabilityProvider`

``` java
public class LevelProvider implements ICapabilityProvider, INBTSerializable<CompoundTag> {
    public static Capability<LevelCapability> LEVEL = CapabilityManager.get(new CapabilityToken<>() {});
    private LevelCapability levelCapability = null;
    @Override
    public @NotNull <T> LazyOptional<T> getCapability(@NotNull Capability<T> capability, @Nullable Direction direction) {
        if (capability == LEVEL) {
            return LazyOptional.of(this::createCapability).cast();
        }
        return LazyOptional.empty();
    }

    @Nonnull
    private LevelCapability createCapability() {
        if (levelCapability == null) {
            this.levelCapability = new LevelCapability();
        }
        return levelCapability;
    }

    @Override
    public void deserializeNBT(CompoundTag nbt) {
        createCapability().loadNBTData(nbt);
    }

    @Override
    public CompoundTag serializeNBT() {
        CompoundTag nbt = new CompoundTag();
        createCapability().saveNBTData(nbt);
        return nbt;
    }
}
```

可以看到，这里我们实现了两个接口`ICapabilityProvider`和`INBTSerializable<CompoundTag>`，`ICapabilityProvider`是实现能力必须实现的接口，`INBTSerializable<CompoundTag>`接口是Forge用来读取和保存数据的接口，来把我们的数据持久化。

`getOrCreateCapability`内容很简单，就是如果没有创建能力就创建一个新的能力，如果有就返回一个旧的能力。

现在，我们的能力已经准备好了，下一步就是向玩家实体附加能力，这里我们需要监听`AttachCapabilitiesEvent<Entity>`事件

``` java
    @SubscribeEvent
    public static void onAttachment(AttachCapabilitiesEvent<Entity> event) {
        if (event.getObject() instanceof Player player) {
            if (!player.getCapability(LevelProvider.LEVEL).isPresent()) {
                event.addCapability(new ResourceLocation(ExampleMod.MODID, "level"), new LevelProvider());
            }
        }
    }

```

在这里，我们先判断目标实体是否为玩家，再向该实体添加能力，这样玩家进入游戏的时候就自动把我们的能力附加上去了

这里需要注意的是，当被附加的实体死亡或穿越维度时，能力并不会自动继承，如果想像死亡不掉落一样持久化保存能力值，需要自己监听玩家重生事件来克隆能力。

``` java
    @SubscribeEvent
    public static void onPlayerCloned(PlayerEvent.Clone event) {
        Player original = event.getOriginal();
        Player newPlayer = event.getEntity();
        original.reviveCaps();
        var oldLevelCap = original.getCapability(LevelProvider.LEVEL);
        var newLevelCap = newPlayer.getCapability(LevelProvider.LEVEL);
        newLevelCap.ifPresent((newLevel) -> oldLevelCap.ifPresent((oldLevel) -> {
            if (event.isWasDeath()) {
                newLevel.setLevel(oldLevel.getLevel() - 1);
            } else {
                newLevel.copyFrom(oldLevel);
            }
        }));
        original.invalidateCaps();
    }
```

这样我们就实现了，死亡会自动扣除一点等级的能力，普通跨越维度会调用我们之前准备好的`copyFrom`方法，深拷贝能力对象。

值得注意的一点是，玩家从末地返回主世界的跨纬度行为，也是会克隆玩家实体的，所以我们在这里要判断是由于死亡造成的玩家实体克隆。

当然，要对玩家的能力进行操作的话也很简单。只需要像这样就能调用能力里的方法进行操作

``` java
    player.getCapability(LevelProvider.LEVEL)
            .ifPresent(levelCapability -> levelCapability.setLevel(level));
```