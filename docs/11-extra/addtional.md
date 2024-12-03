# 11.3 附加

## 插件

这里是一些有的没的的数据小插件

### code complexity

用于表明该方法或者类的复杂度，复杂度高说明if套娃等行为过多，也意味着阅读起来困难更大

### WakeTime

用于统计使用IDEA的各项数据，包括使用时长，使用项目，使用语言等等数据

### Statistic

用于统计该项目的代码行数等各项数据

## lombok

Lombok是一个Java库，能自动插入编辑器并构建工具，简化Java开发。通过添加注解的方式，不需要为类编写getter或eques方法

在gradle中导入该库：在dependencies中加入如下内容

``` gradle
dependencies {
    compileOnly 'org.projectlombok:lombok:1.18.28'
    annotationProcessor 'org.projectlombok:lombok:1.18.28'
}
```
