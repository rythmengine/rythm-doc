<h1 data-book="template_guide">模板开发指南</h1>

### [introduction]介绍

Rythm模板引擎是一个文本生成工具，用于将动态内容填充进静态模板中

![java-version](../img/hello_world.png)

使用Rythm来生成文本的基本工作就是创建模板文件，在其中写下静态内容，同时使用Rythm的语法来定义动态部分。模板通常都会有1个或者多个模板参数声明。Java程序通过向模板传递不同的模板变量值来生成不同的文本。Rythm模板采用Java语法来构建动态部分。一个Rythm模板由一下几种类型的元素构成：

* 静态部分，其内容直接出现在生成文本中。
* 表达式，用于输出动态内容。参见[表达式详解](expression.md)
* 指令，用于控制模板执行过程。参见[指令参考](directive.md)


#### [at]`@`魔符

Rythm模板语法简洁易学。你只需记住这点就行了：

**`@` 作为唯一特殊符号引导其他所有的Rythm元素**

<div class="alert alert-info"><i class="icon-info-sign"></i> 如果需要在生产文本中输出<code>@</code>的话，你需要使用两个连续的<code>@</code>来表示转义，如下例所示：</div>

```lang-html,fid-c5a55bb34cf4476984a6faea9f875fe0
这是一个邮件地址： someone@@gmail.com
```

#### [static-content]静态内容

任何静态内容（即没有定义Rythm元素包括代码块）将被直接输出到生成文本中，包括空格和换行。

<div class="alert alert-info"><i class="icon-info-sign"></i> 如果模板运行在简并模式下，多余的空格和换行将被过滤掉。参见<a href="#compact">简并输出</a></div>

#### [comment]注释

单行注释由`@//`开始

```lang-java,fid-9a4a6250e85345c980e3d3fe250fa373
@// this is one line comment
```

多行注释块由 `@*` 和  `*@` 定义起始和终结

```lang-html,fid-f8d8dc2ad15e45c193b36a77aa4dccd7
@*
    this is a multiple line comment.
    The text inside this block will be ignore by Rythm template processor
*@
```

### [argument]模板参数声明

