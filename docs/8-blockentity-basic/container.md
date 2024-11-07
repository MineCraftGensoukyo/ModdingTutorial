# 8.5 容器

一个容器需要实现Container接口。但是自己写全部的逻辑实在有点复杂，当然，你也可以学习原版箱子，直接继承`RandomizableContainerBlockEntity`来节省很多工作。Forge提供的`Capability`的`ForgeCapabilities.ITEM_HANDLER`也可以替代这项工作，但是是后话了。

## RandomizableContainerBlockEntity

继承这个类后，还是有一些东西需要你自己实现，这里给出一个例子，在这里我们给这个方块实体添加了一个大小为27的容器，

``` java
public class TestTE extends RandomizableContainerBlockEntity {
    private final int SIZE = 27;
    NonNullList<ItemStack> items = NonNullList.withSize(SIZE, ItemStack.EMPTY);

    protected TestTE(BlockPos pPos, BlockState pBlockState) {
        super(BlockEntityTypeRegistry.TestTE.get(), pPos, pBlockState);
    }

    @Override
    protected NonNullList<ItemStack> getItems() {
        return this.items;
    }

    @Override
    protected void setItems(NonNullList<ItemStack> pItemStacks) {
        this.items = pItemStacks;
    }

    @Override
    protected Component getDefaultName() {
        return Component.translatable("ui.modid.new_te");
    }

    @Override
    protected AbstractContainerMenu createMenu(int pContainerId, Inventory pInventory) {
        return null;
    }

    @Override
    public int getContainerSize() {
        return SIZE;
    }
}
```

看起来这已经是一个完善的容器了，看起来。但是他没有数据持久化，退出重进游戏，你的数据就，没啦。

这时候我们需要回顾一下上一章的知识，把物品数据通过`load`和`saveAdditional`保存，这里可以用原版的工具类`ContainerHelper`提供的方法

``` java
    public void load(CompoundTag pTag) {
        super.load(pTag);
        this.items = NonNullList.withSize(this.getContainerSize(), ItemStack.EMPTY);
        ContainerHelper.loadAllItems((CompoundTag) getPersistentData().get("test_item"), this.items);
    }

    protected void saveAdditional(CompoundTag pTag) {
        CompoundTag nbt = new CompoundTag();
        ContainerHelper.saveAllItems(nbt, this.items);
        getPersistentData().put("test_item", nbt);
        super.saveAdditional(pTag);
    }
```

仔细的读者已经发现了，上文的`createMenu`方法返回了null，这是用来玩家与这个容器方块实体交互的时候打开一个gui，比如玩家与箱子交互时，就会打开ChestMenu，这就是后话了，有兴趣的可以自己看原版木桶的逻辑的实现（因为箱子涉及到大箱子时，有两个方块，逻辑比较复杂）