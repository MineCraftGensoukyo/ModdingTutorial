# 8.2 BaseEntityBlock的特性

以下是一些可能需要覆写的`BaseEntityBlock的方法`，来实现一些功能

## getRenderShape
这个方法返回这个方法的渲染方式，默认返回是`INVISIBLE`，也就是不渲染，如果覆写为`MODEL`，就和正常方块的行为一样按assets里的json文件模型

``` java
    public @NotNull RenderShape getRenderShape(@NotNull BlockState blockState) {
        return RenderShape.MODEL;
    }
```

## getTicker
这个方法用于在客户端或者服务端上更新实体

``` java
    @Override
    public <T extends BlockEntity> BlockEntityTicker<T> getTicker(Level level, @NotNull BlockState state, @NotNull BlockEntityType<T> blockEntityType) {
        return level.isClientSide() ? null : createTickerHelper(blockEntityType, BlockEntityTypeRegistry.TEST.get(), TestTE::tick);
    }
```

同时在方块实体TestTE类中创建这个静态方法

``` java
    public static void tick(Level level, BlockPos pos, BlockState state, TestTE entity) {

    }
```

这样就有了一个在服务端每tick都执行的方法了。

## getMenuProvider

如果对应的方块实体实现了`MenuProvider`接口，那么它将返回一个`MenuProvider`对象，可以用来在use方法内打开对应的gui