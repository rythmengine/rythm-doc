# User Defined Transformer

### [create-class]Create the class/method

In order to create a user defined transformer, you need to create a class with transform methods and annotate it with `@com.greenlaw110.rythm.extension.Transformer` annotation. When a class is annotated with `@Transformer`, all public static method with return type are treated as transform methods. For example, in the [TransformerTest.java](https://github.com/greenlaw110/Rythm/blob/master/src/test/java/com/greenlaw110/rythm/advanced/TransformerTest.java) we defined three transform methods which provides `dbl` semantic to `Integer`, `String` and general `Object` type:

```lang-java
@Transformer
public class TransformerTest extends TestBase {
    public static Integer dbl(Integer i) {
        return i * 2;
    }
    public static String dbl(String s) {
        if (null == s) return "";
        return s + s;
    }
    public static String dbl(Object o) {
        if (null == o) return "";
        return dbl(o.toString());
    }
    ...
}
```

The `Transformer` annotation could also be used on a single method instead of the entire class, which makes it really easy to add specific transformer to your domain model classes. For example, in the [Order.java](https://github.com/greenlaw110/rythmfiddle/blob/master/app/demo/Order.java) class we defined a transform method `asCurrency`

```lang-java
    @Transformer(requireTemplate = true)
    public static String asCurrency(int amount) {
        return asCurrency(null, amount);
    }
    public static String asCurrency(ITemplate template, int amount) {
        Double d = (float)amount / 100.00;
        return S.formatCurrency(template, d, null, null);
    }
```

<div class="alert alert-info">Note in this transformer annotation we have specified <code>requireTemplate</code> parameter to true because we need the localization context of the template to format currency.</div>

### [register]Register transformer

After you defined the transformer classes/methods you need to register them to the `RythmEngine`. You can find relevant code in the [TransformerTest.java](https://github.com/greenlaw110/Rythm/blob/master/src/test/java/com/greenlaw110/rythm/advanced/TransformerTest.java) file:

```lang-java
@Test
public void testUserDefinedTransformer() {
    Rythm.engine().registerTransformer(TransformerTest.class);
    String t = "@args String s, int i\n" +
            "double of \"@s\" is \"@s.app_dbl()\",\n " +
            "double of [@i] is [@i.app_dbl().format(\"0000.00\")]";
    String s = Rythm.render(t, "Java", 99);
    assertContains(s, "double of \"Java\" is \"JavaJava\"");
    assertContains(s, "double of [99] is [0198.00]");
}
@Test
public void testUserDefinedTransformerWithNamespace() {
    // test register with namespace specified
    Rythm.engine().registerTransformer("foo", "", TransformerTest.class);
    String t = "@args String s, int i\n" +
            "double of \"@s\" is \"@s.foo_dbl()\",\n " +
            "double of [@i] is [@i.foo_dbl().format(\"0000.00\")]";
    String s = Rythm.render(t, "Java", 99);
    assertContains(s, "double of \"Java\" is \"JavaJava\"");
    assertContains(s, "double of [99] is [0198.00]");
}
```

So as it shows above, there are two `RythmEngine` methods you can use to register your transformers:

<ul>
<li>
<a href="http://rythmengine.org/api/com/greenlaw110/rythm/RythmEngine.html#registerTransformer(java.lang.Class...)"><code>RythmEngine.registerTransformer(Class...)</code></a>
</li>
<li>
<a href="http://rythmengine.org/api/com/greenlaw110/rythm/RythmEngine.html#registerTransformer(java.lang.String, java.lang.String, java.lang.Class...)"><code>RythmEngine.registerTransformer(String, Class...)</code></a>
</li>
</ul>

The first method accept an array of classes, and the second method has one additional parameter to define the namespace. Once an namespace is specified, then you need to invoke transformer with `namespace_<transformer>` notation. As shown in the above code, when you register the transformer without namespace specified, the namespace is the default "app", so you invoke the transformer in your template code using `app_`:

```lang-java
double of "@s" is "@s.app_dbl()"
``` 

And when you registered the transformer with namespace `foo`, your template code should be:

```lang-java
double of "@s" is "@s.foo_dbl()"
```

You can also specify the namespace in the `@Transformer` annotation when you define the transformer class or methods:

```lang-java
@Transformer("foo")
public static String transformXX(...)
```

### See also

* [Expression](template_guide.md#expression)
* [Built-in transformers](builtin_transformer.md)
* [Transformer API](http://rythmengine.org/api/com/greenlaw110/rythm/extension/Transformer.html)

