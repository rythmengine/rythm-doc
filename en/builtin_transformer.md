# Builtin Transformers

<div class="alert alert-info">
Checkout transformer at &lt;<a href="expression.md#transformer">here</a>&gt; if you don't know what is a transformer
</div>

Rythm provides a set of built-in transformers to easy your work

### [escape]Escaping

Use escaping transformers when needed. (Usually you don't need to as Rythm is intelligent enough to escape your emissions with property escape scheme. Check it out at [here](expression#escape)).

#### escape()

Emit expression using the [**current**](template_context.md#escape) escape scheme. 

<div class="alert alert-info">
Since Rythm always escape expression output, meaning <code>@foo.escape()</code> has exactly the same effect of <code>@foo</code>.
</div>

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-70bccea25d8942b1b87081534d49aa3a
@args Bar bar
@bar.raw()
@bar.escape()
@bar
```

#### escapeHtml()

Alias:

* `escapeHTML()`
* `escape("html")`

Emit expression using the [html](http://rythmengine.org/api/com/greenlaw110/rythm/utils/Escape.html#HTML) escape scheme.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-80a06b8dcfc745ae88d5772b22a56b7a
@args Bar bar
@raw() {
@bar
@bar.escapeHtml()
@bar.escapeHTML()
@bar.escape("html")
}
```

#### escapeXml()

Alias:

* `escapeXML()`
* `escape("xml")`

Emit expression using the [xml](http://rythmengine.org/api/com/greenlaw110/rythm/utils/Escape.html#XML) escape scheme.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-c22f5795355a4e0faaccc3b890026843
@args Bar bar
@raw() {
@bar
@bar.escapeXml()
@bar.escapeXML()
@bar.escape("xml")
}
```

#### escapeJavaScript()

Alias:

* `escapeJS()`
* `escape("js")`

Emit expression using the [js](http://rythmengine.org/api/com/greenlaw110/rythm/utils/Escape.html#JS) escape scheme.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-ec3ccf28950f4b158cde2336632d9429
@args Bar bar
@raw() {
@bar
@bar.escapeJS()
@bar.escapeJavaScript()
@bar.escape("js")
}
```



