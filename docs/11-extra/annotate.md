# 11.2 注解
或许你在写代码的时候，经常被IDEA说教某某字段标黄，说要注解为@NotNull，尤其是渲染方法中。

这里可以在文件夹中添加一个`package-info.java`的文件，内容如下

``` java
@ParametersAreNonnullByDefault
@MethodsReturnNonnullByDefault
package com.gensoukyo.simple_chat.gui;

import net.minecraft.MethodsReturnNonnullByDefault;

import javax.annotation.ParametersAreNonnullByDefault;
```

这样就能把处于同一文件夹中的NotNull标黄通知都去掉了