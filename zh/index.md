@nocompact() {
<h1 data-book="index">文档中心</h1>

### [why]为何使用Rythm

参见[Rythm特性](feature.md)

### [get]获取Rythm

两种方式可以获取Rythm：

1. 下载[Rythm发行包](@_play.configuration.get("dist.url"))
1. 使用Maven库：


#### Maven库

将Rythm依赖加入你的`pom.xml`文件：

```xml
<dependency>
    <groupId>org.rythmengine</groupId>
    <artifactId>rythm-engine</artifactId>
    <version>1.0.1-SNAPSHOT</version>
</dependency>
```
    
如果使用SNAPSHOT版本你需要在你的`pom.xml`文件中加入以下块：

```xml
<parent>
  <groupId>org.sonatype.oss</groupId>
  <artifactId>oss-parent</artifactId>
  <version>7</version>
</parent>
```
    
<br/>
    
### [tutorial] 教程

Rythm新用户可以通过教程来逐步了解Rythm

* [环境配置](tutorial.md#env)
* [hello world程序](tutorial.md#hello)


### [template-guide] 模板指南

为模板作者提供的详尽说明书

* [Rythm介绍](template_guide.md#introduction)
* [模板参数声明](template_guide.md#argument)
* [表达式](template_guide.md#expression)
* [流控](template_guide.md#flow-control)
* [模板调用](template_guide.md#invoke_template)
* [内联函数](template_guide.md#inline_tag)
* [包含其他模板](template_guide.md#include)
* [宏定义及使用](template_guide.md#macro)
* [模板继承和布局](template_guide.md#inheritance)

### [developer-guide] 开发指南

详尽说明如何将Rythm和你的Java程序集成

* [简洁](developer_guide.md#introduction)
* [使用模式](developer_guide.md#pattern)
* [单例与否](developer_guide.md#singleton_or_not)
* [配置Rythm](developer_guide.md#Configuration)
* [模板生成](developer_guide.md#render)
* [运行时设定](developer_guide.md#render_setting)
* [运行时上下文](developer_guide.md#render_context)
* [其他功能](developer_guide.md#miscs)
* [扩展Rythm](developer_guide.md#extension)

### [ref] 参考手册

* [配置参考](configuration.md)
* [指令参考](directive.md)
* [内置表达式转换器](builtin_transformer.md)

### [misc] 其他文档

* [如何自定义转换器](user_defined_transformer.md)
* [沙箱模式](sandbox.md)
}
