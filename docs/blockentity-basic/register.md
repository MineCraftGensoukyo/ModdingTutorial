# 8.1 注册方块实体

首先，方块实体的方块应该继承自BaseEntityBlock并实现其中的接口方法newBlockEntity，这里要返回一个方块实体的实例，先按下不表。

``` java
public class TestBlock extends BaseEntityBlock {
    protected TestBlock() {
        super(BlockBehaviour.Properties.of().strength(2.5F));
    }

    @Nullable
    @Override
    public BlockEntity newBlockEntity(BlockPos pPos, BlockState pState) {
        return null;
    }
}
```

然后把这个方块注册进DeferredRegister<Block> BLOCKS中

``` java
    public static final RegistryObject<Block> TEST = BLOCKS.register("crucible", TestBlock::new);
```

方块实体本身需要继承BlockEntity类

``` java
public class TestBE extends BlockEntity {
    public TestBE(BlockPos pPos, BlockState pBlockState) {
        super(BlockEntityTypeRegistry.TEST_BE.get(), pPos, pBlockState);
    }
}
```

下面就是注册方块实体的部分

``` java
public class BlockEntityTypeRegistry {
    public static final DeferredRegister<BlockEntityType<?>> BLOCK_ENTITY_TYPES = DeferredRegister.create(ForgeRegistries.BLOCK_ENTITY_TYPES, SimpleAlchemy.MODID);
    public static final RegistryObject<BlockEntityType<TestBE>> TEST_BE = BLOCK_ENTITY_TYPES.register("test",
            () -> BlockEntityType.Builder.of(TestBE::new, BlockRegistry.TEST.get()).build(null));
}
```

这时候，可以反过来看看我们的方块部分，这样就可以新建一个方块实体的实例了

``` java
    @Nullable
    @Override
    public BlockEntity newBlockEntity(BlockPos pPos, BlockState pState) {
        return new TestBE(pPos, pState);
    }
```

这样，我们就完成了第一个方块实体的注册和与普通方块的绑定。