# 7.3 Block 与 Blockstate 

## 介绍
方块模型由两部分组成BlockModel和BlockState。
前者决定了该方块的手持时的渲染。后者决定在世界中的渲染形态。
缺失前者：无论如何都是紫黑块。缺失后者：放在世界当中时，会变成紫黑块。

## 使用
当美工给你发来了一个blockbench导出的物品/方块模型json文件时，就可以继承`BlockStateProvider`为其生成对应的blockstate

``` java
public class ModBlockState extends BlockStateProvider {
    public ModBlockState(PackOutput output, String modid, ExistingFileHelper exFileHelper) {
        super(output, modid, exFileHelper);
    }

    @Override
    protected void registerStatesAndModels() {

    }
}
```

比如你需要生成一个四个方向旋转的blockstate，就可以用horizontalBlock方法

``` java
    @Override
    protected void registerStatesAndModels() {
        this.horizontalBlock(BlockRegistry.Grill.get(), new ModelFile.UncheckedModelFile(new ResourceLocation(MODID,"block/grill")));
    }
```

运行runData后，就有一个生成好的blockstate文件了

## 附录
这里有原版的方法对应生成的模型模板[skyinr的Datagen教程](https://skyinr.github.io/DatagenBook/#/2?id=blockstateprovider%e6%a8%a1%e5%9e%8b%e7%94%9f%e6%88%90%e6%96%b9%e6%b3%95%e8%af%b4%e6%98%8e)