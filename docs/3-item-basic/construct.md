# 3.1 新建物品类

## 开始自定义一个物品

正如开始所讲，这一章是要制作一个可以多次饮用的水壶。显然，那个几乎没有什么用处的水壶是不行的，我们需要自定义一个物品。

那么首先，我们需要**继承**Item**类**，并且实现**父类**的**构造函数**，这样我们才能在注册的时候成功将它注册为物品。

```java
// imports 从本章开始不再写这个

public class EmptyKettleItem extends Item {
    public EmptyKettleItem(Properties pProperties) {
        super(pProperties);
    }
}
```

然后给它注册进去：

```java
    public static final RegistryObject<Item> EMPTY_KETTLE = ITEMS.register("empty_kettle", () -> new EmptyKettleItem(new Item.Properties()));
```

然后放进创造模式物品栏：

```java
    @SubscribeEvent
    public static void addCreativeTab(BuildCreativeModeTabContentsEvent event) {
        if (event.getTabKey() == CreativeModeTabs.FOOD_AND_DRINKS) {
            event.accept(EMPTY_KETTLE.get());
            event.accept(KETTLE.get());
        }
    }
```

然后调整模型、材质和i18n的资源文件（这里不再赘述，和之前用的kettle用的同一个材质）

好了，我们又得到了一个没什么用的物品。

## 一些声明

如果我在这里教右键水往水壶里面灌满水，那这个教程作为入门来讲有点难了。我将会教读者右键任意方块来灌满水，至于怎么右键水，读者们可以按照我教的方法自行探究。

## 思考功能

- 可以多次饮用
- 空水壶可以堆叠在一个物品栏内
- 右键方块灌满水

接下来我们就来实现它。