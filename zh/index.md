# 文档中心
﻿
### [why]为什么选择Rythm

点击[Rythm特性](feature.md)获取详细信息

### [get]获得Rythm

通过以下方式可以获得Rythm

1. 点击[此处](/dist/rythm-engine-1.0-b2-SNAPSHOT-dist.zip)下载Rythm发行包。或者
1. 按照以下描述使用Maven构造工具

### [maven]Maven库

在你的`pom.xml`文件中加入Rythm依赖

```xml
<dependency>
    <groupId>com.greenlaw110.rythm</groupId>
    <artifactId>rythm-engine</artifactId>
    <version>1.0-b2-SNAPSHOT</version>
</dependency>
```
    
如果你使用的是Rythm快照（SNAPSHOT）版本则需要在你的`pom.xml`文件中加入下面的代码:

```xml
<parent>
  <groupId>org.sonatype.oss</groupId>
  <artifactId>oss-parent</artifactId>
  <version>7</version>
</parent>
```
    
<br/>
    
<div class="alert">
你可以在<a href="/dist/rythm-engine-1.0-b2-SNAPSHOT-dist.zip">发行包</a>的`sample`目录中找到一个完整的基于maven2构造的应用例子。你也立刻可以在<a target="_blank" href="https://github.com/greenlaw110/Rythm/tree/1.0/samples/MavnSampleProject">Github</a>上浏览这个例子的代码
</div>

### [tutorial]使用指南

* [环境设置](tutorial.md#env)
* [hello world例子](tutorial.md#hello)
* [公爵书店（Duke's Bookstore）](tutorial.md#bookstore)

