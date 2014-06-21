<h1 data-book="directive">Directive Reference</h1>

This chapter documents Rythm directives and their usage. 

Directive syntax

* Directives should start with `@`, followed by directive name and then followed by `()`, optionally followed by `{` and `}` block.

### [a]A: @args, @assign

* [@args](#args) - declare template arguments
* [@assign](#assign) - assign render part to a local variable

#### [args]@args

Declare template arguments and their types. 

Style 1: declare all arguments in one line:

```lang-java
@args String foo, int bar, Map<String, Object> myMap, ...
```

Style 2: declare arguments in separate lines:

```lang-java
@// the comma at the line end is optional
@args() {
    String foo
    int bar,
    Map<String, Object> myMap
    ...
}
```

**Note**

* There can be multiple `@args` statement in one template
* A template variable must be declared before using it
* You cannot declare the same template argument twice. By same it means the template variable name is the same.

#### [assign]@assign

assign a block of rendering result into one local variable.

```lang-java,fid-9ddf00d5be894bbea260173d37e62751
@// assign something into local variable foo
@assign(foo) {
foo's full content
and blah blah...
}
@// output foo content
@foo
@// use foo in other places
@for (foo.toString().split("\n")) {
@_index: @_
}
```

**Note**

* The local variable been assigned to is of type `Object`, which means
* The local variable is declared while assignment happen, meaning there cannot be other variable (including template argument) with the same name been declared
* You can't treat it as a `String`, and use String method directly
* Do not use quotation mark to enclose the variable name like `foo` in the sample

### [b]B: @break

#### [break]@break

Break out from within a loop

```lang-java,fid-c120be57889f4767bcce578edfb3c3a1
@for(int i : [1 .. 9]) {
	@if (i > 5) {@break}
    @i
}
```

**Note**

* You can omit the `()` after `@break` because `break` is a Java keyword

As writting a if condition to break is a common practice, Rythm enable you to do it in a simple way:

```lang-java
@for(int i : [1 .. 9]) {
	@breakIf(i > 5) @// the same effect as @if (i > 5) {@break}
    @i
}
```


### [c]C: @cache, code type ...

* [@cache()](#cache) - cache render part
* [code type](#code_type) - code type switch
* [comment](#comment) - inline or multi-line comments
* [@compact()](#compact) - force compact render part
* [@continue](#continue) - continue to next loop 

#### [cache]@cache

The `@cache()` directive caches render part:

```lang-java
@cache() {
<ul id="news-list">
@for(msg: news) {
<li>
    <div class="caption">@msg.caption</div>
    <div class="detail">@msg.detail</div>
</li>
}
</ul>
}
```

So the above code cache news list for the configured [cache timeout](configuration.md#default_cache_ttl) (seconds), which is 60*60 (1 hour) if you haven't configured the item.

You can pass in timeout to `@cache()` directive as parameter:

```lang-java
@// cache for 60 seconds
@cache(60) {
...
}

@// cache for 1 minute
@cache("1mn") {
...
}

@// cache for 2 hours
@cache("2h") {
...
}
```

When timeout parameter is specified in `int` type, it means in seconds. If value timeout value is specified in a String, then it's parsed by [duration parser](configuration.md#cache_duration_parser_impl) and converted into seconds. The [default duration parser](http://rythmengine.org/api/org/rythmengine/extension/IDurationParser.html#DEFAULT_PARSER) regonize the following timeout strings:

* `1d`: 1 day
* `3h`: 3 hours
* `8mn` or `8min`: 8 minutes
* `23s`: 23 seconds  

#### [code_type]Code type implicit directive

There is no explicit `@codeType()` directive. The code type context switch is done implicitly when [smart escape](configuration.md#feature_smart_escape_enabled) is enabled.

```lang-java
@// under default code type (html)
@content
<script type="text/javascript">
@// enter js code type implicitly
alert("@content");
</script>
@// exit js code type implicitly 
```

Right now Rythm support Javascript and CSS code type via `<script></script>` and `<style></style>` tag

#### [comment]Comment directive

Comment directive enable you to add comment to Rythm template source without output them in the render result:

```lang-java
@// a single line comment

@*
A multiple line
comment
*@

@**
 * And it can be format 
 * into good shape
 *@
```

#### [compact]@compact

Force compact a render part without regarding to the [compact mode configration](configuration.md#codegen_compact_enabled).

```lang-java,fid-17bfaba2d305430daa868ee5e234ac73
@compact(){
abc      ddd

1

3

4
}
```

#### [continue]@continue

Continue to the next loop iteration without executing the rest loop code

```lang-java,fid-09cb8ded1e2d419e97063be7ed077d4e
@for(int i : [1 .. 9]) {
	@if (i < 5) {@continue}
    @i
}
```

**Note**

* You can omit the `()` after `@break` because `break` is a Java keyword

Like [@break]#break directive, you can write the if condition inside the `@continue()` directive:

```lang-java
@for(int i : [1 .. 9]) {
	@continueIf(i < 5) @// the same effect as @if (i < 5) {@continue}
    @i
}
``` 

### [d]D: @debug, @def ...

* [@debug](#debug) - print out debug info from within template
* [@def](#def) - define inline tag
* [@doLayout](#doLayout) - alias of [@render](#render)

#### [debug]@debug

Print debug info from with template

```lang-java
@if (order.closed()) {
    @debug("The order[%s] is closed", order.id)
}
```

The above code will print out a log message to console if the order is closed.

#### [def]@def

Define a function (or you can call it inline tag) which can be called in the same template. There are 2 kind of functions can be defined with `@def`:

Define a function without return value:

```lang-java,fid-9ccdd3831c4645db8c9fba7232643ac7
@def sayHelloTo(String who) {
    Hello @who!
}

@sayHelloTo("Rythm")
@sayHelloTo("Velocity")
```

Define a function with return value:

```lang-java,fid-febc8a31c0a045e6a4a52a5d0d4b07e0
@def String absoluteUrl(String path){
    return "http://rythmengine.org/" + path;
}
@absoluteUrl("/doc/directive")
@absoluteUrl("/doc/index")
```

A interesting fact about `@def` with and without return value:

When there is no return type or return type is `void`, the source code inside the block is supposed to be template output. If you want to write Java code you need to enclose them inside the `@{` and `}` scripting block:

```lang-java
@def sayHello(String who) {
    @{
        // pure java code inside the @def without return type
        // needs to be put inside scripting block
        User user = User.current();
    }
    Hello @who (by @user.getFullName())
}
```

When return type is presented, the source code inside the block is treated as Java code, if you want to output something you need to write `pe(...)` of the template base method:

```lang-java,fid-d12d56d2eb1a430887c72545567ed2c8
@def boolean ok() {
	int i = new Random().nextInt(100);
    boolean ok = i % 2 == 0;
    if (ok) {
    	pe("success!");
    } else {
    	pe("fail!");
    }
    return ok;
}
@if (ok()) {
	nice :)
} else {
	OMG :(
}
```

<div class="alert alert-warn">
<i class="icon-warning-sign"></i>
invoking `pe(...)` to print out stuff inside a `@def()` function with return type is not encouraged, as it brings side effect and is not a good design pratice.
</div>

See also:

* [Comparison of code reuse techniques used in Rythm](#TODO)

#### [doLayout] @doLayout

This is an alias of [@render](#render).

### [e]E: @escape, @exec ...

* [@escape](#escape)
* [@exec](#exec)
* [@extends](#extends)

#### [escape]@escape

switch escape scheme for render part.

```lang-java,fid-4c9639275b6246bb945076f8b096d751
foo in default context: @foo
bar in default context: @bar
@escape("js") {
    foo in JS context: @foo
    bar in JS context: @bar
}
@escape("csv") {
    foo in CSV context: @foo
    bar in CSV context: @bar
}
```

Escape accept any Object type parameters that evaluated to the following strings (case insensitive):

1. `RAW` – do not escape. Same effect with [@raw()](#raw)
2. `CSV` - escape scheme set to csv
3. `HTML` - escape scheme set to html
4. `JS` | `JAVASCRIPT` - escape scheme set to javascript
5. `JSON` - escape scheme set to Java
6. `XML` - escape scheme set to XML

**Note**

* The parameter is case incensitive
* If the parameter evaluated to `null` or empty string or no parameter has been passed in, then Rythm will find out the escape scheme based on the current [code type context](developer_guide.md#rc_code_type).
* You cannot pass a literal parameter without quotation mark like `@escape(JS)`, in that case the engine will treat `JS` as a variable name. This new logic in 1.0-b9 is different from old implementation.

#### [exec]@exec

Execute a [macro](#macro).

```lang-java
@exec(terms_and_conditions)
```

**Note**

* A macro can be executed multiple times in a template
* You don't have to use `@exec` to execute a macro, rather, you can execute it by name:

```lang-java
@terms_and_conditions()
```

#### [extends]@extends

Indicate the parent template (ie. layout template) of this template.

```
@extends(myLayout)
```

You can pass parameters to the layout template in the extends statement.

Suppose your parent template is defined as:

```
@args String pageId, String theme
...
<body class="@pageId @theme">
...
@doLayout()
...
```

```
@extends(myLayout, pageId: "document", theme: "dark")
```

or pass parameter by position

```
@extends(myLayout, "document", "dark")
```

**Note**

* Rythm allow unlimited level of template inheritence

<i class="icon-magic"></i> Try `@extends` by yourself on [rythm fiddle](http://fiddle.rythmengine.org/#/editor/886606b3a7034088b991855bef8f89da)

### [f-g]F-G: @for, @get

* [@for](#for) - loop through collection/array
* [@get](#get) - fetch the named content defined by [set](#set) 

#### [for]@for

A powerful iteration tool with Java based syntax and a couple of useful enhancements. This section covers the following parts about `@for()` loop:

* Two for loop types
    * [type I](#for_type_i): primary `for` loop
    * [type II](#for_type_ii): enhanced `for` loop
* [Loop variables](#loop_variable)
* [Join loop output](#loop_join)
* [Loop variable type inference](#loop_type_inference)
* [More loop styles](#more_loop_styles)
* [Smart loop expression](#smart_loop_expression)

There are two type of `@for` loop:

<a id="for_type_i"></a>**Type I** - the primary loop:

```
<ul>
@for(int i = 0; i < products.size(); ++i) {
    <li><a href="/product/@product.getId()">@product.getName()</a></li>
}
</ul>
```

<a id="for_type_ii"></a>**Type II** - the enhanced loop

```
<ul>
@for (Product product: products) {
    <li><a href="/product/@product.getId()">@product.getName()</a></li>
} else {
    <div class="alert">No product found</div>
}
</ul>
```

You can omit loop variable type in [most cases](#loop_type_inference) when Rythm has type information on the iterable, meaning the `for` statement in the above sample could be simpled as:

```
@for (product: products) {
```

You can even omit the variable name in which case rythm use the implicit loop variable `_`:

```
<ul>
@for (products) {
    <li><a href="/product/@_.getId()">@_.getName()</a></li>
} else {
    <div class="alert">No product found</div>
}
</ul>
```

##### [loop_variable]Loop variables

There are several useful loop utility variables defined when you are using [type II](#for_type_ii) loop, the variables are prefixed with `<varname>_`, or `_` if you are using the implicit loop variable `_` 

* `<varname>_index`: the loop index, start from `1`
* `<varname>_isFirst`: `true` for the first loop
* `<varname>_isLast`: `true` for the last loop
* `<varname>_parity`: alternates between `odd` and `even`
* `<varname>_isOdd`: `true` for loop index is `odd` in sequence
* `<varname>_size`: size of the loop

```
<ul>
@for (Product product: products) {
    <li class="@product_parity">@product_index. @product</li>
}
</ul>
```

##### [loop_join]Join loop output

You can extend the both [type I](#for_type_i) and [type II](#for_type_ii) loop with a `.join()`:

```
@for (products).join(","){
{
    "name": "@_.getName()",
    "category": "@_.getCategory()",
    "price": @_.getPrice()
}
}
```

The above code join the loop output by `,` which makes it very easy to generate the JSON output. And actually you can omit the "`,`" in the `join()` as it is the default separator.

##### [loop_type_inference]Loop variable type inference 

In case Rythm has the type information of the iterable, the variable type can be omitted:

```
@args List<User> users
...
@for(user: users) {
    ...
}
```

In the above sample code, Rythm knows the `users` is of type `List<User>` and it infer the type of the loop variable `user` is `User`, so you don't need to declare the loop variable type. Rythm also support deduct the loop variable type from `Map` iterables

```
@args Map<String, Integer> myMap
...
@for(key: myMap.keySet()) {...} @// key type is String
@for(val: myMap.values()) {...} @// val type is Integer
```

Limit of loop variable type inference:

* The iterable must be declared with @args statement
* The iterable cannot be an expression, e.g. `@for(v: foo.bar()){...}`

<div class="alert">
<i class="icon-warning-sign"></i>
When you omit the loop variable type in case Rythm cannot get the iterable type info, Rythm will hook the loop variable to `Object` type.
</div>

##### [more_loop_styles]More loop styles

For [type II loop](#for_type_ii), besides the Java loop style, Rythm also support JavaScript and Scala style:

**classic Java style**

```
@for(s : list) {...}
```

**JavaScript style**

```
@for(s in list) {...}
```

**Scala style**

```
@for(s <- list) {...}
```

##### [smart_loop_expression]Smart loop expression

Rythm support several loop expression variations to save template author's typings

String spliting

```
@for("a, b, c").join(){@_} @// output a,b,c
@for("a : b:c").join(){@_} @// output a,b,c
@for("a; b;c").join(){@_} @// output a,b,c
@for("a - b - c").join(){@_} @// output a,b,c
@for("a_b_c").join(){@_} @// output a,b,c
```

Number ranges

```
@for (1 .. 5).join() {@_} @// output: 1,2,3,4
@for ([1 .. 5]).join() {@_} @// output: 1,2,3,4,5
```

<i class="icon-magic"></i> Try `@for` by yourself on [rythm fiddle](http://fiddle.rythmengine.org/#/editor/976408dec0074a8e8c1f7364eab04e20)

#### [get]@get (deprecated)

Retrieve template attribute which is [set](#set) previously and print it out.

```
@set("pageTitle": "orders")
...
@get("pageTitle")
```

Note the `@set` and `@get` is not to be used inside a single template because the above code could be simply replaced by

```
@{String pageTitle = "orders";}
...
@pageTitle
```

The initial purpose of `@set/@get` pair is to pass data from a sub template to parent/layout template:

Sub template:

```
@extends(layout)
@set(pageTitle: "orders")
```

Layout template:

```
<title>@get("pageTitle")</title> 
```

However there is an new approach to passing data from sub template to layout template:

Sub template:

```
extends(layout, pageTitle: "orders")
```

Layout template:

```
@args String pageTitle
<title>@pageTitle</title> 
```

See also

* [@set](#set)

### [i]I: @i18n, @if, @import ...

* [@i18n](#i18n) - internationalization a string
* [@if](#if) - if/else flow control
* [@import](#import) - declare packages to import
* [@include](#include) - include another template content in place
* [@inherited](#inherited) - render layout template default section content in place
* [@init](#init) - specify the code to be executed before rendering processs start
* [@invoke](#invoke) - call another template

#### [i18n]@i18n

Translate the supplied message code and optional parameters using [current locale](developer_guide.md#rc_locale):

```lang-java,fid-160dfb84d05e41aa905af701c1430e93
@args String x
// -- locale: default
@x: @i18n(x)

// -- locale: en
@locale("en"){
@x: @i18n(x)
}

// -- locale: zh_CN
@locale("zh_CN") {
@x: @i18n(x)
}
```

`@i18n()` support complex message code translation with parameters:

```lang-java,fid-24c567bdf2834f558effe55df9efbed8
// -- complex msg with params
@i18n('msg', "planet", 7, new Date())
// -- complex msg with params in Chinese
@locale("zh_CN") {
@i18n('msg', "planet", 7, new Date())
}

// -- relevant resource keys
@verbatim(){
x=foo
planet=Mars
msg=At {2,time,short} on {2,date,long}, we detected {1,number,integer} spaceships on the planet {0}.
}
```

**Note**

Literally you always use the [i18n transformer](builtin_transformer.md#i18n) to replace the `@i18n()` directive or vice versa. 

#### [if]@if/else/elseif

The branch flow control. It simply use the Java syntax:

```lang-java,fid-d2b7f39615bb4dd2a228775b37aafee3
@if(age<8) {
  @age < 8
} else if(age<16) {
  @age < 16
} else {
  @age >= 16
}
```

To make it easy and fun, Rythm support smart evaluation in the `if` statement:

```lang-java,fid-f63cce8f03694c0c8545087986c95ac3
@args String name, int money, List<String> names

---- smart evaluate: String ----
@if (name) {
  name is true
} else {
  name is false
}

---- smart evaluate: Number ----
@if (money) {
  money is non-zero
} else {
  money is zero
}

---- smart evalute: collections ---
@if(names) {
  names is not empty
} else {
  names is empty
}
```

Below is the smart evaluation rule table

<table class="table mono" style="font-size: 10pt">
<thead>
<tr>
<th>expression type</th>
<th>rule</th>
</tr>
</thead>
<tbody>
<tr>
<td>byte b</td>
<td>b != 0</td>
</tr>
<tr>
<td>char c</td>
<td>c != 0</td>
</tr>
<tr>
<td>boolean b</td>
<td>b</td>
</tr>
<tr>
<td>int n</td>
<td>n != 0</td>
</tr>
<tr>
<td>long l</td>
<td>l != 0L</td>
</tr>
<tr>
<td>float f</td>
<td>Math.abs(f) > 0.00000001</td>
</tr>
<tr>
<td>double d</td>
<td>Math.abs(d) > 0.00000001</td>
</tr>
<tr>
<td>String s</td>
<td>
<pre style="background:transparent;border:none;padding:0">
if (S.isEmpty(s)) return false;
if ("false".equalsIgnoreCase(s)) return false;
if ("no".equalsIgnoreCase(s)) return false;
return true;
</pre>
</td>
</tr>
<tr>
<td>Collection c</td>
<td>null != c && !c.isEmpty()</td>
</tr>
<tr>
<td>Map m</td>
<td>null != m && !m.isEmpty()</td>
</tr>
<tr>
<td>Byte b</td>
<td>null != b && eval(b.byteValue())</td>
</tr>
<tr>
<td>Boolean b</td>
<td>null != b && b</td>
</tr>
<tr>
<td>Character c</td>
<td>null != c && eval(c.charValue())</td>
</tr>
<tr>
<td>Float f</td>
<td>null != f && eval(f.floatValue())</td>
</tr>
<tr>
<td>Double d</td>
<td>null != d && eval(d.doubleValue())</td>
</tr>
<tr>
<td>Number n</td>
<td>null == n ? false : eval(n.intValue())</td>
</tr>
<tr>
<td>Object condition</td>
<td>
<pre style="background:transparent;border:none;padding:0;">
if (condition == null) {
    return false;
} else if (condition instanceof String) {
    return eval((String) condition);
} else if (condition instanceof Boolean) {
    return (Boolean) condition;
} else if (condition instanceof Collection) {
    return eval((Collection) condition);
} else if (condition instanceof Map) {
    return eval((Map) condition);
} else if (condition instanceof Double) {
    return eval((Double) condition);
} else if (condition instanceof Float) {
    return eval((Float) condition);
} else if (condition instanceof Long) {
    return eval((Long) condition);
} else if (condition.getClass().isArray()) {
    return Array.getLength(condition) > 0;
} else if (condition instanceof Number) {
    return eval((Number) condition);
}
return true;
</pre>
</td>
</tr>
</tbody>
</table>

#### [import]@import

Declare java package to be imported. This is exactly the same as `import` keyword in Java except a few enhancement:

* You can `@import` multiple package in the same line
* You can use `@import` in any place of your template

Sample 1: import a package and use class in it without full qualification

```lang-java
@import java.text.*

@//now you can use the imported package content without full qualification
@{
    Date today = new Date();
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
    String formatted = df.format(today);
}
Today is @formatted
```

<div class="alert alert-info">
You don't usually do format in above way. Rythm provides built-in transformers to make it much easier, just write something like:
<code>@(new Date().format("yyyy-MM-dd"))</code> is okay
</div>

Sample 2: import a class methods/fields statically and use them without full qualification

```lang-java
@import static java.util.Locale.*
@locale(GERMAN){
...
}
```

Sample 3: import multiple packages in the same line:

```lang-java
@import java.text.*, java.util.regex.*
```

When you have many packages that is not easy to read in one line, you can use the second style of `@import` directive and organize them into multiple lines:

```lang-java
@import(){
    java.text.*
    java.util.regex.*
    static java.util.Locale.*
}
```

Or mix the both styles together:

```lang-java
@import() {
    java.text.*,java.util.regex.*
    static java.util.Locale.*
}
```

##### [implicit_import]Implicit Import

Rythm will import the following packages implicity even you haven't declared them with `@import` directive:

* java.util.*
* java.io.* (not imported when running in sandbox mode)
* org.rythmengine.template.TemplateBase


#### [include]@include

Include content from another template in place. If you have a certain part of template code that can be used in multiple templates, then you can separate them into a single template source file, and then use `@include` directive to load the content of that file in place of the current template. The process happen at parsing time, it' mostly like a `@exec` directive which load a macro defined in the same template file. Samples:

Suppose you have defined a template `dialog`:

```lang-html
<div id="dia-global" class="modal hide fade">
    <div class="modal-header">
        <h1 data-bind="text: title"></h1>
    </div>
    <div class="modal-body" data-bind="html: body">
    </div>
    <div class="modal-footer">
        <a class="btn btn-primary" data-bind="click: callback(), text: btnText"></a>
        &nbsp;&nbsp;
        <a class="btn close" data-dismiss="modal" data-bind="text: btn2Text(), visible: btn2Text()"></a>
    </div>
</div>
```

You can include the template in any other templates using `@include()`

```lang-java
@include(dialog)
```

See [Locate template(TBD)](template_guide.md#locate) to understand how Rythm locate `dialog` template path.

#### [inherited]@inherited

Alias of [@renderInherited](#renderInherited).

Include layout template section default content in place. This directive can only be used within [@section() {}](#section) context.

Suppose the layout template defines a section footer with default content:

```lang-html
<div id="footer">
@render(footer) {
    copyright(c) 2001 XYZ co ltd.
}
</div>
```

In the child template you can output the default content along with specific footer:

```lang-java
@section(footer) {
    The specific page of XYZ @inherited
}
```

And it renders the following output:

```lang-html
<div id="footer">
The specific page of XYZ
copyright(c) 2001 XYZ co ltd.
</div>
```

#### [init]@init

Specify a code segment to run before executing template build method. This is mainly to setup certain state which is only evaluated at runtime but must be setup before any render output issued. The following sample comes from [rythm website](http://rythmengine.org/doc/feature)

```lang-java
@init() {
    if (page.startsWith("feature")) super.__setRenderArg("curPage", "feature");
    else super.__setRenderArg("curPage", "doc");
}
```

So the above code says, if the page variable (String type) starts with "`feature`" then set the template argument "`curPage`" to "`feature`" in the layout template which is specified with "`super`", otherwise set "`curPage`" to "`doc`". this impact the top menu bar's highlight part. The status is only evaluated at runtime when rythm get the value of "`page`" template argument; and it must be computed before building the output of the template.

<div class="alert alert-info">
    The code inside the <code>@init(){}</code> block is Java code, not template outputs.
</div>

#### [invoke]@invoke

Invoke/call another template. Like [@include](#include), `@invoke` reuse another external template source file, however `@invoke` is more powerful:

* [passing parameters to another template](#invoke_params) 
    * [implicit parameters](#invoke_params_implicit)
* [passing body](#invoke_body) 
    * [body with arguments](#invoke_body_argument) 
* [further processing the invocation result](#invoke_result_decoration) 
    * [escape invocation result](#invoke_escape) 
    * [cache invocation result](#invoke_cache) 
    * [assign invocation result to a variable](#invoke_assign) 
* [chain different invocation result processing together](#invoke_chain)
* [dynamic invocation](#invoke_dynamic)

You don't need to use `@invoke()` to call a template, instead you use the template name directly. So the following two styles of template invocation call are exactly the same effects:

Style 1: call template with `@invoke` directive

```lang-java
@invoke("foo", 1, 2, 3)
``` 

Style 2: call template with template name

```lang-java
@foo(1, 2, 3)
```

##### [invoke_params]passing parameters to another template

`@invoke` process happens at runtime, which allows you to pass runtime parameters to the template being called. You can pass parameters by position or by name as shown below:

```lang-java
@hello("Rythm") @// by position
@hello(who: "Rythm") @// by argument name
@foo("param1", 2, 3.0) @// by position
@foo(p1: "param1", p2: 2, p3: 3.0) @// by name
```

You can even use the JS style:

```lang-java
@foo({
    p1: "param1",
    p2: 2,
    p3: 3.0
})
```

##### [invoke_params_implicit]Implicit parameters

Rythm automatically pass the caller template's argument values to callee template if they were the same name and same type. Thus you don't need to explicitly pass parameters when invoking the callee template from caller template if they share the same arguments. Click "Try " to view the sample in rythm fiddle to understand this concept

```lang-java
@args String who
@foo()
```

##### [invoke_body]Passing body

You can pass in a render part enclosed with `{ }` pair when calling a template:

```lang-html
<form>
@fieldSet("order form"){
    <select name="product.id">...</select>
    <input type="text" name="customer.name"></input>
    ...
}
</form>
```

So in the above template call, `@fieldSet` is another template which accept a body when you call it. And the template might looks like:

```lang-html
@args String label
<fieldset>
<legend>@label</legend>
@// use @renderBody() to output the passed in body (the form)
@renderBody() 
</fieldset>
```

##### [invoke_body_argument] body with arguments

You can declare arguments in the body to be passed to another template and let that template to call the body with parameters. 

The code to call another template with body with arguments

```lang-html
@lookupRole(permission: "superuser").callback(List<Role> roleList) {
    <ul>superusers
    @for(Role role: roleList) {
        <li>role.getName()</li>
    }
    </ul>
}
```

The `lookupRole.html` template:

```lang-html
@args String permission
　
@{
    List<Role> roles = Role.find("permission", permission).asList()
}
renderBody(roles)
```

##### [invoke_escape] Escape invocation result

The result of template invocation is output as raw data. You can dictate rythm to escape a template invocation result if needed:

```lang-java
@foo().escapeJSON()
@foo().escape()
@foo().escape("CSV")
```

##### [invoke_cache] Cache invocation result

You can cache a template invocation result by `.cache()` extension:

```lang-java
@foo(1, 2, "3").cache()  @// cache using default TTL
@foo(1, 2, "3").cache("1h") @// cache for 1 hour
@foo(1, 2, "3").cache(60 * 60) @// cache for 1 hour
```

The above statement invoke template `foo` using parameter [1, 2, “3”] and cache the result for one hour. Within the next one hour, the template will not be invoked, instead the cached result will be returned if the parameter passed in are still [1, 2, “3”].

So you see Rythm is smart enough to cache invocation against template name and the parameter passed in. If the template invocation has body passed in, the body will also be taken into consideration when calculating the cache key.

In some cases where the tag invocation result is not merely a function of the template name, parameter and body, but also some implicit variables, where you might expect different result even you have completely the same signature of template invocation. Rythm provide way for you to take those additional implicit variables into account when calculating the cache key:

```lang-java
@{User user = User.current()}
@foo(1, 2, "3").cache("1h", user.isAdmin())
```

The above statement shows how to pass additional parameter to cache the template invocation.

##### [invoke_assign] Assign invocation result to a variable

In case you want to refer the invocation result in more than one places in the current template, you can assign the invocation result to a local variable (which shall not be the same name with any template argument or other local variables), and use `@` expression to refer to the variable instead of invoking the template multiple times:

```lang-java
@countrySelect().assign("selCountries")
...
@selCountries @// output country select here
...
@selCountries @// output country select there
```

##### [invoke_chain] Chain different invocation processing together

You can chain different invocation processings together:

```lang-java
@foo().escapeJS().cache("1h").assign("xResult").callback(String name) {
    alert('@name')
}
``` 

##### [invoke_dynamic] dynamic invocation

Dynamic invocation means the template to be invoked is determined by a runtime variable, in this case, you must use the `@invoke` directive:

```lang-java
@args String platform @// could be pc, iphone, ipad ...
...
@invoke("designer." + platform)
```

So when platform is iphone, the above case has the same effect as calling `@designer.iphone()`.

Usually when a template been invoked cannot be found, Rythm will report an error. Sometimes it is expected that certain template does not exists, in which case one can use `.ignoreNonExistsTag()` extension with invoke keyword:

```lang-java
@invoke("designer." + platform).ignoreNonExistsTag()
```

### [LMN]L-N: @locale, @macro ...

* [@locale](#locale) - set locale for a render part
* [@macro](#macro) - define macro
* [@nocompact](#nocompact) - specify not the a render part that whitespace shall not be compact

#### [locale] @locale

locale setting impact the output of `@format()` transformer and `i18n()` directive/transformer. Usually the locale of a template is set before rendering process started. But you can use `@locale()` to set the locale for a specific template part if the page is designed to display multiple languages:

```lang-java
@locale("zh-cn") {
    ...
}
```

#### [macro] @macro

Use `@macro` to define a macro which can be executed with `@exec()` or invoke directly by name:

```lang-java
@macro("terms_and_conditions") {
<h1>Terms and Conditions</h1>
...
}
```

**Note**, unlike inline function defined by `@def` or normal template, macro doesn't accept parameters.  

#### [nocompact] @nocompact

Specify not to compact a render part without regard to the current compact setting:

```lang-java
@nocompact(){
<pre>
-----------------
@title
------------------
Name: @name
Email: @email
</pre>
}
```

### [R]R: @raw, @render ...

* [@raw](#raw) - output raw content
* [@render](#render) - render layout content
* [@renderBody](#renderBody) - render tag enclosing body
* [@renderInherited](#renderInherited) - render layout template default section content


#### @raw

Specify a render part that expression out shall not be escaped:

```lang-java
@args Component component
<div id="@component.id" class="@component.class">
@raw() {
    @component.body
}
</div>
```

#### [render]@render

Used in layout(parent) template to output sub template main content or sections:

```lang-html
<html>
<head>
...
</head>
<body>

<div id="header">
@// output the "header" section
@render(header) {
    default header 
}
</div>

<div id="main">
@render() @// output sub template content that are not in named sections
</div>

<div id="footer">
@// output the "footer section
@render(footer) {
    default footer
}
</div>
</body>
</html>
```

#### [renderBody] @renderBody

Render the calling template enclosing body. See [here](#invoke_body)

#### [renderInherited] @renderInherited

Alias of [@inherited](#inherited)

#### [return] @return

Stop the template execution immediately:

```lang-html
@args User user
@extends(main, "Order Manager")
@if (!user.is("order-manager")) {
    <div class="alert">
        You don't have the right to access this page
    </div>
    @return
}
```

Note you don't have to type `()` after `@return` because `return` is a Java reserved word.

#### [return-if] @returnIf

In simple case you can pass an expression to `@return()` to save the `@if` statement:

```lang-java
@returnIf(!user.is("order-manager"))
@// the above is exactly the same as:
@if(!user.is("order-manager")) {
    @return
}
```

### [s]S: scripting, @section ...

* [scripting in template](#scripting)
* [define layout section](#section)
* [set layout variable value](#set)

#### [scripting] Scripting in template

You can write any java code in template source with scripting block:

```lang-java,fid-c8e5e5f75bb248339fbf54e512abe635
@{
    String s = "foo";
    int l = s.length();
    ...
}@
```

You can actually save the last `@` of the scripting block:

```lang-java,fid-721ee87739164d8095437e0f52229c61
@{
    // write any java source code here
}
```

#### [section]Define layout section

If you have different sections defined by <a href="#render"><code>@render(<section-name>)</code></a> in the layout template, you can define the content of the sections in the sub template via `@section` directive. Corresponding to the layout template defined in the sample of [@render](#render), here is the sub template:

```lang-html
@args List<Order> orders
@extends(layout)
@section(header) {
    <h1>Order Manager</h1>
}
@for (orders) {
    <div class="order">
        ...
    </div>
}
```

#### [set] Set template property value (deprecated)

Pair with [@get](#get) `@set` directive is used to pass value from a sub template to layout template. However it is deprecated now. See more on [@get](#get)

### [tv]T-V: @ts, @verbatim

* [@ts](#ts) output timestamp
* [@verbatim](#verbatim) specify part that shall be output literally without interpretation

#### [ts]@ts()

Output current timestamp:

```lang-java,fid-b628d73e4e2f41baa71f2c8762c20ec0
@ts()
```

#### [verbatim]@verbatim

Specify part of the template that shall be output literally:

```lang-java,fid-ca0cc8800e4c4766b23edb42c1d1ac4a
A sample rythm template
@verbatim() {
<pre><code>
    @args String who
    Hello @who!
</code></pre>
}
```

### See also

* [template user's guide](template_guide.md)
* [developer's guide](developer_guide.md)
