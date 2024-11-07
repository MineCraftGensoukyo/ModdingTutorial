# 7.4 语言文件

## 介绍

语言文件本质上就是一对键值对，可以继承`LanguageProvider`来快速生成对应的语言文件

``` java
public class ModLanguage extends LanguageProvider {
    public ModLanguage(PackOutput output, String modid, String locale) {
        super(output, modid, locale);
    }

    @Override
    protected void addTranslations() {

    }
}
```

`LanguageProvider`有多个重载方法add()，这是为了写的时候方便很多
比如我们想直接写一对翻译键值对，就可以

``` java
    @Override
    protected void addTranslations() {
        this.add("itemGroup.mystia_izakaya", "Mystia's Izakaya");
    }
```

如果想要快速给物品/方块设置名字，也可以直接调用`add`方法，这也得益于父类中这么多的重载方法

``` java
        this.add(ItemRegistry.ChromeBall.get(), "ChromeBall");
```

但是这个文件只能用来放一个语言对应的语言文件，如果我们需要英文和中文，就需要两个类`ModLanguageEN`和`ModLanguageCN`分别继承`LanguageProvider`，再分别在`DataGenerator`中调用

``` java
        //Language
        event.getGenerator().addProvider(
                event.includeClient(),new ModLanguageEN(output,MODID,"en_us"));
        event.getGenerator().addProvider(
                event.includeClient(),new ModLanguageCN(output,MODID,"zh_cn"));
```

## 附加

如果你觉得想加个东西，中英文语言还得跑两个文件实在是太憨了，这里有一个非常奇妙的解决方案，两个语言，一次满足

``` java
public class ModLanguage implements DataProvider {
    private final Map<String, String> enData = new TreeMap<>();
    private final Map<String, String> cnData = new TreeMap<>();
    private final PackOutput output;
    private final String locale;

    public ModLanguage(PackOutput output, String locale) {
        this.output = output;
        this.locale = locale;
    }

    private void addTranslations() {
        this.add(SimpleAlchemy.MODID, "Simple Alchemy", "简易炼金");
        this.add(ItemRegistry.Crucible.get(), "Crucible", "坩埚");
    }

    @Override
    public @NotNull CompletableFuture<?> run(@NotNull CachedOutput cache) {
        this.addTranslations();
        Path path = this.output.getOutputFolder(PackOutput.Target.RESOURCE_PACK)
                .resolve(SimpleAlchemy.MODID).resolve("lang");
        if (this.locale.equals("en_us") && !this.enData.isEmpty()) {
            return this.save(this.enData, cache, path.resolve("en_us.json"));
        }

        if (this.locale.equals("zh_cn") && !this.cnData.isEmpty()) {
            return this.save(this.cnData, cache, path.resolve("zh_cn.json"));
        }

        return CompletableFuture.allOf();
    }

    private CompletableFuture<?> save(Map<String, String> data, CachedOutput cache, Path target) {
        JsonObject json = new JsonObject();
        data.forEach(json::addProperty);
        return DataProvider.saveStable(cache, json, target);
    }

    public void add(Block key, String en, String cn) {
        this.add(key.getDescriptionId(), en, cn);
    }

    public void add(Item key, String en, String cn) {
        this.add(key.getDescriptionId(), en, cn);
    }

    private void add(String key, String en, String cn) {
        if (this.locale.equals("en_us") && !this.enData.containsKey(key)) {
            this.enData.put(key, en);
        } else if (this.locale.equals("zh_cn") && !this.cnData.containsKey(key)) {
            this.cnData.put(key, cn);
        }
    }

    @Override
    public @NotNull String getName() {
        return "language:" + this.locale;
    }
}
```

这里基本上照抄了`LanguageProvider`的逻辑，并进行了一些小修改，在`addTranslations`方法中，调用我们自己的`add`方法，分别传入目标键，英文值，中文值，这样就可以在`DataGenerator`里面这样调用，实现我们的愿望。

``` java
        //Language
        generator.addProvider(
                event.includeClient(), new ModLanguage(output, "zh_cn"));
        generator.addProvider(
                event.includeClient(), new ModLanguage(output, "en_us"));
```