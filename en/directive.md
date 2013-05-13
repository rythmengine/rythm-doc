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

### [d]D: debug, def ...

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

### [e]E: escape, exec, expand

* [@escape](#escape)
* [@exec](#exec)
* [@expand](#expand)

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

if the parameter evaluated to `null` or empty string or no parameter has been passed in, then Rythm will find out the escape scheme based on the current [code type context](developer_guide.md#rc_code_type).
