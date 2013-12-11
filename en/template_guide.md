<h1 data-book="template_guide">Template Author's Guide</h1>

This document details how to use Rythm to write template source.

### [introduction]Introduction

Rythm Template Engine is a text generator that merges dynamic content into static template. 

![java-version](../img/hello_world.png)

In order to use Rythm to generate a text file, the essential work is to create template file which write down the static part and use Rythm syntax to define the dynamic parts. It is also important to understand that a template file has one or more arguments which is the origin of the dynamic content. The Java program pass parameters to template file via arguments to generate different result.

So a template file is a text file, some parts of which have placeholders for dynamically generated content. The template’s dynamic elements are written using the Java language.

Dynamic elements are resolved during template execution. The rendered result is then sent as part of the HTTP response.

In summary, a rythm template is composed of the following types of components:

* static content, which is literally output to the render result without any changes
* expressions, which output the dynamic content to the render result. See [more about expressions](expression.md)
* directives, which control the behavior of the template in this and that way. Please refer to [directive reference](directive.md)


#### [at]The magic `@`

The Rythm template syntax is designed to be easy to learn and what you need to understand is one fact:

**The magic `@` character is used to lead all rythm element.**

<div class="alert alert-info"><i class="icon-info-sign"></i> If you want to output a literal <code>@</code> character, you need to double it. E.g.</div>

```lang-html,fid-c5a55bb34cf4476984a6faea9f875fe0
This is an email address in template: someone@@gmail.com
```

#### [static-content]Static content

All static content (those text not defined in Rythm elements including script blocks) are output literally including whitespace and new lines. 

<div class="alert alert-info"><i class="icon-info-sign"></i> When a template part is executed in compact mode, the additional whitespace and new lines are removed. See <a href="#compact">Compact output</a></div>

#### [comment]Comment

one line comment start with `@//`

```lang-java,fid-9a4a6250e85345c980e3d3fe250fa373
@// this is one line comment
```

multiple lines comment are put inside `@*` and  `*@` block.

```lang-html,fid-f8d8dc2ad15e45c193b36a77aa4dccd7
@*
    this is a multiple line comment.
    The text inside this block will be ignore by Rythm template processor
*@
```

### [argument]Template arguments

