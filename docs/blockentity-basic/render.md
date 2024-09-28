# 8.4 方块实体的渲染

我们在前文的BaseBlockEntity篇中提过，默认的getRenderShape方法返回的是RenderShape.INVISIBLE，这意味着mc本想让你的方块实体有着自己的渲染方法，而不是按照一个不会动的json文件。

详细了解渲染方面的知识可以查询这个文档[Cobalt](https://zomb-676.github.io/CobaltDocs/#/README) (Cobalt 是一套 Minecraft 的渲染相关的说明文档)

## BlockEntityRenderer

首先我们需要一个渲染类，实现BlockEntityRenderer<>接口，它需要一个泛型参数来指定方块的 BlockEntity 类，同时在构造方法里初始化了很多用于渲染的对象。

``` java
public class TestBlockRenderer implements BlockEntityRenderer<TestTE> {
    private final ItemRenderer itemRenderer;
    private final EntityRenderDispatcher entityRenderDispatcher;

    public TestBlockRenderer(BlockEntityRendererProvider.Context ctx) {
        this.itemRenderer = ctx.getItemRenderer();
        this.entityRenderDispatcher = ctx.getEntityRenderer();
    }

    @Override
    public void render(TestTE TestTE, float pPartialTick, PoseStack poseStack, MultiBufferSource pBuffer, int pPackedLight, int pPackedOverlay) {
    }
}
```

## 注册
为了注册 BER，您必须在 mod 事件总线上订阅事件 EntityRenderersEvent.RegisterRenderers 并调用 #registerBlockEntityRenderer

``` java
@Mod.EventBusSubscriber(modid = MOD.MODID, bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
public class ClientEvent {
    @SubscribeEvent
    public static void registerRenderers(EntityRenderersEvent.RegisterRenderers event) {
        event.registerBlockEntityRenderer(BlockEntityTypeRegistry.TEST.get(), TestBlockRenderer::new);
    }
}
```

## 渲染示例

当然，这个render方法是每tick调用的，也就是说性能消耗上要大于写入静态缓存的普通方块，但是也能实现很多炫酷的渲染要求。有了上面的itemRenderer，我们就可以自由渲染物品模型，并且玩一些花活

让我们看看以下的一个示例代码

``` java
    @Override
    public void render(CrucibleTE crucibleTE, float pPartialTick, PoseStack poseStack, MultiBufferSource pBuffer, int pPackedLight, int pPackedOverlay) {
        if (!crucibleTE.items.isEmpty()) {
            BlockPos blockPos = crucibleTE.getBlockPos();
            int itemNum = crucibleTE.getItemNum();
            if (itemNum == 0) {return;}
            float angle = 360F / itemNum;
            long l = 0;
            if (crucibleTE.getLevel() != null) {
                l = crucibleTE.getLevel().getGameTime() % 360;
                if (random.nextInt(20) < 1) {
                    crucibleTE.getLevel().addParticle(ParticleTypes.WITCH, blockPos.getX() + random.nextFloat(1), blockPos.getY() + random.nextFloat(0.5F) + 0.25, blockPos.getZ() + random.nextFloat(1), 0.0D, 0.0D, 0.0D);
                }
            }
            for (int i = 0; i < itemNum; i++) {
                poseStack.pushPose();
                ItemStack itemStack = crucibleTE.items.get(i);
                poseStack.translate(0, 0.5, 0.5);
                poseStack.rotateAround(Axis.YP.rotationDegrees(l * 4 + angle * i), 0.5F, 0, 0);
                this.itemRenderer
                        .renderStatic(
                                itemStack,
                                ItemDisplayContext.GROUND,
                                pPackedLight,
                                OverlayTexture.NO_OVERLAY,
                                poseStack,
                                pBuffer,
                                crucibleTE.getLevel(),
                                (int) crucibleTE.getBlockPos().asLong()
                        );
                poseStack.popPose();
            }
        }
    }
```

慢慢剖析这段代码，前面部分就是获取方块实体中存储的物品NonNullList<ItemStack>并计算夹角

这里我们先进行防御式编程，确保level非null，然后通过对游戏时间取模来获得跟随时间增长的数来作为旋转角度


``` java
    if (crucibleTE.getLevel() != null) {
        l = crucibleTE.getLevel().getGameTime() % 360;
    }
```

这里我们我们让每一刻有1/20的几率，在方块内部生成一个紫色的粒子效果，营造出一种魔法的感觉

``` java
    if (random.nextInt(20) < 1) {
        crucibleTE.getLevel().addParticle(ParticleTypes.WITCH, blockPos.getX() + random.nextFloat(1), blockPos.getY() + random.nextFloat(0.5F) + 0.25, blockPos.getZ() + random.nextFloat(1), 0.0D, 0.0D, 0.0D);
    }
```

下面循环中的部分是重头戏

poseStack记录了当前环境的变换，由 Matrix4f pose（位姿）与 Matrix3f normal（法线）组成。poseStack是一个类似堆栈的结构，需要的时候push一个取用，操作完后再pop退出。

在以下代码中，我们先平移后绕Y轴正方向进行旋转一定角度，再调用itemRenderer的renderStatic方法渲染出一个物品。

关于poseStack的操作是如何影响渲染的位置的，可以参考这篇文章[CoordinateSystem](https://zomb-676.github.io/CobaltDocs/#/render/coordinateSystem)


``` java
    poseStack.pushPose();
    ItemStack itemStack = crucibleTE.items.get(i);
    poseStack.translate(0, 0.5, 0.5);
    poseStack.rotateAround(Axis.YP.rotationDegrees(l * 4 + angle * i), 0.5F, 0, 0);
    this.itemRenderer
            .renderStatic(
                itemStack,
                ItemDisplayContext.GROUND,
                pPackedLight,
                OverlayTexture.NO_OVERLAY,
                poseStack,
                pBuffer,
                crucibleTE.getLevel(),
                (int) crucibleTE.getBlockPos().asLong()
            );
    poseStack.popPose();
```

最后，我们就得到了一个把方块实体容器内物品转着旋转的渲染类