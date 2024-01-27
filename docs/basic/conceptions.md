# 2.x 基础概念
## 事件和事件总线
Forge通过事件总线来拦截游戏中的事件，然后进行处理。

例如，当玩家右键点击方块时，Forge会拦截这个事件，然后进行处理。

主要的事件总线有两条，一条是Mod事件总线，一条是Forge事件总线。

Mod事件均在软件包`net.minecraftforge.client.event`中，而Forge事件均在软件包`net.minecraftforge.event`中。

### Mod事件总线
Mod事件总线我们已经见过两次了，一是在注册的时候，二是在引入现有创造模式物品栏的时候，这段代码就是获取了一个Mod事件总线的实例：

#### 注册物品
注册物品的事件比较特殊，在你使用register方法时就自动帮你订阅了这个事件，所以你不需要再手动订阅了。

```java
var bus = FMLJavaModLoadingContext.get().getModEventBus();
```

然后我们在这条总线上注册了我们的容器：

```java
ITEMS.register(bus);
```

#### 订阅事件
而其他大多数事件都不能用这种形式进行注册，所以我们需要手动告诉Forge我们要订阅这个事件。

像引入现有创造模式物品栏的时候，我们就是这样做的：
```java
@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD, modid = Thirst.MOD_ID)

```

### Forge事件总线
而Forge事件总线我们还没见过，这里先简单说说：

Forge事件总线是一个静态的变量，我们可以通过`MinecraftForge.EVENT_BUS`来获取这个变量的实例。

也就是说我们注册的时候可以这样写：
```java
MinecraftForge.EVENT_BUS.register(/*Something Registerable*/);
```

而参数我们应当填进去一个类本身或者类实例，这个类中会有些方法，它们的参数是事件的实例。

然后只要我们对这个方法添加注解`@SubscribeEvent`，然后添加到时间总线中，这个方法就会在对应的事件发生时被调用。

如果是静态方法，在注册的时候填入的参数是类本身，如果是实例方法，那么就填入类的实例。

和Mod事件总线一样，另外一种方法就是使用注解，只需要把`@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD, modid = MODID)`改成`@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.FORGE, modid = MODID)`即可

可能你没听懂，没关系，后面遇到的时候我会给出一个具体的例子说明的，