When a template is rendered, it nearly always needs to pass some dynamic information to the template. This is done via template arguments. Unless [rare cases](#when_no_args_decl) you should declare arguments of a template using <a href="directive.md#args"><code>@args</code></a> directive:

```java
@args User user, Order order
...
```

The main reason that argument declaration is needed is that Rythm is a static typed template engine, it needs to parse your template source code and convert it into a valid java source code and then call a compiler to compile the template into byte code. The benefit of this static typed processing (vs. interpreted processing as velocity, freemarker etc) is:

1. It's type safe and thus compiler help you capture some errors
1. It's much faster than interpreting processing

#### [more_about_template_argument]More about template argument:

* You are not required to declare template arguments at top of the template source code. It's more like a c++ style, you just need to declare arguments before they are used.
* You can have multiple `@args` declaration in a single template source file

#### [when_no_args_decl]When you can waive template arguments declaration

In some cases you can waive for template argument declaration:

1. The template does not have arguments. All dynamic information is retrieved out via method calls, tag calls etc. Obviously you don't need to declare something that doesn't exists
1. Your template is simple enough that you don't need to call any field or method on your template arguments, the only thing you need is the `toString()` method, in that case you can waive argument declaration and Rythm will treat those arguments as an `Object` type. Note you can't have complex expression on those arguments. E.g `Hi @username` is okay, but `Hi @user.name` won't be compiled unless you declared `@args User user`
1. You enabled the [feature.type_inference](directive.md#feature_type_inference_enabled) configuration and you are sure that the first time running the template, all parameters are not null

<div class="alert alert-warn"><b>Note</b>: it is always recommended to declare your template arguments, not only because it is required, but also it improves your template readability</div>

For more about template argument declaration, please refer to [@args directive](directive.md#args)

### [expression]Expression

Output expression is the core function of Rythm template, it is used to emit dynamic content passed into the template, including emission of object fields and methods, class fields and methods: 

```lang-java,fid-2030fcec5c0245af930769663f36bfc3
@args User user, Foo foo
  
// emit instance field
@foo.bar

// emit instance method
@user.getName()

// emit class field
@Long.MAX_VALUE
@Integer.MAX_VALUE

// emit class method
@Boolean.parseBoolean("true")
```

The bracket `( )` can be used to compose complicated expressions or to separate an expression from other part of the template:

```lang-java,fid-4f9eb89804144d7da51cde92e64dc34c
@(1 + 5) @// print out the result of 1 + 5

@// use ( ) to separate expression from other part of the template
@(foo.bar)_and_some_other_string 
```

#### Source of expression

As shown in the [above](#expression) example, the instances/classes/fields/methods inside an expression could come from anyone of the followings:

* [Template arguments](#argument)
* [Invoke another template](#invoke_template)
* [Template built-ins](template_built_ins.md)
* [Variable declared in scripting blocks](#scripting)
* Directly referenced classes
* Variable declared with `@assign()` directive
* Variable declared with `.assign()` extension to template call

#### See also 
* [Escape expression result](expression.md#escape)
* [Null safe expression](expression.md#null-safe) 
* [Transform expression output](expression.md#transformer)

### [scripting]Java code scripting

Rythm allows you to put any Java code inside a template:

```
@{
    String fullName = client.name.toUpperCase() + " " + client.forname;
}
<h1>Client @fullName</h1>
```

You can use `p()` function to print content to the current output inside scripting block:

```
@{
    String fullName = client.name.toUpperCase() + " " + client.forname;
    p("<h1>").p(fullName).p("</h1>");
}
```

But obviously this is not the way to go for most times.

### [flow-control]Template flow control

Rythm provides three flow control directive for template author to control the template rendering:

#### [brancing] Branching

```
@if (isMobile) {
    @// render mobile block
} else {
    @// render pc block
}
```

#### [loop] Looping

```
<ul>
@for(Product product: products) {
    <li>@product.name. Price: @product.price.format("## ###,00") €</li>
}
</ul>
```

#### [return] Stop rendering and return

```
@if (user.hasRole("guess")) {
    <p>You don't have permission to view this page</p>
    @return
}
@// continue rendering the secured page
```

For more about the flow control please refer to the directive reference:

* Branching - <a href="directive.md#if"><code>@if/else/elseif</code></a>
* Looping - <a href="directive.md#for"><code>@for</code></a>
* Return - <a href="directive.md#return"><code>@return</code></a>

### [invoke_template]Invoke template

The capability of invoking other template from the current template is an important feature that facilitate template reuse and makes template user's life easier. Many template engine provides tools to support template reuse, e.g. the `#include` and `#parse` of velocity, the `#include`, `#macro` and `#import` of freemarker, like those template engines, Rythm also provides tools to support template reuse, just in a more general and much easier way.

Suppose you have a template `foo.html` and another one named `bar.html` in the same directory. The following command allows you invoke `bar.html` inside `foo.html`:

```lang-java,fid-ca0397f1d9514836967b820b6b781cc0
@foo()
```

See following topics for more about template invocation:

* [Passing arguments when invoking a template](#inv_arg)
* [Handling paths](#inv_path)
* [Dynamic template invocation](#inv_dyna)
* [Passing body with template invocation](#inv_body)
    * [Pass body callback parameter spec to template call](#inv_body_callback)
* [Further processing template invocation result](#inv_process)
    * [Escape invocation result](#inv_esc)
    * [Cache invocation result](#inv_cache)
    * [Assign invocation result](#inv_assign)
    * [Chain the invocation decoration functions](#inv_chain)

#### [inv_arg] Passing arguments when invoking a template

As stated above most template has arguments. Thus when calling a template with arguments you need to pass arguments to them. Again the foo and bar case. Now suppose the bar template has an argument declaration:

```lang-java
@** the bar.html template **@
@args String whoCallMe, boolean sayBye
Hi @whoCallMe, this is inside bar.
@if (sayBye) {
Bye @whoCallMe
}
```

Now the foo template needs a little bit update:

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar("foo", true)
```

In the above example, we pass parameters to template by parameter declaration position. But Rythm also provides a more friendly parameter passing approach when you have many parameters

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar(whoCallMe: "foo", sayBye: true)
```

or even with the nice javascript style

```lang-java,fid-05fff88e36d24054807d6e5a5bc01477
@** the foo.html template **@
@bar({
    whoCallMe: "foo",
    sayBye: true
})
```

#### [inv_path] Handling paths

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
        <li>role.getName()</li>
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
renderBody(roles)
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

### [inline_tag] Inline tags

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