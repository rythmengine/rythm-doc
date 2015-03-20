<h1 data-book="builtin_transformer">内置转换器</h1>

Rythm提供了一整套内置转换器让你的工作变得轻松精致

<div class="alert alert-info">
<b>注意</b>，除非特别说明，所有的转换器都接受<code>Object</code>类型的参数。换句话说，你可以将转换器用于任何类型的对象，而不仅仅是<code>String</code>类型。这样比较爽吧 ;-)
</div>

通常来讲如果一个需要接受字串类型的转换器碰到了其他类型的对象，该对象将被先转换为String</code>然后在接受转换器的处理

### [escape0] 转义

转义是模板输出中常用的一个功能。不过通常来讲你都不需要跟转义转换器打交道，因为Rythm会智能地判断当前上下文，选择合适的转义方法。详情参见[这里](expression#escape)).

#### [escape] escape()

用[**当前**](template_context.md#escape)上下文的缺省方法来转义输出 

<div class="alert alert-info">
缺省情况下Rythm总是转义表达式输出，因此<code>@foo.escape()</code>和<code>@foo</code>在通常情况下会输出一样的结果
</div>

```lang-java,fid-70bccea25d8942b1b87081534d49aa3a
@args Bar bar
@bar.raw()
@bar.escape()
@bar
```

#### [escapeHTML] escapeHTML()

别名：

* `escapeHtml()`
* `escape("html")`

用[html](http://rythmengine.org/api/org/rythmengine/utils/Escape.html#HTML)方式来转义输出

```lang-java,fid-80a06b8dcfc745ae88d5772b22a56b7a
@args Bar bar
@raw() {
@bar
@bar.escapeHtml()
@bar.escapeHTML()
@bar.escape("html")
}
```

#### [escapeXML]escapeXML()

别名：

* `escapeXml()`
* `escape("xml")`

用[xml](http://rythmengine.org/api/org/rythmengine/utils/Escape.html#XML)来转义输出

```lang-java,fid-c22f5795355a4e0faaccc3b890026843
@args Bar bar
@raw() {
@bar
@bar.escapeXml()
@bar.escapeXML()
@bar.escape("xml")
}
```

#### [escapeJS]escapeJS()

别名

* `escapeJavaScript()`
* `escape("js")`

用[JavaScript](http://rythmengine.org/api/org/rythmengine/utils/Escape.html#JS)方式来转义输出

```lang-java,fid-ec3ccf28950f4b158cde2336632d9429
@args Bar bar
@raw() {
@bar
@bar.escapeJS()
@bar.escapeJavaScript()
@bar.escape("js")
}
```

#### [escapeJSON]escapeJSON()

别名

* `escapeJson()`
* `escape("json")`

用[json](http://rythmengine.org/api/org/rythmengine/utils/Escape.html#JSON)方式来转义输出

```lang-java,fid-2c94b9f4c22148b6974dcffbc4bf2214
@args Bar bar
@raw() {
@bar
@bar.escapeJSON()
@bar.escapeJson()
@bar.escape("json")
}
```

#### [escapeCSV]escapeCSV()

别名

* `escapeCsv()`
* `escape("csv")`

用[csv](http://rythmengine.org/api/org/rythmengine/utils/Escape.html#CSV)方式来转义输出

```lang-java,fid-51c053ac57644ba9862bae03ee69c2ae
@args Bar bar
@raw() {
@bar
@bar.escapeCSV()
@bar.escapeCsv()
@bar.escape("csv")
}
```

### [format] 格式化

#### [format_number] 格式化数字

* `format()`
* `format(String pattern)`
* `format(String pattern, Locale locale)`

使用Pattern和Locale来格式化数字。**注意** 该转换器只适用于[数字](http://docs.oracle.com/javase/6/docs/api/java/lang/Number.html)类型变量。

如果没有指定Locale，Rythm会调用[I18N.locale()](http://rythmengine.org/api/org/rythmengine/utils/I18N.html#locale())来获取该值。在指定Pattern的情况下，Locale用来获取[DecimalFormatSymbols](http://docs.oracle.com/javase/6/docs/api/java/text/DecimalFormatSymbols.html)，并用之于构造一个[DecimalFormat](http://docs.oracle.com/javase/6/docs/api/java/text/DecimalFormat.html)对象；在没有Pattern的情况下Locale用来获取一个 [NumberFormat](http://docs.oracle.com/javase/6/docs/api/java/text/NumberFormat.html)对象。

```lang-java,fid-9c306f912d4842228bb45a928e89a593
@args Number x, Number y, Number z
[@x]: @x.format()
---
[@y]: @y.format("###,000,000.00")
---
[@z]: @z.format("###,000,000.0000", Locale.SIMPLIFIED_CHINESE)
```

#### [format_date] 格式化时间日期

* `format()`
* `format(String pattern)`
* `format(String pattern, Locale locale)`
* `format(String pattern, Locale locale, String timezone)`

用pattern, locale和timezone串来格式化时间日期变量。**Note** 该格式化工具仅适用于[java.util.Date](http://docs.oracle.com/javase/6/docs/api/java/util/Date.html)或者[org.joda.time.DateTime]()类型的变量.

如果没有指定Locale，Rythm会调用[I18N.locale()](http://rythmengine.org/api/org/rythmengine/utils/I18N.html#locale())来获取该值。在pattern指定的情况下Locale用来构造一个[SimpleDateFormat](http://docs.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html)来格式化日期; 如果没有指定pattern，Rythm用[DateFormat.getDateInstance(DateFormat.Default, locale)](http://docs.oracle.com/javase/6/docs/api/java/text/DateFormat.html#getDateInstance())来构造格式器。如果timezone指定了，该值会用于设定日期格式

```lang-java,fid-a1f92368a9a54283af097da64d48676b
@args Date x = new Date()
[@x]: @x.format()
---
[@x]: @x.format("yyyy-MM-dd")
---
[@x]: @x.format(null, Locale.SIMPLIFIED_CHINESE)
---
[@x]: @x.format(null, Locale.US, "GMT")
```

#### [format_currency] 格式化货币

* `formatCurrency()`
* `formatCurrency(String currencyCode)`
* `formatCurrency(String currencyCode, Locale locale)`

使用货币代码和Locale来格式化货币。和数字转换器`format()`不同, `formatCurrency`不需要变量必须是数字类型。对于非数字类型的变量，Rythm会首先将其转换为字串，然后调用`Double.parseDouble`来获取数字值。**注意**：`formatCurrency`不是空值安全的，你需要使用[空值安全表达法](expression.md#null-safe)如果你希望有空值安全保证。

```lang-java,fid-4af27082ad6d43f3be3e9d81876ea1d3
@args Object o, Number n
[@o]: @o?.formatCurrency()
----
[@n]: @n?.formatCurrency()
```

##### 参考资料

* [http://docs.oracle.com/javase/6/docs/api/java/util/Currency.html](http://docs.oracle.com/javase/6/docs/api/java/util/Currency.html)
* [http://docs.oracle.com/javase/6/docs/api/java/text/NumberFormat.html#getCurrency()](http://docs.oracle.com/javase/6/docs/api/java/text/NumberFormat.html#getCurrency())

### [string] 字串处理

对输出字串做各种处理

#### [capitalizeWords] capitalizeWords()

将字串中的每个字的首字母变成大写。这里字是由非数字字母符号分割的

```lang-java,fid-d0eaed51cdb044e3813843845453a819
@args Bar bar, String s
@bar
@bar.capitalizeWords()
------------
@s
@s.capitalizeWords()
```

#### [noAccents] noAccents()

该转换器通常用于处理欧洲语言。用非注音符号字母来替代注意符号字母。

```lang-java,fid-beacda9f4c7945b7bd743e3da8b4cd7f
@args Bar bar
@bar
@bar.noAccents()
```

#### [lowerFirst] lowerFirst() and capFirst()

将字串的首字母变成小写或者大写。

```lang-java,fid-ae7162075ef5498eb8af84beb8f77b02
@args Bar bar, String s
@bar
@bar.lowerFirst()
---
@s
@s.capFirst()
```

#### [camelCase] camelCase()

将下划线表达转成驼峰表达。例如`hello_world`变成`HelloWorld`

```lang-java,fid-baaa389eae37422b8a48508bd0889fe9
@args Bar bar
@bar
@bar.camelCase()
```

#### [divide] divide()

TBD

#### [lowerCase]lowerCase()

TBD

#### [upperCase]upperCase()

TBD

### [misc]Misc utilities

#### [len]len()

TBD

#### [nl2br] nl2br()

将字串中的换行符变成html的`<br/>`标签

```lang-java,fid-e0497048c78248129c13d7b69aa14d52
@args Bar bar
@bar
@bar.nl2br()
```

#### [urlEncode] urlEncode()

使用UTF-8编码对字串做urlEncode操作

```lang-java,fid-f77daffacf0848bfaa36308b0a4b8804
@args Bar bar
@bar
@bar.urlEncode()
```

#### [i18n] i18n()

对字串去国际化消息
Internationalization a string or string of an object

```lang-java,fid-c659f57f6ffd48978b795b244c5b2eeb
@args String x
//-- default locale
@x: @x.i18n()

// -- locale: en
@locale("en"){
@x: @x.i18n()
}

// -- locale: zh_CN
@locale("zh_CN") {
@x: @x.i18n()
}
```

#### [join] join()

* `join()` - join use `,`
* `join(String separator)`
* `join(char separator)`

使用缺省分割符`,`或者指定分割符连接一个[Iterable](http://docs.oracle.com/javase/6/docs/api/java/lang/Iternable.html)类型的变量

```lang-java,fid-e0497048c78248129c13d7b69aa14d52
@args Bar bar
@bar
@bar.join()
```

### [see] 相关资料

* [表达式](template_guide.md#expression)
* [用户定义转换器](user_defined_transformer.md)
* [转换器API](http://rythmengine.org/api/org/rythmengine/extension/Transformer.html)

