# 3.3 给玩家添加物品

## 请牢记

我单独为这一个简单的内容专门开一讲，希望读者们可以理解，这个非常重要。

***禁止在逻辑客户端（包括逻辑的和物理的）修改任何实体***

***禁止在逻辑客户端（包括逻辑的和物理的）修改任何实体***

***禁止在逻辑客户端（包括逻辑的和物理的）修改任何实体***

轻则导致bug（如幽灵实体），重则游戏崩溃。

请牢记这一点。

你可以使用`Level#isClientSide`来判定目前在哪个逻辑端位上执行。

## 添加物品

那么实际的代码很简单，就这点：

```java
Player player = pLivingEntity instanceof Player ? (Player)pLivingEntity : null;
if (!pLevel.isClientSide && player != null) {
    player.addItem(new ItemStack(EMPTY_KETTLE.get()));
}
```

这行代码同时也体现了我们的注册容器其实是个**指针**的特性。

那么同样，我们添加相对应的空水壶和水壶装水的逻辑，请读者自行完成。（放心，Github那个仓库里面不是右键方块的，右键方块得靠读者自己）

## 相关概念

- 端位
