<h1 data-book="directive">Directive Reference</h1>

This chapter documents Rythm directives and their usage. 

Directive syntax

* Directives should start with `@`, followed by directive name and then followed by `()`, optionally followed by `{` and `}` block.

### [a]A: args, assign

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

### [b]B: break

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

### [c]C: cache, code type ...

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

### [d]D: debug, def

* [@debug](#debug) - print out debug info from within template
* [@def](#def) - define inline tag

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



### [e]E: escape, exec ...

* [@escape](#escape)
* [@exec](#exec)
* [@expand](#expand)
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

A macro can be executed multiple times in a template

Aliases:

* [@expand](#expand)

#### [expand]@expand

Alias of [@exec](#exec).

```lang-java
@expand(terms_and_conditions)
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

### [f]F: for

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