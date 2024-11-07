# 5.2.1 创建HUD

和我们之前学过的所有内容不同的是，HUD全部被定义在`VanillaGuiOverlay`这一Enum下而非Class下，同时它存放在`MinecraftForge`的子包下而非`Minecraft`的子包下。

Forge的HUD的定义相对比较复杂，这里用一个简单一点的形式（当然，代价就是HUD变多了之后代码会变得冗长）

## HUD创建

创建一个渲染文字的HUD，把它给做在一个类里面。

```java
public class WaterLevelHud {
    private static int waterLevel = 0;

    public static final IGuiOverlay WATER_LEVEL_HUD = (((gui, guiGraphics, partialTick, screenWidth, screenHeight) -> {
        var font = gui.getMinecraft().font;
        var text = "Water Level: " + waterLevel;
        guiGraphics.drawString(font, text, 100, 100, 0xFFFFFF);
    }));
}
```

然后我们在模组事件监听器里面把HUD注册进去：

```java
@SubscribeEvent
public static void addHud(RegisterGuiOverlaysEvent event) {
    event.registerAbove(VanillaGuiOverlay.HOTBAR.id(), "water_level_hud", WATER_LEVEL_HUD);
}
```

然后我们再给水箱添加一个`changeWaterLevel`的方法，在每次调用的时候改一下这里的静态变量，然后改下之前在网络里面做的`handle`方法。

这个HUD就大功告成了……吗？

对，对吗？

让我们进游戏试试

Boom!恭喜你，你的模组迎来了本教程教学至今的第一次崩溃！可喜可贺可喜可贺。

## 端位控制

我们想想，我们的`handle`是在哪里执行的，服务端对吧？

回顾下[端位](../../item-basic/conceptions.md#物理端位)的相关内容，物理客户端的操作在物理服务端执行是会引发崩溃的。

再仔细看看，如果想让一个类只在物理客户端运行，那么需要添加一个`@OnlyIn(Dist.CLIENT)`注解。

```java
@OnlyIn(Dist.CLIENT)
public class WaterLevelHud {
    private static int waterLevel = 0;

    public static final IGuiOverlay WATER_LEVEL_HUD = (((gui, guiGraphics, partialTick, screenWidth, screenHeight) -> {
        var font = gui.getMinecraft().font;
        var text = "Water Level: " + waterLevel;
        guiGraphics.drawString(font, text, 100, 100, 0xFFFFFF);
    }));
}
```

好的，我们的HUD就可以正常运行了……吗？

我们是不是还忘了什么？

## 网络I/0

还是之前那句话，`handle`是在服务端运行的，但是我们的HUD是在客户端。

那凭啥数据能直接互通啊，那就需要网络了啊！

这部分的实现我不再在教程中书写，还请读者自行完成。