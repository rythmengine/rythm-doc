<h1 data-book="directive">Directive Reference</h1>

This chapter documents Rythm directives and their usage.

### [args]@args

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

**Notes**

* There can be multiple `@args` statement in one template
* A template variable must be declared before using it
* You cannot declare the same template argument twice. By same it means the template variable name is the same.

### [assign]@assign

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

**Notes**

* The local variable been assigned to is of type `Object`, which means
* You can't treat it as a `String`, and use String method directly
* Do not use quotation mark to enclose the variable name like `foo` in the sample

### [break]@break

Break out from within a loop

```lang-java,fid-c120be57889f4767bcce578edfb3c3a1
@for(int i : [1 .. 9]) {
	@if (i > 5) {@break}
    @i
}
```

**Note**

* You can ignore the `()` after `@break` because `break` is a Java keyword

