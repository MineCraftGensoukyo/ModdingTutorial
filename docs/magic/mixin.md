# Mixin

Mixin，俗称迷信，使用ASM的Tree API解析的原始字节码。这个API生成一个基于节点的底层字节码视图，然后可以通过Mixin转换器（Transformer）合并到目标类中。

Mixin虽然好用，但是相比较而言也更加危险，兼容性更低，引发的错误也更难排查，很多时候mixin造成的报错会使得日志堆栈都是原版路径，甚至引发某些奇妙的行为。在调试整合包中的mixin问题时候，可以加入-Dmixin.debug.export=true作为jvm参数来导出所有mixin类

关于mixin的原理和文档，可以直接阅读mouse0w0翻译的mixin文档[mixin](https://mouse0w0.github.io/tags/Mixin/)

关于用例，也可以直接阅览fabric的mixin文档[FabricMixin](https://fabricmc.net/wiki/zh_cn:tutorial:mixin_introduction)

(虽然这件事很奇怪，但是fabric的mixin文档确实相比较而言比较完善，Forge甚至没有关于mixin的文档)

这里只给出我个人的几个示例）

## 使用mixin

这里可以直接按照mouse0w0的教程进行配置[在Minecraft Forge中使用Mixin](https://mouse0w0.github.io/2022/03/01/Mixins-on-Minecraft-Forge/)

## 修改返回值

[高级Mixin用法——回调注入器](https://mouse0w0.github.io/2018/12/05/Advanced-Mixin-Usage-Callback-Injectors/)

如果你想使得玩家在装备某个物品的时候，使得死亡音效为biu~时，翻查mc源码发现，玩家死亡音效是写死的

``` java
   protected SoundEvent getDeathSound() {
      return SoundEvents.PLAYER_DEATH;
   }
```

常规方法对这种要求束手无策，这里我们就需要mixin来直接修改这个方法的返回值

``` java
@Mixin(Player.class)
public abstract class MixinPlayer {
    @Shadow public abstract Inventory getInventory();

    @Inject(at = @At(value = "HEAD"),method = "getDeathSound", cancellable = true)
    private void injectAddData(CallbackInfoReturnable<SoundEvent> cir) {
        if (this.getInventory().getArmor(3).is(ItemRegistry.ReimuHeaddress.get())) {
            cir.setReturnValue(SoundRegistry.touhou_dead);
        } else {
            cir.setReturnValue(SoundEvents.PLAYER_DEATH);
        }
    }
}
```

这里我们`mixin了Player`类的`getDeathSound`方法，如果你的IDEA安装了Minecraft Dev插件，就能看到类名坐标有一个SpongePower的logo，这就是mixin类的标志。

Inject注释表明我们将注入这个方法，注释中有两个必要参数，要注入的方法和要注入的点。示例中表明我们将在Player.getDeathSound方法的开头中插入我们的代码，mixin方法的参数可以有Minecraft Dev插件自动生成。

注入点参考如下[Mixin参考——注入点参考](https://mouse0w0.github.io/2020/03/24/Mixin-Reference-Injection-Point-Reference/)

在方法里，我们拿到玩家的物品栏并且判断装备栏的头部是否为这一个物品，如果是，就返回原来的SoundEvents.PLAYER_DEATH

但是显而易见，一个人的mixin对这里进行了修改，那么如果另一个人也想对这里进行修改，那么二者就会打架，虽然mixin也提供了Priority优先级，但是总归还是有一定兼容风险的。

## 使用目标类的内容

有时候，在需要修改的方法中，引用了目标类的方法或者字段，这里就可以使用@Shadow注解来调用，或者强转this类型进行调用

比如目标类中有一个私有字段`private final Minecraft minecraft`，这时候我们就可以在我们的mixin类中直接添加字段

``` java  
    @Final
    @Shadow
    private Minecraft minecraft;
```

同样的，也可以用类似的方法调用方法，比如目标方法为

``` java  
   public int getWidth() {
      return getWidth(this.minecraft.options.chatWidth().get());
   }

我们只需要有样学样的在mixin类中加入如下方法，即可使用

`````` java  
    @Shadow
    public int getWidth() {
        return 0;
    }
```

## 添加新方法

当然，mixin也允许你在类中凭空添加一个自己的方法，也就是重写方法

首先，我们要准备一个接口，这个方法就是我们将要写的新方法了

``` java
public interface GuiGraphicsInterface {
    void livelyDanmaku$drawLine(int x1, int y1, int x2, int y2, int pZ, int pColor, int width);
}
```

然后我们直接对GuiGraphics进行mixin

``` java
@Mixin(GuiGraphics.class)
public abstract class MixinGuiGraphics implements GuiGraphicsInterface {
    @Override
    public void livelyDanmaku$drawLine(int x1, int y1, int x2, int y2, int pZ, int pColor, int width) {

    }
}
```

这里我们的mixin类实现了预设的接口，并覆写了这个方法。在使用的时候我们需要先将GuiGraphics类转为GuiGraphicsInterface接口类，再调用我们自己的方法`livelyDanmaku$drawLine`

``` java
    @Override
    protected void renderWidget(@NotNull GuiGraphics pGuiGraphics, int pMouseX, int pMouseY, float pPartialTick) {
        GuiGraphicsInterface guiGraphics = (GuiGraphicsInterface) pGuiGraphics;
        guiGraphics.livelyDanmaku$drawLine(p1.x, p1.y, p2.x, p2.y, 0, TRANSLUCENT_BLACK, 2);
    }
```

提醒：每个mixin类都必须在`mixins.json`文件中表明