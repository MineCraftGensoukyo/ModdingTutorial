# 3.4.1 建议自学内容
## 注册相关
是的，我们又需要回到注册那个地方了。

在本章，我们见到过这样一个代码：

```java
super(pProperties.defaultDurability(4));
```

有没有感觉有点眼熟？

还记得我再上一章的时候还用过另一个代码吗：

```java
public static final RegistryObject<Block> CISTERN = BLOCKS.register("cistern", () -> new Block(BlockBehaviour.Properties.of().noOcclusion()));
```

是不是感觉后面半段有点像？

这里我建议读者通过之前说的两个方式，寻找原版物体的注册方式（方块在`Blocks`类中）和按住`Ctrl`单击查看Forge原生代码的方式学习下除了`defaultDurability`以外物品还有哪些Forge自带的功能的和方块除了`noOcclusion`以外还有什么Forge自带的效果。

如果你比较茫然，我可以给你提供一些可能的效果（但不是全部）：

- 物品
  - 可食用
  - 堆叠上限
  - 火焰抵抗
  - ...
- 方块
  - 碰撞体积（是否以及量）
  - 地图颜色
  - 光照等级
  - ...

当然，如果你觉得时间不够，你至少需要掌握这个方法，不必要每一个都记住，需要的时候去找找就行。

## 物品行为相关

当然，物品行为不可能只有教程里提到的像`finishUsingItem`这些，肯定还有不少。

当然，和注册行为一样，你只需要掌握这个方法就好，具体每个方法，我也记不下来，毕竟太多了。