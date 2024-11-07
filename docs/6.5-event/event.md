# 11.0 事件系统

事件系统并不是原版提供的，而是Forge提供给开发者们使用的。当然mod作者自己也可以自己把自己的事件发布到总线上，但那就是后话了。

为什么要有事件系统。经常，开发者需要介入到原版的逻辑中来进行一定的修改，我们可以先看看隔壁没有事件系统的Fabric，很多功能都需要直接对原版代码进行mixin来实现，也就对兼容性和作者水平提出了一定的挑战。

而Forge提出了事件系统，由Forge在源码中插入Hook，将事件发布到总线上，然后开发者们通过监听总线上的事件，实现对原版逻辑的一定程度的干涉。所以从某种意义上来说，Forge本身是最大的coremod。

在Forge开发里有两条总线，Mod总线和Forge总线，所有和初始化相关的事件都是在Mod总线内，其他所有事件都在Forge总线内。

这里先从如何监听事件开始

首先需要给类打上`@Mod.EventBusSubscriber`注解。这个注解就是用来标注这个类里面的所有打了`@SubscribeEvent`静态方法都是事件处理器

``` java
@Mod.EventBusSubscriber
public class PlayerEvent {

}

```

注解中也可以写入一些参数

- `bus` ：用来选择是在哪条总线上进行监听，比如要监听Mod总线，就需要 `bus = Mod.EventBusSubscriber.Bus.MOD`
- `value` ：用来选择在哪个端进行监听，比如要监听的是客户端事件，就需要`value = Dist.CLIENT` 
- `modid` ：用来确定你的modid，可以直接写上`modid = ExampleMod.MOD_ID`

接下来就是事件处理器，需要用`@SubscribeEvent`注解

这个注解中也可以写入参数
- `priority` ：在多个mod同时订阅该事件时，用于判断哪个先进行，为EventPriority枚举类
-

``` java
    @SubscribeEvent
    public static void PlayerXPEvent(PlayerXpEvent.XpChange event) {
        event.setAmount(event.getAmount() + 1);
    }
```

在这个示例中，我们监听了`PlayerXpEvent.XpChange`事件，并把经验值增加了1。

如果你查看`PlayerXpEvent.XpChange`这个类的源码，就会发现这样一行注释

`This event is fired when the player's experience changes through the Player.giveExperiencePoints(int) method. It can be cancelled, and no further processing will be done`

这段话指出，该事件通过在原版方法`Player.giveExperiencePoints(int)`中插入钩子实现，并且可以取消，当加入`event.setCanceled(true);`时即可不执行下面的代码

那让我们看看钩子(Hook)到底是啥

当我们直接阅读这一方法时，能看到如下这段

``` java
   public void giveExperiencePoints(int pXpPoints) {
      net.minecraftforge.event.entity.player.PlayerXpEvent.XpChange event = new net.minecraftforge.event.entity.player.PlayerXpEvent.XpChange(this, pXpPoints);
      if (net.minecraftforge.common.MinecraftForge.EVENT_BUS.post(event)) return;
      pXpPoints = event.getAmount();

```

这里我们看到，Forge把这一段代码插入到原版mc代码中，先将事件发布到Forge总线上。如果事件被取消，那么直接返回方法，否则就拿到事件中xp值，继续执行。也就是说，Forge代替我们解决掉了这些麻烦，我们只需要监听，然后处理就好了。

Forge提供的事件浩如烟海，也就是说，很多其实看上去写死的东西，Forge都提供了事件。如果可以通过事件解决的问题，就可以尽量不要使用mixin。同时，对Forge提供事件的熟悉程度也是很考验开发经验的一件事。所以经常向有丰富经验的人询问也是一个好习惯。