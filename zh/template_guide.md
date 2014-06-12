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

在正式的项目当中模板文件通常被组织在不同的目录中以便于管理。对Rythm来讲，这带来了一个问题：如何找到并调用在不同目录中的模板文件。幸运的是Java的包管理提供一种很好的方式来处理不同目录中的文件，使用`.`来分割目录路径。假设你的项目中Rythm模板文件被组织在如下目录中：

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

现在`app/rythm/Application/index.html`我们需要调用其他的模板，其调用方式实例如下：

```
@Application.tmplA() @// 全路径调用
@Application.bar.tmplB() @// 全路径调用
@tmplA() @// 相对路径调用
@bar.tmplB() @// 相对路径调用
@foo.zee.tmpl3() @// 全路径调用
```

如果你需要调用在某个其他目录下的多个模板，可以`@import`指令来减少全路径调用：

```
@import foo.*
　
@foo.tmpl2() @// 全路径调用
@foo.zee.tmpl3() @// 全路径调用
@tmpl2() @// import路径调用
@zee.tmpl3() @// import路径调用
```

#### [inv_ext] 调用不同扩展名的模板

在被调用模板和当前模板具有相同扩展名的情况下我们不需要添加扩展名，比方说你在`foo.html`模板中调用`bar.html`模板，可以直接写`@bar()`即可，而不需要`@bar.html()`。

不过如果你要调用的模板和当前模板扩展名不同，则需要添加扩展名。同样在`foo.html`文件中，如果要调用`bar.js`模板，则必须是使用`@bar.js()`。目前Rythm支持以下扩展名：

* `.html`
* `.json`
* `.js`
* `.css`
* `.csv`
* `.tag`
* `.xml`
* `txt`
* `rythm`


#### [inv_dyna] 动态模板调用

有时候被调用模板的名字只能在模板执行的时候才能决定。针对这种情况Rythm提供了`@invoke`指令：

```
@args String platform @// platform可能是"pc", "iphone", "ipad" ...
...
@invoke("designer." + platform)
```

在上面的示例中你有若干模板，其名字分别对应不同的平台。在当前模板中调用这些模板的时候需要根据运行时的平台信息来决定调用哪个模板。`@invoke`指令允许你使用变量来制定需要调用的模板名字。假如在运行时`platform`的值为`"pc"`，上面的代码的效果就相当于

```
@designer.pc()
```

#### [inv_body] 传递代码块给调用模板

Rythm中除了能传递参数给调用模板，也能传递代码块，如下例所示：

```
@greenscript.js() {
    $(function()) {...}
}
```

<div class="alert alert-info"><i class="icon-info-sign"></i> 这个功能相对于FreeMarker的<a href="http://freemarker.org/docs/ref_directive_macro.html#autoid_107">嵌套块</a></div>

##### [inv_body_callback] 模板调用之回调

Rythm引入了模板调用回调特性，可以在调用模板中回调并传递参数给当前模板的代码块

```
@lookupRole(permission: "superuser").callback(List<Role> roleList) {
    <ul>superusers
    @for(Role role: roleList) {
        <li>@role.getName()</li>
    }
    </ul>
}
```

下面是`lookupRole.html`模板:

```
@args String permission
　
@{
    List<Role> roles = Role.find("permission", permission).asList()
}
@renderBody(roles)
```

#### [inv_process] 处理模板调用生成结果

如果没有加上任何调用结果处理扩展，模板调用生成的文本将被直接写在调用模板的当前位置。Rythm提供了一些更能允许用户更加灵活地处理调用结果。

##### [inv_esc] 调用结果转义

可以对模板调用结果进行各种格式的转义：

```
@foo().escape("csv")
```

<div class="alert alert-info"><i class="icon-info-sign"></i> 对调用模板结果进行转义和对一个包括模板调用的代码块进行转义是不同的</div>

```
@escape("csv") {@foo()}
```

上例中并不对`@foo()`调用结果进行转义，而是确保`@foo()`执行过程中所有的表达式都需要做`csv`转义。而`@foo().escape("csv")`则对`@foo()`调用的结果本身进行`csv`转义。

##### [inv_cache] 缓存调用结果

