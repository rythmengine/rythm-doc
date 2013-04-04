# Builtin Transformers

Rythm provides a set of built-in [transformers](expression.md#transformer) to easy your work

<div class="alert alert-info">
<b>Note</b>, unless specified, all transformers apply to <code>Object</code> type. In other words, you are use them to transform variable of any type, not only <code>String</code>s. 
</div>

If a variable been transformed is not a <code>String</code>, then it will be converted to <code>String</code> via [S.str(Object)](http://rythmengine.org/api/com/greenlaw110/rythm/utils/S.html#str(java.lang.Object)) call 

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

#### escapeHTML()

Alias:

* `escapeHtml()`
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

#### escapeXML()

Alias:

* `escapeXml()`
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

#### escapeJS()

Alias:

* `escapeJavaScript()`
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

#### escapeJSON()

Alias:

* `escapeJson()`
* `escape("json")`

Emit expression using the [json](http://rythmengine.org/api/com/greenlaw110/rythm/utils/Escape.html#JSON) escape scheme.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-2c94b9f4c22148b6974dcffbc4bf2214
@args Bar bar
@raw() {
@bar
@bar.escapeJSON()
@bar.escapeJson()
@bar.escape("json")
}
```

#### escapeCSV()

Alias:

* `escapeCsv()`
* `escape("csv")`

Emit expression using the [csv](http://rythmengine.org/api/com/greenlaw110/rythm/utils/Escape.html#CSV) escape scheme.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-51c053ac57644ba9862bae03ee69c2ae
@args Bar bar
@raw() {
@bar
@bar.escapeCSV()
@bar.escapeCsv()
@bar.escape("csv")
}
```

### String operations

Use string operation transformers to manupulate the output string.

#### capitalizeWords()

Captialize the first character of each words. Words are separated by non digits-alphabetic characters.

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-d0eaed51cdb044e3813843845453a819
@args Bar bar, String s
@bar
@bar.capitalizeWords()
------------
@s
@s.capitalizeWords()
```

#### noAccents()

Replace accent character (usually found in European languages) with corresponding non-accent character

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-beacda9f4c7945b7bd743e3da8b4cd7f
@args Bar bar
@bar
@bar.noAccents()
```

#### lowerFirst() and capFirst()

Make the first letter of the object to be lowercase or uppercase

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-ae7162075ef5498eb8af84beb8f77b02
@args Bar bar, String s
@bar
@bar.lowerFirst()
---
@s
@s.capFirst()
```

#### camelCase()

Change each word from underscore style to camel case style

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-baaa389eae37422b8a48508bd0889fe9
@args Bar bar
@bar
@bar.camelCase()
```

### Misc utilities

#### nl2br()

Change line break to html `<br/>` element

##### <i class="icon-magic"></i> Try yourself

```lang-java,fid-e0497048c78248129c13d7b69aa14d52
@args Bar bar
@bar
@bar.nl2br()
```
