# 4.2 方块状态

方块状态是什么我们在前面已经解释过了。

我们现在就要为我们的水箱添加“水位”这一状态，在这里，可以去回顾下[Blockstates文件的讲解](../basic/complete.md#讲解blockstates文件)

## 初始化方块状态

这一部分内容也是可以从原版中找到参考的，可以参考按钮相关的内容。

我们为方块类里定义一个Property，简单来讲就是定义了一个状态：

```java
public static final IntegerProperty WATER_LEVEL = IntegerProperty.create("level", 0, 4);
```

其中三个参数分别对应了：

- 状态的id，我们将会在修改blockstates文件的时候用到它
- 后面两个参数根据不同的Property的类型有所不同（数量也会不同），对于IntegerProperty来说是最小值和最大值

你完全可以通过实现`Property<?>`来自定义Property，这里提供几个原版给出的实现

- IntegerProperty，实现了Property<Integer>
- BooleanProperty，实现了Property<Boolean>
- EnumProperty<E extends Enum<E>>，实现了Property<E>
- DirectionProperty，是EnumProperty<Direction>的便利实现。

然后我们将这个Property添加到这个方块的每个方块状态上：

```java
@Override
protected void createBlockStateDefinition(StateDefinition.Builder<Block, BlockState> pBuilder) {
    pBuilder.add(WATER_LEVEL);
}
```

为它提供一个默认值：

```java
    @Nullable
    @Override
    public BlockState getStateForPlacement(@NotNull BlockPlaceContext pContext) {
        return this.defaultBlockState().setValue(WATER_LEVEL, 0);
    }
```

至此初始化的内容就完成了。

## 右键升高水位

首先我要说明的是，这里用的是一个已经被弃用的方法（但是它仍然能用，不过我目前尚且不知道怎么让它能够应用左手右键）。

这里提供两种思路来解决这个问题（但是教程中没有完成这件事）：

- 参考稍后的左键取水的方式进行逻辑的编写
- 将这件事情做在`Item`里面，如果Item的代码过长了，可以尝试使用`Block`和`Function`的**映射**关系缩减代码

那么还是一样的，找到我们需要用的方法，这里使用的方法是`use`。

```java
@Override
public @NotNull InteractionResult use(@NotNull BlockState blockState, @NotNull Level level, @NotNull BlockPos blockPos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult result)
```

然后我们来为它添加逻辑。

在这个逻辑中，我们首先需要确定我们手上的是不是水壶。

如果是水壶，那么取水壶剩余的水量和水箱里面还剩余的空间的最小值，然后水壶里面减少这部分水量，水箱中增加这部分水量。

很自然的，我们就有了这段代码：

```java
@Override
public @NotNull InteractionResult use(@NotNull BlockState blockState, @NotNull Level level, @NotNull BlockPos blockPos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult result) {
    ItemStack itemStack = player.getItemInHand(hand);
    if (itemStack.is(KETTLE.get())) {
        int kettleCurrentDuration = itemStack.getMaxDamage() - itemStack.getDamageValue();
        int currentCisternLevel = blockState.getValue(WATER_LEVEL);
        int cisternLeavingLevel = 4 - currentCisternLevel;
        int couldInputLevel = Math.min(kettleCurrentDuration, cisternLeavingLevel);
        itemStack.setDamageValue(itemStack.getDamageValue() + couldInputLevel);
        if (itemStack.getDamageValue() == itemStack.getMaxDamage()) {
            itemStack.shrink(1);
            if (!level.isClientSide) {
                player.addItem(new ItemStack(EMPTY_KETTLE.get()));
            }
        }
        blockState.setValue(WATER_LEVEL, currentCisternLevel + couldInputLevel); // ?
    }
    return InteractionResult.SUCCESS;
}
```

## 修改Blockstates文件

在本章开始之前，我让读者回顾了下Blockstates相关的内容，没错，我们的`level`可以用到Blockstates里面，我们可以把Blockstates改成这样：

```java
{
  "variants": {
    "level=0": [{
      "model": "thirst:block/cistern"
    }],
    "level=1": [{
      "model": "thirst:block/cistern1"
    }],
    "level=2": [{
      "model": "thirst:block/cistern2"
    }],
    "level=3": [{
      "model": "thairst:block/cistern3"
    }],
    "level=4": [{
      "model": "thirst:block/cistern4"
    }]
  }
}
```

这样，每个不同的水位就会有不同的模型了，代码已经在[GitHub](https://github.com/MineCraftGensoukyo/Thirst/tree/chapter4)仓库上了，就不在这里放模型代码和材质图片了（太长了）

好的，到这里位置，思路清晰，逻辑通顺，一气呵成。

打开游戏，拿起水壶，右键水箱……？

对，对吗？

## 不可变的方块状态

那找找原因吧——要么是方块状态错误，要么是渲染错误。

很简单，再拿一个水壶右键它，然后发现，水壶里的水又没了。

那很明确了，Property错误。

检查一遍逻辑，没啥问题。

那这一切就指向了一件事情：Property没有正确变化。

事实上，在Forge中，当一个方块被放置时，它的Property就已经被确定了，是不可更改的。

我们如果想要修改一个方块的Property，那么必须重新放置那个方块。

现在，我们把我们之前的代码改一下：

```java
    @Override
    public @NotNull InteractionResult use(@NotNull BlockState blockState, @NotNull Level level, @NotNull BlockPos blockPos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult result) {
        ItemStack itemStack = player.getItemInHand(hand);
        if (itemStack.is(KETTLE.get())) {
            int kettleCurrentDuration = itemStack.getMaxDamage() - itemStack.getDamageValue();
            int currentCisternLevel = blockState.getValue(WATER_LEVEL);
            int cisternLeavingLevel = 4 - currentCisternLevel;
            int couldInputLevel = Math.min(kettleCurrentDuration, cisternLeavingLevel);
            itemStack.setDamageValue(itemStack.getDamageValue() + couldInputLevel);
            if (itemStack.getDamageValue() == itemStack.getMaxDamage()) {
                itemStack.shrink(1);
                if (!level.isClientSide) {
                    player.addItem(new ItemStack(EMPTY_KETTLE.get()));
                }
            }
            level.setBlock(blockPos, blockState.setValue(WATER_LEVEL, currentCisternLevel + couldInputLevel), 0b0011);
        }
        return InteractionResult.SUCCESS;
    }
```

此时再进入游戏就会发现，一切都正常了。