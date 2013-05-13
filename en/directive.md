<h1 data-book="directive">Directive Reference</h1>

This chapter documents Rythm directives and their usage.

### [a]A (args, assign)

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

### [b]B (break)

#### [break]@break

Break out from within a loop

```lang-java,fid-c120be57889f4767bcce578edfb3c3a1
@for(int i : [1 .. 9]) {
	@if (i > 5) {@break}
    @i
}
```

**Note**

* You can ignore the `()` after `@break` because `break` is a Java keyword

### [c]C (cache, code type ...)

* [cache](#cache) - cache render part
* [code type](#code_type) - code type switch
* [comment](#comment) - inline or multi-line comments
* [compact](#compact) - force compact render part
* [continue](#continue) - continue to next loop 

#### [cache]Cache

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
@// 
@content

```