Rythm提过一个比较有趣的模板调用扩展是缓存调用结果。如果某个模板需要被多次调用，则可以使用模板调用缓存扩展。当使用模板调用缓存扩展的时候Rythm会智能地根据调用模板的名字以及传递的参数来确定是否直接返回缓存内容还是需要执行模板调用：

```lang-java,e5fbfe4d819c403b8427ac01598a315a
@bar("foo", true).cache() @// cache using default TTL, which is 1 hour
@bar("foo", true).cache("1h") @// cache invocation result for 1 hour
@bar("foo", true).cache(60 * 60) @// cache invocation result for 1 hour
```

上面示例中的三条调用语句的效果都是一样的，即对`bar`模板的调用结果进行缓存，其对应参数分别是`["foo", true]`。缓存时间是一个小时。在缓存时间内继续用相同的参数调用`foo`模板不会执行调用，而是直接返回缓存内容。

如果模板调用有块传递，则块内容将被用于计算缓存关键字。

在某些情况下模板执行结果不仅仅和参数以及内容块相关，有一些隐式变量也影响调用模板的执行情况。在这种情况下，Rythm缺省的缓存关键字计算机制可能会导致错误的结果。Rythm提供一个方法可以让你显示地传递影响模板执行结果的隐式变量：

```lang-java
@{User user = User.current()}
@bar("foo", true).cache("1h", user.isAdmin())
```

上面的例子中，Rythm将会使用模板`bar`的名字，传递进`@bar()`的两个参数`"foo"`和`true`，以及表达式`user.isAdmin()`的值来计算缓存关键字。

<div class="alert alert-info"><b>参见</b>：<a href="directive.md#cache"><code>@cache()</code>指令</a></div>

##### [inv_assign] 模板调用结果赋值

如果你在多个地方需要使用同一次模板调用的结果，没有必要执行两次模板调用，甚至调用缓存都没有必要，直接将模板调用结果赋值给一个变量即可：

```lang-java,fid-16ecce74f21e4c5db4305c4b4304bb8a
@bar("foo", false).assign(fooResult)
```

执行完上述指令后模板`bar`的调用结果文本会被保存在变量`fooResult`中，你可以对这个变量做任何处理包括在输出：


```
@fooResult.toLowerCase().raw()
```

##### [inv_chain] 串联模板调用扩展

Rythm允许将模板调用的扩展方法串联起来使用：

```lang-java
@someTemplate().raw().cache("1h").assign("xResult").callback(String name) {
    echo @name
}
```

### [inline_tag] 定义并使用内联函数

相对于其他模板引擎来讲，Rythm的模板调用已经非常简单易用了。但是Rythm还是提供了一种更加轻量级的代码复用机制：内联函数。

假设你在同一个模板文件中总是重复相似的结构（注：这是在html源文件中非常普遍的一种现象），但是又没有必要将这种结构提取出来作为一个独立的模板文件，这种情况就非常适合使用内联函数。必须说明一点，作为Rythm的作者，内联函数是我最喜欢的特性之一！

以下代码来自于一个真实的项目，确切地说就是你现在看到的页面（我假设你并不是在github上读这篇文档;-)）

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

注意到没有，内联函数`book`可是帮助我将这段代码从35行简化到了14行。而且可读性增加了好多。

#### [line_tag_limit] 内联函数的限制

尽管内联函数是一个非常爽的特性，我还是必须列出使用它的限制因素：

* 首先你不能直接使用模板调用扩展，包括转义、缓存还有赋值。不过还是有解决办法：

    ```
    @myInlineTag().cache("3h") @// 这个是不行滴
    @// 下面是可以工作的
    @cache("3h") {
        @myInlineTag()
    }

    @myInlineTag().raw().assign("myResult") @// 这行也是不行滴
    @// 下面的方式可以工作
    @chain().raw().assign("myResult") {
        @myInlineTag()
    }
    ```

* 当传递变量给内联函数的时候不能按变量名传递. **始终牢记：内联函数的参数传递只能按位置进行**:

    ```
    @myInlineTag(varName: "value") @// 这个会导致编译错误
    @myInlineTag("value") @// 这样可以
    ```

#### [inline_tag_return_type] 带返回类型的内联函数

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