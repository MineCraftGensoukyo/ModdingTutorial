# 2.3 基础方块
## 注册方块
注册方块其实和注册物品很像，只是我们要注册的东西是方块而非物品，所以我们的容器所存储的类型也要改为`Block`。
其它的其实没有什么变化，在这里就直接给出代码了。（注释内说明了类的位置）
### 文件结构

![文件结构](images/structure_2_3.png)

### BlockRegister.java
```java
// moe.gensoukyo.thirst.register.BlockRegister

// imports
public class BlockRegister {
    public static final DeferredRegister<Block> BLOCKS = DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);

    public static final RegistryObject<Block> CISTERN = BLOCKS.register("cistern", () -> new Block(BlockBehaviour.Properties.of()));
}
```
### Thirst.java
```java
// moe.gensoukyo.thirst.Thirst

// imports
@Mod(Thirst.MODID)
public class Thirst {
    public static final String MODID = "thirst";
    private static final Logger LOGGER = LogUtils.getLogger();

    public Thirst() {
        var bus = FMLJavaModLoadingContext.get().getModEventBus();
        ITEMS.register(bus);
        BLOCKS.register(bus);
    }
}
```
## 游戏内生成方块
注意，此处我说的是“生成方块”而非获取方块，此时这个方块还不能被玩家获取，我们只是让它在游戏中生成。
```java
/setblock ~ ~ ~ thirst:cistern
```
![生成方块](images/empty_block.png)