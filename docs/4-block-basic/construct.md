# 4.1 一些重复的操作

和物品一样，我们需要新建一个方块类，然后我们需要为它添加逻辑。

由于这部分内容和物品的修改基本一致，这里就直接提供代码了：

```java
public class CisternBlock extends Block {
    public CisternBlock(Properties pProperties) {
        super(pProperties.strength(1.5f));
    }

    @Override
    public @NotNull InteractionResult use(@NotNull BlockState blockState, @NotNull Level level, @NotNull BlockPos blockPos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult result) {
        //...
    }
}
```

```java
public static final RegistryObject<Block> CISTERN = BLOCKS.register("cistern", () -> new CisternBlock(BlockBehaviour.Properties.of().noOcclusion()));
```