# Template Author's Guide

### [introduction]Introduction

Rythm Template Engine is a text generator that merges dynamic content into static template. 

![java-version](../img/hello_world.png)

In order to use Rythm to generate a text file, the essential work is to create template file which write down the static part and use Rythm syntax to define the dynamic parts. It is also important to understand that a template file has one or more arguments which is the origin of the dynamic content. The Java program pass parameters to template file via arguments to generate different result.

So a template file is a text file, some parts of which have placeholders for dynamically generated content. The template’s dynamic elements are written using the Java language.

Dynamic elements are resolved during template execution. The rendered result is then sent as part of the HTTP response.

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

The instances/classes/fields/methods inside an expression could come from anyone of the followings:

* [Template arguments](template_argument.md)
* [Template built-ins](template_builtins.md)
* [Variable declared in scripting blocks](scripting.md)
* Directly referenced classes as shown in the [above](expression) example


See also 
* [Escape expression result](expression.md#escape), 
* [Null safe expression](expression.md#null-safe), 
* [Transform expression output](expression.md#transformer)



### [invoke]Invoke template

