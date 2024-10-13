# 9.1 注册
这部分可以直接参考Flandre923的[第一个实体](https://fuwari-ald.pages.dev/posts/minecraft1_20_4/out_29-%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%AE%9E%E4%BD%93/)


跟方块实体渲染类似，你也可以在render方法里面加入各种自己的操作，比如渲染物品，渲染粒子效果

## 定义属性

当你继承的是LivingEntity的时候，mc会告诉你没有拿到合适的属性，无法生成实体。这时候需要在注册的时候手动指定实体属性

在实体类中，准备好属性的定义
``` java
    public static AttributeSupplier.Builder registerAttributes() {
        return Mob.createMobAttributes()
                .add(Attributes.FOLLOW_RANGE,32.0D)
                .add(Attributes.MOVEMENT_SPEED, 0.3F)
                .add(Attributes.MAX_HEALTH, 100.0D)
                .add(Attributes.ATTACK_DAMAGE, 2.0D);
    }
```
属性的作用和属性名基本相符，比如追踪距离，移动速度，最大血量，攻击伤害等。

接下来就是在事件中增加实体类型其相应的属性

``` java
@Mod.EventBusSubscriber
public class EntityAttributeEvent {
    @SubscribeEvent
    public void onEntityAttributeChange(EntityAttributeCreationEvent event){
        event.put(TestEntity.get(), TestEntity.registerAttributes().build());
    }
}
```

这样就能生成一个正常的LivingEntity了

## 生成

想要生成一个实体，需要在服务端调用addFreshEntity方法，这里给出一个在物品右击事件的时候在目标位置生成一个实体的例子

``` java
public class TestItem extends Item {
    public TestItem() {
        super(new Properties());
    }

    public @NotNull InteractionResult useOn(UseOnContext pContext) {
        Level level = pContext.getLevel();
        if (level instanceof ServerLevel) {
            Player player = pContext.getPlayer();
            Vec3 clickLocation = pContext.getClickLocation();
            if (player != null) {
                TestEntity TestEntity = new TestEntity(level);
                TestEntity.setPos(clickLocation.x, clickLocation.y, clickLocation.z);
                level.addFreshEntity(TestEntity);
                return InteractionResult.SUCCESS;
            }
        }
        return InteractionResult.CONSUME;
    }
}
```

当然，如果你的实体继承自Mob，可以直接使用ForgeSpawnEggItem，就像这样

``` java
    public static final RegistryObject<Item> TestItem = ITEMS.register("test_item", () -> new ForgeSpawnEggItem(EntityTypeRegistry.TEST_ENTITY,0x000000, 0xFFFFFF,new Item.Properties()));
```

我们就定义了一个底色为纯白，壳色为纯黑的刷怪蛋