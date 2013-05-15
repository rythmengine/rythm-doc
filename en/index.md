<h1 data-book="index">Document Center</h1>

### [why]Why Rythm

Check out Rythm [features](feature.md)

### [get]Get Rythm

You can get Rythm in 2 ways:

1. Download Rythm distribution pack by clicking [here](/dist/rythm-engine-1.0-b9-SNAPSHOT-dist.zip). or
1. Use the Maven repository as described below

#### Maven repository

Add Rythm dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.rythmengine</groupId>
    <artifactId>rythm-engine</artifactId>
    <version>1.0-b9-SNAPSHOT</version>
</dependency>
```
    
If you are using SNAPSHOT version of Rythm, make sure you have the following section in the `pom.xml` file also:

```xml
<parent>
  <groupId>org.sonatype.oss</groupId>
  <artifactId>oss-parent</artifactId>
  <version>7</version>
</parent>
```
    
<br/>
    
<div class="alert"><i class="icon-info-sign" style="font-size: 120%"></i>
For a complete maven2 sample application, please find it inside the sample folder of the <a href="/dist/rythm-engine-1.0-b5-SNAPSHOT-dist.zip">distribution package</a>, or browse it on the <a target="_blank" href="https://github.com/greenlaw110/Rythm/tree/1.0/samples/MavnSampleProject">Github</a>
</div>

### [tutorial]Tutorial

A step by step tutorial for new user.

* [Set up your environment](tutorial.md#env)
* [The hello world](tutorial.md#hello)
* [Duke's Bookstore](tutorial.md#bookstore)

### [template-guide]Template Author's Guide

A comprehensive template author's guide book.

* [Introduction](template_guide.md#introduction)
* [Expression](template_guide.md#expression)
* [Invoke other template](template_guide.md#invoke)

### [developer-guide]Developer's Guide

The document details how to use Rythm in your Java program.

* [Introduction](developer_guide.md#introduction)
* [Common usage pattern](developer_guide.md#pattern)
* [To Singleton Or Not](developer_guide.md#singleton_or_not)
* [Configuration](developer_guide.md#Configuration)
* [Render Your Templates](developer_guide.md#render)
* [Render Setting](developer_guide.md#render_setting)
* [Render Context](developer_guide.md#render_context)
* [Other Features](developer_guide.md#miscs)
* [Extends Rythm](developer_guide.md#extension)

### [ref]References

* [Configuration reference](configuration.md)
* [Directive reference](directive.md)
* [Built transformer reference](builtin_transformer.md)

### [misc]Miscs topics

* [How to define your transformer](user_defined_transformer.md)
* [Sandbox mode](sandbox.md)