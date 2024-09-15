# 6.2.1.1 模型错误

## 错误解释

这里的模型错误指的是模型本身的数据有错误，以注册流体为例：

截止2024年9月15日，AI的回答中有这么一句：

```Java
// 在你的 ModFluidTypes 类中
public static final DeferredRegister<FluidType> FLUID_TYPES = DeferredRegister.create(ForgeRegistries.Keys.FLUID_TYPES, "yourmodid");

public static final RegistryObject<FluidType> YOUR_FLUID_TYPE = FLUID_TYPES.register("your_fluid", () -> new FluidType(FluidType.Properties.create()
    .density(1000)       // 密度
    .viscosity(1000)     // 粘度
    .temperature(300)    // 温度
    .lightLevel(0)));    // 光照等级
```

事实上，在将这段代码放入你的项目以后，会发现它直接报错，再仔细研究这个类会发现，这个类里面压根没有create方法或者该方法是私有的。

而在它的回答当中，其实也有部分正确的内容，此时不难推断，AI的数据库当中，有一部分正确的内容，但由于AI无法真正理解相关内容的作用，所以导致给出了部分的错误答案。

出现这种问题往往是因为在版本更替时，这些API经历了大量的变动，导致AI无法正确分别清楚两个版本之间的代码的区别。

## 解决方案

其实没有什么好的解决方案，放弃用AI吧，此时出现这种情况说明AI的认知是有问题的，除非你告诉它到底怎么做才是正确的，但你都能告诉它了，你还要问它干什么呢？