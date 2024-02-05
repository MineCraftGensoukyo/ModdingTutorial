# 4.3 左键？

好的，那我们现在开始做左键取水的功能。

等等？左键……那不是挖方块嘛？

## 可取消的事件

确实，左键确实是挖方块。

但是左键同样也是一个Forge事件，而这个Forge事件是可取消的，在取消后，对应的效果就不再会发生。

想要取消一个事件可以用这段代码：

```java
event.setCanceled(true);
```

## 实现效果

可以去回顾一下[Forge事件总线](../basic/conceptions.md#forge事件总线)相关内容。

在那里，我曾说过用一个具体例子，那这就来了。

```java
@Mod.EventBusSubscriber(modid = MODID, bus = Mod.EventBusSubscriber.Bus.FORGE)
public class ForgeEventSubscriber {
    @SubscribeEvent
    public static void fillWaterFromCistern(PlayerInteractEvent.LeftClickBlock event) {
    }
}
```

当然，还是一样，这里必须是一个static方法才能被识别（非static方法就不能用注解啦）。

逻辑呢就很简单了，这里只讲空水壶的部分：

1. 先取消事件
2. 如果水箱是空的就结束逻辑
3. 如果水箱不是空的，取水箱的剩余水量和水壶的空余量的较小值。
4. 水箱的水量减去该值，水壶的水量加上该值。

就好啦。

总的代码长这样：

```java
public static void fillWaterFromCistern(PlayerInteractEvent.LeftClickBlock event) {
        BlockPos pos = event.getPos();
        Level level = event.getLevel();
        BlockState blockState = level.getBlockState(pos);
        ItemStack itemStack = event.getItemStack();
        if (itemStack.is(KETTLE.get()) && blockState.getBlock() instanceof CisternBlock) {
            event.setCanceled(true);
            int currentCisternLevel = blockState.getValue(CisternBlock.WATER_LEVEL);
            int kettleLeavingDuration = itemStack.getDamageValue();
            int couldOutputLevel = Math.min(kettleLeavingDuration, currentCisternLevel);
            itemStack.setDamageValue(itemStack.getDamageValue() - couldOutputLevel);
            level.setBlock(pos, blockState.setValue(CisternBlock.WATER_LEVEL, currentCisternLevel - couldOutputLevel), 0b0011);
            if (!level.isClientSide) {
                event.getEntity().swing(InteractionHand.MAIN_HAND);
            }
        } else if (itemStack.is(EMPTY_KETTLE.get()) && blockState.getBlock() instanceof CisternBlock) {
            event.setCanceled(true);
            int currentCisternLevel = blockState.getValue(CisternBlock.WATER_LEVEL);
            if (currentCisternLevel == 0) {
                return;
            }
            final int MAX_KETTLE_LEVEL = 4;
            int couldOutputLevel = Math.min(MAX_KETTLE_LEVEL, currentCisternLevel);
            itemStack.shrink(1);
            if (!level.isClientSide) {
                ItemStack kettle = new ItemStack(KETTLE.get());
                kettle.setDamageValue(4 - currentCisternLevel);
                event.getEntity().addItem(kettle);
                event.getEntity().swing(InteractionHand.MAIN_HAND);
            }
            level.setBlockAndUpdate(pos, blockState.setValue(CisternBlock.WATER_LEVEL, currentCisternLevel - couldOutputLevel));
        }
    }
```