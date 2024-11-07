# 7.2 Provider

## Provider实现

所有数据生成器都实现了`DataProvider`接口，当然对于基础需求，我们没有必要自己都自己写，原版提供了一套可以直接拿来用的轮子

比如如果我们只需要单纯的为一个物品提供一个贴图，那么可以直接继承原版的`ItemModelProvider`类

``` java
public class ModItem extends ItemModelProvider {
    public ModItem(PackOutput output, String modid, ExistingFileHelper existingFileHelper) {
        super(output, modid, existingFileHelper);
    }

    @Override
    protected void registerModels() {

    }
}
```

然后在`registerModels`方法中，调用`basicItem`方法生成。
注意，basicItem方法需要在对应的`assets/modid/textures/item/`下有一个与物品名称相同的png材质文件，否则会报错无法生成

``` java
@Override
protected void registerModels() {
    this.basicItem(ItemRegistry.YourItem.get());
}
```

## 添加进事件

然后，就是把你自己的数据生成器添加进事件中

``` java
@Mod.EventBusSubscriber(modid = SimpleAlchemy.MODID, bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
public class DataGenerator {
    @SubscribeEvent
    public static void gatherData(GatherDataEvent event) {
        /*
        如上章的部分
        */
        generator.addProvider(
                // 在哪个端被触发
                event.includeClient(),
                // 你自己的数据生成器
                new ModItem(output,MODID,efh)
        );
    }
}
```

然后执行runData，就能看到在resource里面生成了对应的json文件

``` json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "modid:item/item_name"
  }
}
```

## 额外
当然，如果你觉得basicItem方法只能在texture下面找材质文件实在是太傻了，也可以自己写方法来自定义生成
比如如果你想在`texture/cooker`路径下放材质文件，那么就可以自己写这么一个方法

``` java
    public ItemModelBuilder localItem(Item item, String filePath) {
        return localItem(Objects.requireNonNull(ForgeRegistries.ITEMS.getKey(item)),filePath);
    }

    public ItemModelBuilder localItem(ResourceLocation item, String filePath) {
        return getBuilder(item.toString())
                .parent(new ModelFile.UncheckedModelFile("item/generated"))
                .texture("layer0", ResourceLocation.fromNamespaceAndPath(item.getNamespace(), "item/" + filePath + item.getPath()));
    }
```

这样就可以自定义材质文件的路径，就像这样

``` java
this.localItem(ItemRegistry.Cutting_Board.get(), "cooker/");
```

如果是一个BlockItem，还需要渲染时的缩放和角度，也可以写这么一个方法

``` java
    public void blockItem(Item item, String path) {
        blockItem(Objects.requireNonNull(ForgeRegistries.ITEMS.getKey(item)), path);
    }

    public void blockItem(ResourceLocation item, String path) {
        getBuilder(item.toString())
                .transforms()
                .transform(ItemDisplayContext.GUI).rotation(30, 45, 0).scale(0.6F, 0.6F, 0.6F).end()
                .transform(ItemDisplayContext.GROUND).scale(0.23F, 0.25F, 0.25F).end()
                .transform(ItemDisplayContext.FIRST_PERSON_RIGHT_HAND).rotation(0, 45, 0).translation(0, 0, -8).end()
                .transform(ItemDisplayContext.FIRST_PERSON_LEFT_HAND).rotation(0, 45, 0).translation(0, 0, -8).end()
                .transform(ItemDisplayContext.THIRD_PERSON_RIGHT_HAND).rotation(75, 135, 0).scale(0.4F, 0.4F, 0.4F).translation(0, 0, 0).end()
                .transform(ItemDisplayContext.THIRD_PERSON_LEFT_HAND).rotation(75, 135, 0).scale(0.4F, 0.4F, 0.4F).translation(0, 0, 0).end()
                .end()
                .parent(new ModelFile.UncheckedModelFile(resourceLocation(path)));
    }
```