Rythm和有些模板引擎（比如Velocity和FreeMarker）不同，除了几种[少见的情况](#when_no_args_decl)，通常来讲一个模板总是需要参数声明来并依此来生成不同的结果文本。Rythm模板使用<a href="directive.md#args"><code>@args</code></a>指令来声明模板变量。

```java
@args User user, Order order
...
```

Rythm需要模板参数声明的主要原因在于Rythm是一个静态类型模板引擎，它解析模板源代码并将其转换为Java源代码然后调用Java编译器生成字节码。这种静态类型模板和动态解释模板（比如Velocity，FreeMarker）相比具有以下优势：

1. 类型安全。编译器可以帮助捕获很多错误
1. 性能更好。

#### [more_about_template_argument] 模板参数声明的小窍门

* 模板参数声明不需要放在模板文件最上面。Rythm的模板参数声明更像C++，只要保证在使用模板变量前声明过即可
* 一个模板可以有多个模板参数声明指令

#### [when_no_args_decl] 模板参数声明免除

一下情况可以省去模板参数声明：

1. 该模板没有变量，所有的动态部分通过调用Java静态方法或者调用其他模板来实现
1. 该模板足够简单，当引用到模板变量的时候没有任何方法和属性调用。在这种情况下Rythm直接使用`toString()`来输出变量内容，并将变量类型定义为`Object`。例如`你好 @username`可以不用声明变量`username`，但是`你好@user.name`会包编译错误，除非申明了变量`user`: `@args User user...`
1. 如果你打开了[类型推导特性](directive.md#feature_type_inference_enabled)，可以不用声明模板变量。不过前提条件是第一次运行模板的时候传递进去的参数不能为空。

<div class="alert alert-warn"><b>注意</b>：建议总是使用模板参数声明，不仅仅因为Rythm需要参数声明来编译模板，而且还增加了模板文档的可读性</div>

请参考[@args 指令](directive.md#args)了解更多模板参数声明规则

### [expression] 表达式

表达式输出是Rythm模板引擎的一项核心功能，用于将动态内容传递到生产文本，包括输出对象字段、方法以及类字段和方法：

```lang-java,fid-2030fcec5c0245af930769663f36bfc3
@args User user, Foo foo
  
// 输出对象实例字段
@foo.bar

// 输出对象实例方法
@user.getName()

// 输出类字段
@Long.MAX_VALUE
@Integer.MAX_VALUE

// 输出类方法
@Boolean.parseBoolean("true")
```

当某些复制表达式可能会和模板其他部分引起混淆的情况下，可以使用括弧`( )`来明确表达式的范围：

```lang-java,fid-4f9eb89804144d7da51cde92e64dc34c
@(1 + 5) @// 打印1 + 5的计算结果

@// 使用 ( ) 将表达式和其他部分分割开
@(foo.bar)_and_some_other_string 
```

#### JavaBeans规范

Rythm不直接支持通过JavaBeans规范获取对象属性。比如`Foo`类有个`getBar()`方法，在Rythm中需要使用`@foo.getBar()`来获取`bar`属性。不过Rythm可以通过动态表达式求解来简化这种表达式。动态表达式需要在普通表达式后面添加一个`@`符号：

```lang-java
@foo.getBar() @//正常获取对象属性方式
@foo.bar@ @//动态表达式求解，注意bar后面那个@
```

需要注意的是动态表达式求解会稍稍损失一些性能。

#### 表达式来源

如[上例](#expression)所示，嵌入在表达式当中的对象/类/字段/方法来自如下几种可能：

* [模板参数声明](#argument)
* [调用其他模板](#invoke_template)
* [模板内置函数](template_built_ins.md)
* 在[代码块]中声明的变量(#scripting)
* 直接的类引用
* 有[@assign()指令](directive.md#assign)赋值的变量
* 由<a href="#inv_assign"><code>.assign()</code></a>模板调用扩展赋值的变量

#### 参见
* [表达式输出转义](expression.md#escape)
* [空值安全](expression.md#null-safe) 
* [表达式输出转换器](expression.md#transformer)

### [scripting]Java代码块

Rythm运行在模板中嵌入Java代码：

```
@{
    String fullName = client.name.toUpperCase() + " " + client.forname;
}
<h1>@fullName，你好</h1>
```

如果在代码块中需要输出内容，可以使用`p()`方法：

```
@{
    String fullName = client.name.toUpperCase() + " " + client.forname;
    p("<h1>").p(fullName).p("，你好</h1>");
}
```

很明显通常情况下直接在代码块中输出并非一种好的选择。

### [flow-control]模板流控

Rythm提供流程控制指令来控制模板执行过程：

#### [brancing] 分支

```
@if (isMobile) {
    @// 生成移动设备相关内容
} else {
    @// 生成普通电脑相关内容
}
```

#### [loop] 循环

```
<ul>
@for(Product product: products) {
    <li>@product.getName(). Price: @product.getPrice().format("## ###,00") €</li>
}
</ul>
```

#### [return] 终止模板执行

```
@if (user.hasRole("guest")) {
    <p>你无权访问当前页面</p>
    @return
}
@// 以下为授权访问内容
```

流程控制指令参考：

* 分支 - <a href="directive.md#if"><code>@if/else/elseif</code></a>
* 循环 - <a href="directive.md#for"><code>@for</code></a>
* 返回 - <a href="directive.md#return"><code>@return</code></a>

### [invoke_template] 模板调用

在模板中调用其他模板是Rythm提供的四种代码复用机制中非常重要的一环。模板调用机制经过精心设计来为Rythm用户提供极致的易用性。

假设你在同一个目录下有一个模板`foo.html`和另一个模板`bar.html`。下面的指令让你在`foo.html`中调用`bar.html`：

```lang-java,fid-ca0397f1d9514836967b820b6b781cc0
@foo()
```

关于模板调用的详尽说明：

* [参数传递](#inv_arg)
* [模板路径的处理](#inv_path)
* [调用不同扩展名的模板](#inv_ext)
* [动态模板调用](#inv_dyna)
* [传递内容块](#inv_body)
    * [内容块回调参数声明](#inv_body_callback)
* [处理模板调用生成结果](#inv_process)
    * [结果转义](#inv_esc)
    * [缓存调用结果](#inv_cache)
    * [调用结果赋值](#inv_assign)
    * [串联调用结果处理方法](#inv_chain)

#### [inv_arg] 模板调用参数传递

如上所述大部分模板都有参数声明。因此在调用其他模板的时候会涉及参数传递问题。还是以`foo.html`和`bar.html`来说明这个问题。假设`bar.html`模板有如下参数声明：

```lang-java
@** the bar.html template **@
@args String whoCallMe, boolean sayBye
你好 @whoCallMe, 这里是bar模板的世界
@if (sayBye) {
@whoCallMe 再见！
}
```

现在我们需要对`foo.html`模板做一些改变：

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar("foo", true)
```

上例中我们在`foo.html`模板中通过参数位置来传递参数给调用模板`bar.html`。Rythm也支持一种更加友好易读的参数传递方式：按参数名传递参数

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar(whoCallMe: "foo", sayBye: true)
```

Rythm甚至支持JSON方式的参数传递

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar({
    whoCallMe: "foo",
    sayBye: true
})
```

#### [inv_path] 模板路径处理

Usually templates are organized in different folders in a non-trial project. This brings the issue of how to invoke a template that is not in the same folder of the current template. Following the idea of java package, Rythm allows it to use dot separated path to identify a template. Consider the following directory structure:

```
\---app
    \---rythm
        |   tmpl1.html
        |
        +---Application
        |   |   index.html
        |   |   tmplA.html
        |   |
        |   \---bar
        |           tmplB.html
        |
        \---foo
            |   tmpl2.html
            |
            \---zee
                    tmpl3.html
```

Now inside the `app/rythm/Application/index.html`, you call the other templates like follows:

```
@Application.tmplA() @// full path invocation
@Application.bar.tmplB() @// full path invocation
@tmplA() @// relative path invocation
@bar.tmplB() @// relative path invocation
@foo.zee.tmpl3() @// full path invocation
```

You can also save typing by using `@import` directive:

```
@import foo.*
　
@foo.tmpl2() @// full path invocation
@foo.zee.tmpl3() @// full path invocation
@tmpl2() @// import path invocation
@zee.tmpl3() @// import path invocation
```

#### [inv_ext] Invoke template with different extension

Normally you don't need to add template extension (e.g. `.html`) when you invoke another template if the template been invoked has the same extension as the current template. For example, if you invoken template `bar.html` from template `foo.html`, simply use `@bar()` is enough.

However if you want to invoke a template with a different extension, say you want to invoke a template `bar.js` from template `foo.html`, you need to add the extension: `bar.js()`

#### [inv_dyna] Dynamic template invocation

There are some cases that template being invoked can only be determined at runtime. Rythm provides dynamic template invocation to handle this situation:

```
@args String platform @// could be pc, iphone, ipad ...
...
@invoke("designer." + platform)
```

In the above code, your templates are stored in different files which corresponding to platform types, and when you are writing the template, you don't know which template exactly to invoke.  The above code enable you to invoke the needed template based on a runtime variable `platform`. So when the `platform` value is "pc", the above code exactly has the same effect as

```
@designer.pc()
```

#### [inv_body]Passing body with template invocation

In Rythm it is possible to pass additional content in addition to parameters when invoking a template:

```
@greenscript.js() {
    $(function()) {...}
}
```

<div class="alert alert-info"><i class="icon-info-sign"></i> This feature is known as <a href="http://freemarker.org/docs/ref_directive_macro.html#autoid_107">nested body</a> in freemarker</div>

##### [inv_body_callback]Pass body callback parameter spec to template call

With the introduction of callback extension, Rythm make it possible to pass in parameters when callback the body in the callee template:

```
@lookupRole(permission: "superuser").callback(List<Role> roleList) {
    <ul>superusers
    @for(Role role: roleList) {
        <li>@role.getName()</li>
    }
    </ul>
}
```

and in your `lookupRole.html` template:

```
@args String permission
　
@{
    List<Role> roles = Role.find("permission", permission).asList()
}
@renderBody(roles)
```

#### [inv_process] Further processing template invocation result

Usually when a template is invoked, the rendering result is inserted at where the invocation is happening. Rythm makes it flexible to allow user to further processing the result before inserted.

##### [inv_esc]Escape invocation result

It is possible to escape template invocation result using specific format:

```
@foo().escape("csv")
```

<div class="alert alert-info"><i class="icon-info-sign"></i> Escape template invocation result is different from escaping a template block contains the invocation </div>

```
@escape("csv") {@foo()}
```

The above code makes sure all expressions output within `@foo()` call will be escaped using “csv” format, while `@foo().escape(“csv”)` ensure the tag invocation result itself get escaped.

##### [inv_cache]Cache invocation result

One interesting tool Rythm provides is it allows you to cache the template call result when it returns and use the cached content next without calling the template again:

```lang-java,e5fbfe4d819c403b8427ac01598a315a
@bar("foo", true).cache() @// cache using default TTL, which is 1 hour
@bar("foo", true).cache("1h") @// cache invocation result for 1 hour
@bar("foo", true).cache(60 * 60) @// cache invocation result for 1 hour
```

The above statement invoke tag myTag using parameter ["foo", true] and cache the result for one hour. Within the next one hour, the tag will not be invoked, instead the cached result will be returned if the parameter passed in are still ["foo", true].

So you see Rythm is smart enough to cache template invocation against tag name and the parameter passed in. If the template invocation has a body, the body will also be taken into consideration when calculating the cache key.

In some cases where the template invocation result is not merely a function of the template, parameter and body, but also some implicit variables, where you might expect different result even you have completely the same signature of template invocation. Rythm provide way for you to take those additional implicit variable into account when calculating the cache key:

```lang-java
@{User user = User.current()}
@bar("foo", true).cache("1h", user.isAdmin())
```

The above statement shows how to pass additional parameter to cache decoration.

<div class="alert alert-info"><b>See also</b>: <a href="directive.md#cache"><code>@cache()</code> directive</a></div>

##### [inv_assign]Assign invocation result

In case you want to reuse a single invocation result in multiple place in the current template, you can use `.assign()` extension to assign the template invocation result into a variable:

```lang-java,fid-16ecce74f21e4c5db4305c4b4304bb8a
@bar("foo", false).assign(fooResult)
```

Now you have a template variable `fooResult` contains the content of `foo.html`. You can process it and output it later on:

```
@fooResult.toLowerCase().raw()
```

##### [inv_chain] Chain the invocation decoration functions

It is possible to chain the invocation extension functions in one statement:

```lang-java
@someTemplate().raw().cache("1h").assign("xResult").callback(String name) {
    echo @name
}
```

### [inline_tag] Define and use inline tag

Although template invocation is already very simple and easy to use compare to other competitors, Rythm provides an even more lightweight way for code reuse: the inline tag.

Suppose you have some repeated code structure in one template source, and you don't think it's heavy enough to isolate them out and put them into a separate template file, then inline tag is your friend. Seriously I found this feature is one of my favourite feature in my daily works.

The following code is picked up from the source code of rythm-web site project, which is what you are looking at now (unless you are reading this doc on github):

```html
<ul class="nav-sub doc clearfix container">
    @def book(String bookId, String url) {
    <li class="book clearfix @bookId" data-url="@url">
        <i class="icon-book" style="font-size: 32px;margin-bottom: 10px;"></i>&nbsp;
        @{bookId = "main.book." + bookId;}
        <div>@i18n(bookId)</div>
    </li>
    }
    @book("index", "/doc/index.md")
    @book("tutorial", "/doc/tutorial.md")
    @book("template_guide", "/doc/template_guide.md")
    @book("developer_guide", "/doc/developer_guide.md")
    @book("configuration", "/doc/configuration.md")
    @book("directive", "/doc/directive.md")
    @book("builtin_transformer", "/doc/builtin_transformer.md")
</ul>
```

The code demonstrate how you define a inline tag "book" and use it immediately after definition. When I create html templates, I found the way is really a finger type saving feature.

#### [line_tag_limit] Limits of inline tag

There are certain limits you should be aware of using inline tag compare to a normal template

* you can not use template invocation extensions (i.e. escape, cache, assign) on an inline tag call, but there are workarounds:

    ```
    @myInlineTag().cache("3h") @// this is NOT okay
    @// the following is good
    @cache("3h") {
        @myInlineTag()
    }

    @myInlineTag().raw().assign("myResult") @// this is NOT okay
    @// the following is good
    @chain().raw().assign("myResult") {
        @myInlineTag()
    }
    ```

* You cannot pass argument by name when invoking inline tag. **ALWAYS pass parameter to inline tag by position**:

    ```
    @myInlineTag(varName: "value") @// this will cause compilation error
    @myInlineTag("value") @// this is okay
    ```

#### [inline_tag_return_type] Inline tag with return type

One of the cool thing is you are not limited to use inline tag to output repeated code, but also use it to define your utility functions, and yes they are translate into protected methods in the source code generated for your template:

```lang-java
@def boolean isMobile() {
    UserAgent ua = UserAgent.current();
    return ua.isMobile();
}
```

And later on it can be used as:

```
@if (isMobile()) {
   @mobile.designer()
} else {
   @pc.designer()
}
```

The following code demonstrate a subtle difference of Java code scripting inside inline tag when return type is presented or not:

```lang-java
@def boolean isMobileAndShowPlatform() {
    UserAgent ua = UserAgent.current();
    boolean isMobile = ua.isMobile();
    p("The platform is ");
    p(isMobile ? "mobile" : "pc");
    return isMobile;
}

@def showPlatform() {
    @{ UserAgent ua = UserAgent.current() }
    The platform is @if (ua.isMobile()) {mobile} else {pc}
}
```

So the rules are

* if inline tag return type is presented and is not `void`
    * Java code shall be put inside tag definition directly
    * Output message shall be done via `p()` function call

* If inline tag return type is absent or is `void`
    * Java code shall be put inside `@{ }` scripting block
    * Output message shall be put inside tag definition directly


### [include]Include other templates

Rythm support include other template inline:

```
@include("foo.bar")
```

The above statement will put the content of template foo.bar in place. “foo.bar” is translate into file name following template invocation convention.

The difference between `@include(“foo.bar”)` a tag and call the template via `@foo.bar()` is the former put the content of the template into the current template inline, while the latter invoke the template specified and insert the result in place. It is some how like `#include` vs. function call in c language. `@include` is super fast to reuse part of template because it suppress the function invocation at runtime. It’s a inline function call if you speak c++. In other words, `@include` process happen at template parsing time, while tag invocation happen at template executing time.

Things you can achieve with `@include` but not template invocation:

* Reuse [inline tag](#inline_tag) definition

Things you can achieve with template invocation but not `@include`

* [Passing parameters](#inv_arg)
* Layout management via [template inheritance](#inheritance)

Because `@include` are parsed at parsing time therefore it’s not possible to include template dynamically as shown below:

```
@// spec could be one of facebook, google
@args String spec
　
@include("page." + spec) @// THIS WON'T WORK
@invoke("page." + spec) @// THIS WORKS!
```

It is also not possible to apply [invocation decorations](#inv_process) to `@include()` for the same reason.

```
@include("my.common.tag.lib").cache().assign("someVariable").raw() @// THIS WON'T WORK
@my.common.tag.lib().cache().assign("someVariable").raw() @// this works
@// the following also works
@chain().cache().assign("someVariable").raw() {
    @include("my.common.tag.lib")
}
```

#### [reuse_inline_tag] Reuse inline tag across multiple views

A good feature provided with `@include` is that you can import the inline tag definition from the template being included:

Suppose you have created a template named `rythm/util.html` with a set of inline tags:

```
@def hi (String who) {
    Hi @who
}
　
@def bye (String who) {
    Bye @who
}
```

Now in your other template you can import all the inline tags `hi`, `bye` and call them like a local function:

```
@include("util")
@hi("rythm")
@bye("rythm")
```

### [macro]Define and execute Macro

Like `@include`, macro provides a way to reuse template content at parsing time, but inside a template file. Suppose you defined a macro called “myMacro”:

```
@macro("myMacro") {
content inside myMacro
}
```

Later on in the same template file you can invoke “macro-1” using the following code:

```
@exec("myMacro")
```

which produce the following output:

```
content inside myMacro
```

At first glance this effect could be achieved using inline tag or even assignment. However they are fundamentally different in that macro executing happen at parsing time while tag call and assignment are happened at runtime, which could result in subtle differences of content been rendered. Let’s take a look at the following code:

```
@macro("foo") {
    <foo>
        <bar>abc</bar>
    </foo>
}
@compact() {@exec("foo")}
@nocompact() {@exec("foo")}
```

So the above code will display “foo” content in compact and nocompact mode. If we change the implementation to:

```
@assign("foo") {
    <foo>
        <bar>abc</bar>
    </foo>
}
@compact() {@foo}
@nocompact() {@foo}
```

The result will not be expected. The reason is assignment will evaluate the content at runtime, and when “foo” content is assigned, the content is already produced and later on calling it in `@compact` and `@nocompact` block will not make any change. For the same reason the following code works neither:

```
@def foo() {
    <foo>
        <bar>abc</bar>
    </foo>
}
@compact() {@foo()}
@nocompact() {@foo()}
```

<div class="alert alert-info"><i class="icon-info-sign"></i> Macro will be expanded at parsing time, therefore it is very fast at runtime but will generate larger class byte codes, furthermore macro will guaranteed to be executed when invoked with <code>@exec</code>, this is unlike <code>@assign</code>, which is executed for only once when assignment happen.</div>

#### Other methods to execute macro

You can also use @expand to execute macro:

```
@expand(myMacro)
```

Or even

```
@myMacro()
```

<div class="alert alert-info"><i class="icon-info-sign"></i> Macro has higher priority than tag. Meaning if you have both “foo” macro and “foo” tag defined, <code>@foo()</code> will invoke <code>foo</code> macro instead of <code>foo</code> tag</div>

### [reuse-methods-summary] Reuse mechanism summary

The following table brief the four reuse mechanisms of Rythm:

<table style="font-size: 11pt;width: 350px;text-align: center">
<thead>
<tr style="border-bottom: 1px solid #aaa">
<th style="border-right: 1px solid #aaa"></th>
<th>External</th>
<th>Internal</th>
</tr>
</thead>
<tbody>
<tr>
<td style="font-weight: bold;padding-right:10px;text-align:right;border-right: 1px solid #aaa">Runtime</td>
<td><a href="#invoke_template">@invoke</a></td>
<td><a href="#inline_tag">@def</a></td>
</tr>
<tr>
<td style="font-weight: bold;padding-right:10px;text-align:right;border-right: 1px solid #aaa">Parsing time</td>
<td><a href="#include">@include</a></td>
<td><a href="#macro">@macro</a></td>
</tr>
</tbody>
</table>

As shown in the above table, each one of the four reuse mechanims has it's characters. Here are some general guide lines to choose which method to use:

* For common code across multiple templates, use external reuse method, i.e. any one of `@invoke` and `@include`
* For repeating patterns appeared within current template, use internal reuse method, say one of `@def` and `@macro`
* If you need to pass parameters to the reuse part, use Runtime methods: `@invoke` or `@def`
* If you don't need parameters to the reuse part, use Parsing time methods: `@include` or `@macro`

### [inheritance] Template inheritance

Template inheritance is a good way to implement template layout management.

```
@// this type of extended template declaration is deprecated: @extends("main.html")
@extends(main)
<h1>Some code</h1>
```

The main template is the layout template, you can use the doLayout tag to include the content of the declaring template:

```
<h1>Main template</h1>
<div id="content">
    @doLayout()
</div>
```

<div class="alert alert-info"><i class="icon-info-sign"></i> <code>render()</code> or <code>renderSection()</code> without section name specified does the same thing with <code>@doLayout()</code></div>

You can even specify the default content if the sub template does not have any content provided:

```
<h1>Main template</h1>
<div id="content">
    @doLayout() {
        default content
    }
</div>
```

#### [ext_lookup]Extended template lookup

Rythm use the same approach to look up extended template and template being invoked, for example, if you want to extend `rythm/layout/foo/bar.html`, you can declare the extend statement as `@extends(layout.foo.bar)`. Please refer to [Handling paths](#inv_path) for more detail.

#### [inheritance_section]Define section and output section

Like Razor, Rythm provides a section concept to enable you to define sections in sub templates and output them in parent template.

In your layout template, say main.html:

```
<!DOCTYPE html>
<html>
<head>
...
</head>
<body>
  <div id="header">
    <h1>@get("title')</h1>
  </div>
　
  <div id="sidebar">
    @// render "sidebar" section
    @render("sidebar")
  </div>
　
  <div id="content">
    @// render main content
    @render() @// or doLayout()
  </div>
　
  <div id="footer">
    @render("footer") {
        @// here we define default content for footer section
        @// if sub template failed to supply this section then
        @// the default content will be output instead
        <p>Site footer - &copy; Santa Clause</p>
    }
  </div>
</body>
</html>
```

And in your working template, say ‘index.html’:

```
@extends(main)
@set(title="Home page")
　
<h2>Welcome to my site</h2>
<p>This is our home page</p>
<p>Not super exciting is it?</p>
<p>Yada, Yada, Yada</p>
　
@section("sidebar") { @// define "sidebar" section
  <p>This sidebar has "Home Page" specific content</p>
  <ul>
    <li><a href="#">Link One</a></li>
    <li><a href="#">Link Two</a></li>
    <li><a href="#">Link Three</a></li>
  </ul>
}
```

In the above example people with sharp eyes can notice that the footer section render in the layout template is different from the sidebar section in that we supplied a default content to the former. So if user did not define the footer section in the sub template, the default content will be output instead.