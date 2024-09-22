# 7.1 开始使用dataGen

## 订阅事件
Datagen 通过 Data 运行配置运行，该配置与 Client 和 Server 运行配置一起为您生成。数据运行配置遵循 mod 生命周期，直到触发 registry 事件。然后，它会触发 GatherDataEvent，您可以在其中以数据提供程序的形式注册要生成的对象，将所述对象写入磁盘，然后结束该过程。

特别的，你需要点击右侧forgegradle runs中的runData来开始dataGen的流程


``` java
@Mod.EventBusSubscriber(modid = SimpleAlchemy.MODID, bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
public class DataGenerator {
    @SubscribeEvent
    public static void gatherData(GatherDataEvent event) {
        var efh = event.getExistingFileHelper();
        var generator = event.getGenerator();
        var output = generator.getPackOutput();
        var registries = event.getLookupProvider();
        var vanillaPack = generator.getVanillaPack(true);
        var existingFileHelper = event.getExistingFileHelper();
    }
}
```

## 小贴士
默认生成的资源文件在scr/resources下面，在打包成jar文件的时候会自动合并到main/java/resource里面。

每次生成都是覆写式的，清除旧文件，生成新的。

如果不想把没有意义的dataGen缓存文件塞进远程仓库，可以在.gitignore中加入

``` txt
#datagen
src/generated/resources/.cache
run-data
```