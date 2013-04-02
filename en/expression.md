# More about expression

This section is about additional features of expression emission including:

* [Escaping](#escape)
* [Null safe expression](#null-safe)
* [Transformer](#transformer) 

### [escape]Escaping

Expression result is escaped before output depending on the current context to avoid injection attack. For example

```lang-html
<input value="@name">
```

Suppose `name` value is "`" onload="alert('Hack')`", if the output of `@name` is not escaped, the result will be:

```lang-html
<input value="" onload="alert('Hack')">
``` 

Which is definitely not the template author wanted. If the expression output is escaped using `html` scheme, then the result will be:

```lang-html
<input value="&quot; onload=&quot;alert(&apos;Hack&apos;)">
``` 

Not in good look but avoid the html injection attack. Escape also solves a common problem of processing user input which contains special characters, e.g. for template

```lang-html
<script>
alert("@product.name")
</script> 
```

If the product name contains "`"`", then it will cause the javascript error at runtime. If the output is properly escaped, the issue will be addressed.

Not all template engine on the market address the expression escape issue properly, some of them even don't have a default escape scheme and require the template author to manually escape the expression. 

Rythm, on the contrary, provides the state-of-art support on expression escape. Take a look at this example:

```lang-html,fid-974a2340d5004d0ba8a38e0fe646edb8
<html>
...
<p>@description</p>
...
<script>
alert("@description")
</script>
```

Suppose `description` is "`<h1>abc"xyz</h1>`", Rythm will generate:

```lang-html
<html>
...
<p>&lt;h1&gt;abc&quot;xyz&lt;/h1&gt;</p>
...
<script>
alert("<h1>abc\"xyz<\/h1>")
</script>
```

The above example shows clearly that Rythm use different escape scheme in different context (html and javascript). So here is what Rythm did, the initial escape context is set to `html`, when Rythm encountered "`<script>`" it switch the escape context to `Javascript` until it reaches "`</script>`" and then switch back to `html`.

#### [escape-schemes]Escape schemes

Rythm support the following escape schemes:

* html/xml
* javascript/js
* json
* csv

Usually template author does not need to explicitly specify the escape scheme because Rythm automatically switch escape scheme based on the parsing context as shown in the above example. However if in certain case it does require to set escape scheme explicitly, here is how to do it:

```lang-java,fid-18153c67362647489a03c46ad271634f
@args String s

// escape with html scheme
@s.escapeHtml()
@s.escapeHTML()
@s.escape("html")
@s().escapeHtml(s)

// escape with xml scheme
@s.escapeXml()
@s.escapeXML()
@s.escape("xml")

// escape with javascript scheme
@s.escapeJS()
@s.escapeJavaScript()
@s.escape("javascript")
@s.escape("js")
@s.escape("JS")

// escape with JSON scheme
@s.escapeJSON()
@s.escapeJson()
@s.escape("json")

// escape with CSV scheme
@s.escapeCSV()
@s.escapeCsv()
@s.escape("csv")
``` 

See also [Set initial code type](#set-init-code-type) 

<div class="alert alert-info">
    <i class="icon-info-sign"></i>
    You can force Rythm to use a specific scheme to escape an expression by using the escape 
    <a href="transformer">transformer</a>
</div>

### [null-safe]Null safe expression

When output an expression a common concern is how to deal with `null` values. It is not unusual that we expect it output empty string `""` when `null` value is expected. Which might result in verbose code for a simple expression like `@foo.bar.zee`:

```
@if(null != foo && null != bar) {@foo.bar.zee}
```

Fortunately, Rythm provides a feature called "null safe expression", which allows you to use `?` to create null safe expression. And the above code could be simplified as:

```lang-html,fid-a48393ff8f8f4603924d9d53313a5d10
@foo?.bar?.id
```

The above expression will not throw out `NullPointerException` if `foo` or `foo.bar` is `null`.

<div class="alert alert-error"><i class="icon-warning-sign"></i> You cannot use null safe expression in parameters like <code>@foo?.bar(@x?.y)</code></div>

#### [elvis]Elvis expression

In some special case, template author might want to output default value if `null` is found in an expression, and Rythm provides "elvis" operator in expression to handle that case:

```,fid-723147e71ff54033ac44568df2847a29
@(foo ?: "not present")
```  

The above code will output "not present" if variable `foo` is `null`.

<div class="alert alert-error"><i class="icon-warning-sign"></i> You cannot mix use null safe expression and elvis operation like <code>@(foo?.bar ?: "not present")</code></div>

### [transformer]Transformer

Sometimes it needs some kind of transform operations to precess expression output. A typical example is to format a `Date` typed variable:

```lang-java,fid-35de9612490d4415a72117414a94de63
@dueDate.format("yyyy-MM-dd")
```

Here the `format` is called transformer, which accept a Date type object and format(transform) it using specified format pattern. 

One or more Transformer could be applied to an expression to provide further processing. For example

```lang-java,fid-5378a60f92bc49019d7ef3353843a5fe
@theString.capFirst().escapeJavaScript()
```

### See also

* [Template author's guide](template_guide.md#expression)
* [Built-in transformers](builtin_transformer.md)
* [How to define your transformer](user_defined_transformer.md)
* [Disable transformer feature](configuration.md#transformer_feature)
* [Disable built-in transformer](configuration.md#builtin_transformer)
