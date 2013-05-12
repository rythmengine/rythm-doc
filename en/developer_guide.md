<h1 data-book="developer_guide">Developer's Guide</h1>

This document details how to use Rythm in your Java program.

### [introduction]Introduction

Rythm is an easy to use Java template engnine. You can use it with zero configuration in simplest scenario:

```lang-java
// render an inline template
Rythm.render("@args String who;hello @who", "Rythm");
// render a file template
Rythm.render("mytemplate.tmpl", "Rythm");
```

### [pattern]Common usage pattern

When using Rythm in an application, you will generally do the following:

1. Initialize Rythm. This applies to both Singleton or separate runtime engine instance (see more on this below), and you only do this once.
2. Create a `Map<String, Object>` to hold the template parameter
3. Add data into the parameter map
4. Identify the template (either inline content or a file) to be used
5. Render the template by passing the parameter map along with the template identifier to Rythm.

The following code demonstrate how to using `org.rythmengine.Rythm` class, the singleton pattern to render a template:

```lang-java
import org.rythmengine.*;
import java.util.*;
...

public class Foo {
    boolean rythmInitialized = false;
    public void init() {
        if (rythmInitialized) return;
        Map<String, Object> conf = ...
        ...
        Rythm.init(conf);
    }
    ...
    public void doIt() {
        init();
        Map<String, Object> params = new HashMap<String, Object>();
        params.put("who", "World");
        System.out.println(Rythm.render("@args String who\nHello @name", params));
    }
}
``` 

### [singleton_or_not] To Singleton Or Not

The above sample code shows how to use `Rythm` facade (which mimic a Singleton pattern) to init engine and render template. 

Rythm also supports creating multiple engine instance in a JVM process, which makes it very approprate to be used in certain environment, for example, the Servlet 2.2+ compliant web application, as each web application can have its own instance of Rythm. 

Below is the code that rewrite the above sample code with non singleton pattern:

```lang-java
import org.rythmengine.*;
import java.util.*;
...

public class Foo {
    RythmEngine engine = null;
    public void init() {
        if (null == engine) {
            Map<String, Object> conf = ...
            ...
            engine = new RythmEngine(conf);
        }
    }
    ...
    public void doIt() {
        init();
        Map<String, Object> params = new HashMap<String, Object>();
        params.put("who", "World");
        System.out.println(engine.render("@args String who\nHello @name", params));
    }
}
``` 

As you can see, this is also very simple and straightfoward. Except for some simple syntax changes, using Rythm as a singleton or as separate instances requires no changes to the high-level structure of your application or templates.

In this section we reviewed the fundamental patterns to use Rythm in your application including singleton and non-singleton usage. In the next section we will take a close look at how to configure Rythm in an application and some rules Rythm follows to process configuration.

### [configuration]Configuration

Though you can use Rythm with zero configuration, Rythm provides a rich set of configuration to enable you to customize the template enigne behavior to fit your needs. There are several places you can use to inject the configuration to Rythm engine:

1. Passing the configuration (a `Map<String, Object>` instance) to the Rythm engine [constructor](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#RythmEngine(java.util.Map)) or the Rythm facade [init method](http://rythmengine.org/api/org/rythmengine/Rythm.html#init(java.util.Map))
2. Passing the configuration (a `java.io.File` instance) to the Rythm engine [constructor](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#RythmEngine(java.io.File)) or the Rythm facade [init method](http://rythmengine.org/api/org/rythmengine/Rythm.html#init(java.io.File))
3. The implicit `rythm.conf` file which can be found via `Thread.currentThread().getContextClassLoader().getResource("rythm.conf")` call
4. The `System.properties`

The priority of the configuration setting is API overwrites `System.properties` which overwrites the configuration file.

The rythm configuration is implemented to be smart and flexible. Here are the rules about how Rythm read the configurations:

#### [conf_prefix] Key prefix

The configuration key may or may not start with "`rythm.`" prefix. For example, the following configuration settings are exactly the same

```
rythm.engine.mode=dev
engine.mode=dev
```

When both key presented, the one with "`rythm.`" prefix overwrite the one without it.

#### [conf_boolean] Boolean type key suffix

If a configuration item is of boolean type, the key may or may not end with "`.enabled`" or "`.disabled`".  So the following configuration settings are exactly the same

```
built_in.transformer=true
built_in.transformer.enabled=true
built_in.transformer.disabled=false
```

Suppose you have both `foo`, `foo.enabled`, `foo.disabled` exists in the configuration, then the priority is `foo.disabled` overwrite `foo.enabled` which in turn overwrite `foo`. For example, giving the following configuration:

```
feature.transform.enabled=true
feature.transform.disabled=true
```

The `feature.transform.disabled=true` win and Rythm will disable the transform feature.

#### [conf_impl] Implementation key suffix

If a configuration item is of a specific implementation, e.g. the [cache.service.impl](http://rythmengine.org/api/org/rythmengine/conf/RythmConfigurationKey.html#CACHE_SERVICE_IMPL), the key might or might not be end with "`.impl`". So the following configuration settings are exactly the same:

```
cache.service.impl=foo.MyCacheService
cache.service=foo.MyCacheService
```

#### [conf_dir] Directory key suffix

If a configuration item is of Directory(`File`) type, the key may or may not be end with "`.dir`". So the following configurations are exactly the same:

```
home.template=rythm
home.template.dir=rythm
```

When both configuration key presented the one with `.dir` suffix overwrite the one without it.

#### [conf_type] Configuration type

Rythm support specifying configuration value in `String` or the required type.

##### Boolean type

Specify boolean type configuration in API:

```lang-java
Map<String, Object> conf = new HashMap<String, Object>();
conf.put("feature.transform.enabled", true);
// the following is the same but the above is preferred
conf.put("feature.transform.enabled", "true");
```

Specify boolean type configuration in properties file

```
feature.transform.eanabled=true
```

##### Number type

Specify number type configuration in API:

```lang-java
Map<String, Object> conf = new HashMap<String, Object>();
conf.put("sandbox.timeout", 1000);
// the following is the same but the above is preferred
conf.put("sandbox.timeout", "1000");
```

Specify number type configuration in properties file

```
sandbox.timeout=1000
```

##### File type

Specify file type configuration in API:

```lang-java
Map<String, Object> conf = new HashMap<String, Object>();
conf.put("home.template", new File("path/to/template/root"));
// the following is also okay
conf.put("home.template", "path/to/template/root");
```

Specify file type configuration in properties file

```
home.template=path/to/template/root
```

##### Extension implementation

Specify extension implementation in API

```lang-java
Map<String, Object> conf = new HashMap<String, Object>();
conf.put("cache.service", new MyCacheService());
// this is also good
conf.put("cache.service", MyCacheService.class);
// or pass in String
conf.put("cache.service", "MyCacheService.class");
```

Specify extension implementation in properties file

```
cache.service=MyCacheService.class
```

This section explains the Rythm engine configuration and it's rules. For details about each configuration item, please refer to the [configuration reference](configuration.md). In the next section we will inspect how to pass parameters from your application to template. 

### [render]Render template

In this section we will review how to use Rythm API to call template and pass parameters. We will cover the following topics in this section:

* [Passing parameter by name](#by_name)
* [Passing parameter by position](#by_position)
* [Position indicator](#position_indicator)
* [Inline template](#inline_template)
* [External template resource](#external_template)
* [Force to load inline template](#force_inline_template)
* [Force to load file template](#force_file_template)

So the simple way to render a template is to call the `render(<template>, <params>)` method of the `Rythm` facade or a `RythmEngine` instance:

```lang-java
engine.render("@args String who\nHello @who!", "Rythm");
// or use the singleton
Rythm.render("@args String who\nHello @who!", "Rythm");
```

The `render` API is flexible to support different ways of passing parameters and template data. Let's first take a look at parameter passing handling in Rythm engine:

#### [by_name]Passing parameters by name

The sample code in the [common pattern](#pattern) section shows how to pass data from user application to a template using `java.util.Map<String, Object>`. The key of the entry in the map corresponds to the template argument name. 

Specifically in the sample code, the template has declared an argument `who` with `@args String who`. Thus if you want to pass a value bind to the `who` argument to the template, in the application you need to do this:

```lang-java
Map<String, Object> conf = ...
conf.put("who", "World");
Rythm.render("@args String who\nHello @who!", conf);
```

APIs:

* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#render(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#render(java.lang.String, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/Rythm.html#render(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#render(java.lang.String, java.lang.Object...))

#### [by_position]Passing parameters by position

In simple cases you can also use a much light way to pass parameters, i.e. passing parameters by location as follows:

```lang-java
Rythm.render("@args String who\nHello @who!", "World");
Rythm.render("@args String first, int second\n1: @first\n2: @second", "foo", 234);
```

So in the above sample code, we pass parameter's by the position in the declaration sequence. This pattern could save you some typings in very simple cases, but generally you should stick to the passing by name way to make it clear and less error prone when your template exceeds more than one line of code or has more than 3 template arguments.

#### [position_indicator]Position indicator

So in those simple cases we can pass parameters by position, and now we are going to push it one step ahead to make it even simpler. We will use position indicator instead of name in the template:

```lang-java
Rythm.render("@args String @1\nHello @1;Goodbye @2", "Rythm", "Velocity");
```

In the above code we use position indicator `@1` and `@2` as the variable name in the template source. Rythm will change those name into something like `__v_1` and `__v_2` in the generated Java source code. 

Along with passing parameters by position, position indicator suit for simple use cases and makes the code concise and clear.

Now that we have the idea on how to pass parameters to Rythm template, we can take a look at how to specify the template to Rythm engine. Basically you have two ways to specify template:

1. [specify template content directly](#inline-template)
2. [specify template file name or resource reference](#template-resource)

#### [inline_template]Inline template

Until now the samples we shown are all passing template content directly to the engine via a String. This is called inline template. It is used in simple, trivial and temporary cases where putting the short content into an external file or resource doesn't tradeoff. It is recommended not to use inline template when your template is more than 255 characters, in which case passing a file name or a resource reference to Rythm engine makes more sense.

#### [external_template]External template resource

In most cases your template should be reasonably complicated enough to be put into a separate file, in which case you can pass the file path to the `render` API:

```lang-java
Rythm.render("helloWorld.txt", "Rythnm");
```

Where the content of `helloWorld.txt` could be something like:

```
@args String who
Hello @who!
```

An immediate question is how Rythm locate the file `helloWorld.txt`. Rythm provides a default file [resource loader](http://rythmengine.org/api/org/rythmengine/extension/ITemplateResourceLoader.html) to support resource locating and loading. Besides the default file resource loader, the application developer can configure a customized [resource loader](http://rythmengine.org/api/org/rythmengine/extension/ITemplateResourceLoader.html) to plugin logic to load resources fit different environment, e.g. to load template resource from a database or from a [in-memory resource caching system](https://github.com/freewind/rythmfiddle/blob/master/app/models/InMemoryResourceLoader.java) which is exactly the case for the [rythm fiddle tool](http://fiddle.rythmengine.org/).

In case the [resource loader](configuration.md#resource_loader_impl) option has not being configured, or the configured resource loader failed to load the resource by name `helloWorld.txt`, the default file resource loader will pick up the job. It will look for the file based on the [template root directory](configuration.md#home_template_dir). If file resource loader also failed to locate the file, then a class path resource loader will try to load the file based on the class path root. In case non of those resource loader locates the `helloWorld.txt` file, it will be treated as inline template, thus the final output will become "helloWorld.txt" instead of the expected "Hello Rythm!".

#### [force_inline_template]Force rythm to load template as an inline template

So everytime you invoke the `RythmEngine.render()` or `Rythm.render()` method, it will always try to load the template as an external resource and until all resource loaders fail, Rythm will fall back to treat it as an inline template. If you already know the template content is an inline string content, you can force Rythm to load it as inline template directly without bothering the external resource loaders:

```lang-java
engine.renderString("@args String who\nHello @who!", "Rythm");
// or the alias of renderString
engine.renderStr("@args String who\nHello @who!", "Rythm");
```

APIs:

* [http://rythmengine.org/api/org/rythmengine/Rythm.html#renderString(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#renderString(java.lang.String, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/Rythm.html#renderString(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#renderString(java.lang.String, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#renderStr(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#renderStr(java.lang.String, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#renderString(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#renderString(java.lang.String, java.lang.Object...))

#### [force_file_template]Force rythm to load template as an external file

No suprising Rythm also provides API to force load template as an external file:

```lang-java
engine.render(new File("path/to/helloWorld.txt"), "Rythm");
```

APIs:

* [http://rythmengine.org/api/org/rythmengine/Rythm.html#render(java.io.File, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#render(java.io.File, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#render(java.io.File, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#render(java.io.File, java.lang.Object...))

### [render_setting]Render Settings

We have already known that there are a set of configuration can be setup for a single Rythm engine instance. However for each template rendering we can also configure some specific settings including

* [Locale](#rs_locale)
* [Code type](#rs_code_type)
* [A user supplied context map](#rs_user_context)

<div class="alert"><b>Note</b> the render setting only endure one rendering life time. They will be reset to default state after the rendering process finished. Meaning you need to reset them for the next render call if needed</div>

Once render setting get intialized the state of the settings will be passed to the template to be processed, and transfer to the template's [render context](#render_context) 

APIs:

* [http://rythmengine.org/api/org/rythmengine/RythmEngine.RenderSettings.html](http://rythmengine.org/api/org/rythmengine/RythmEngine.RenderSettings.html)
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(org.rythmengine.extension.ICodeType)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(org.rythmengine.extension.ICodeType))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(java.util.Locale)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(java.util.Locale))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(java.util.Map)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(java.util.Map))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(org.rythmengine.extension.ICodeType, java.util.Locale, java.util.Map)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#prepare(org.rythmengine.extension.ICodeType, java.util.Locale, java.util.Map))

#### [rs_locale]Locale Render Setting

The locale setting set up the default locale for a rendering process. This is very useful when using Rythm to render html page for an international web site, which requests might come from different regions:

```lang-java
engine.prepare(new Locale(request.getHeader("Accept-Language"))).render("myTemplate.html", params);
```

See also [Locale context](#rc_locale)

#### [rs_code_type]Code Type Render Setting

Set the default code type for the next rendering. This is useful especially when you are rendering an inline template where the the code type cannot be deduct from file name extension:

```lang-java
engine.prepare(ICodeType.DefImpl.JS).render("@args String foo;alert('@foo');", "Tom's cat is Jerry");
```

#### [rs_user_context]User context

Set a `Map<String, Object>` typed user context to the rendering process. Unlike template parameters, the user context is not aimed to provide parameter for template usage, but to pass information to other user defined extensions, e.g. a user defined transformer or user defined exception handler. 

Both user defined transformer and user defined exception handler can get the template instance, and thus in turn get the user context via API call:

```lang-java
// user defined transformer
@Transformer(requireTemplate = true)
public static foo(String s) {
    foo(null, s);
}
public static foo(ITemplate tmpl, String s) {
    if (tmpl != null) {
        Map<String, Object> userContext = tmpl.__getUserContext();
        ...
    }
}
...
// user defined exception handler
@Override
boolean handleTemplateExecutionException(Exception e, TemplateBase template) {
    Map<String, Object> userContext = template.__getUserContext();
    ...
}
```

### [render_context]Render Context

Render context is an aggregation of rendering states which is baselined by the combination of Rythm configuration and render setting, can could be altered during the rendering process via directives like `@locale()` or specific pattern in a template like `<script ...>`. 

Render context track the following states:

* [locale](#rc_locale)
* [Code type](#rc_code_type)
* [Escape scheme](#rc_escape)

APIs:

* [http://rythmengine.org/api/org/rythmengine/template/ITemplate.__Context.html](http://rythmengine.org/api/org/rythmengine/template/ITemplate.__Context.html)

#### [rc_locale]Locale context

The locale context state impacts the i18n logic. For example, the `@myDate().format()` in the template file use the locale context to find out the default date format:

```lang-java,fid-252ee8b90f544c97b9f3866095440a4f
@args Date date, Double amount
@{date = new Date();}

// -- default locale
@i18n("date"): @date.format()
@i18n("amount"): @amount.formatCurrency()

// -- locale: en_AU
@locale("en","AU") {
@i18n("date"): @date.format()
@i18n("amount"): @amount.formatCurrency()
}

// -- locale: zh_CN
@locale("zh", "CN"){
@i18n("date"): @date.format()
@i18n("amount"): @((amount*6.5).formatCurrency())
}
``` 

The locale state can be initialized in two ways:

1. Via `engine.prepare(Locale)` call before calling the `renderXX()` method. If locale is not prepared before calling render method, then the initial locale is the configured [i18n.locale](configuration.md#i18n_locale) settigns.
2. Interit locale state from the calling template if the current running template is called from within another template.

The locale state is altered when entering the `@locale(){...}` block and reset to previous state when exit the block.

#### [rc_code_type]Code type context

The code type context state impact the escape scheme when [smart escape](configuration.md#feature_smart_escape_enabled) feature is enabled. For example if a template file has `.html` extension, then the code type will be initialized to `ICodeType.DefImpl.HTML`, and the default escape scheme is set to `escapeXML()`. However when the template entered `<script ...>...</script>` block the default escape scheme will be changed to `escapeJS()` until it exits the `<script>` block:

```lang-java,fid-15104db6f90c4836b51e9eb606569a52
@args String html, String js

<p>
  html: @html
  js: @js
</p>
<script>
    html: @html
    js: @js
</script>
```

The code type state can be initialized in two ways:

1. Via `engine.prepare(CodeType)` call before calling the `renderXX()` method. If code type is not prepared then the engine will try to deduct the code type from the file extension info if the template is an external resource. If the code type cannot be inferred from the file extension or the template is an inline template content, then the [default code type](configuration.md#default_code_type_impl) configuration will be used.
2. Inherit code type state from the calling template if the current running template is called from within another template.

The rule of the code type inference:

```
.html or .htm -> ICodeType.DefImpl.HTML
.js -> ICodeType.DefImpl.JS
.json -> ICodeType.DefImpl.JSON
.xml -> ICodeType.DefImpl.XML
.csv -> ICodeType.DefImpl.CSV
.css -> ICodeType.DefImpl.CSS
```

#### [rc_escape]Escape scheme

The escape scheme impact the auto escape logic. In the following example, when the current escape scheme is `XML`, and variable `foo`'s value is `"<h1>Tom's Jerry</h1>"`, expression `@foo` will output `"&lt;h1&gt;Tom&apos;s Jerry&lt;/h1&gt"`; while the current escape scheme becomes `JS`, the same expression `@foo` output `"<h1>Tom\s Jerry<\/h1>"`:

```lang-html,fid-b8cafdf620c243b09af6849d8f599494
@args String foo

<p>@foo</p>
<script>@foo</script>
```

The escape scheme state is always intialized by engine automatically according to the state of [code type](#rc_code_type). And escape scheme can be reset by using `@escape()` directive or implicitly via `<script></script>` block:

```lang-html,fid-7c9eae433dff4644ba5a77ddd6d58193
@args String foo

<p>@foo</p>
<script>@foo</script>
@escape("csv") {@foo}
```

### [miscs]Other features

* [Substitute mode](#substitute)
* [Sandbox](#sandbox)

Rythm engine support 

#### [substitute]Substitute mode

Again in the following case where template is very simple, we have declared a template argument `who` with type `String`:

```lang-java
@args String who
Hello @who
```  

But we actually can change the type declaration from `String` to `Object`, because literally `@who` is equavlent to `@who.toString()`:

```lang-java
@args Object who
Hello @who
```

And the result will be exactly the same. But every java instance is of type `Object`, in which case we can save the argument declaration:

```lang-java
Hello @who
```

Fair enough, right? So here is the rule leads to base of **substitute mode** of Rythm engine: when all your template arguments are referenced in simple way you can save argument declaration. By simple way it means you reference a Java instance ONLY by it's `toString()` method, no other fields/methods referenced. 

By default the Rythm engine always try to treat a template as in **substitute mode** untill it encountered complex expression, then it will switch to full mode. To force Rythm to use only substitute mode to render a template use `substitute()` API:

```
String result = engine.substitute("Hello @who", "Rythm");
String result2 = engine.substitute(myTmplFile, "Rythm");
```

Note, you can not use certain directives and features in substitute mode including:

* scripting
* argument declaration
* assignment
* layout management
* template invocation
* include and macro
* compact, nocompact
* inline tag definition and invocation
* return from template execution

Initially subsitute mode is designed to achieve a light way to render template, kind of a faster version of `String.format()`. However, a side effect of substitute mode is it suit a simple template solution for unknown template resource very well. Let's you want the customer to provide template, for example, to define their email template, because the template is supplied from untrusted source, you don't a mal template to break your system. Sustitute mode is good solution if string substitution is the only requirement.

APIs:

* [http://rythmengine.org/api/org/rythmengine/Rythm.html#substitute(java.io.File, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#substitute(java.io.File, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/Rythm.html#substitute(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/Rythm.html#substitute(java.lang.String, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#substitute(java.io.File, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#substitute(java.io.File, java.lang.Object...))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#substitute(java.lang.String, java.lang.Object...)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#substitute(java.lang.String, java.lang.Object...))

#### [sandbox]Sandbox mode

Substitute mode is idea to handle untrusted template for simple usage scenario, however if your use case is beyond simple string substitution, for example the [rythm fiddle](http://fiddle.rythmengine.org) website wants the user to experience as much as possible Rythm feature, including free scripting, while the system shouldn't be broken just because someone typed in things like `@{System.exit(1);}`. In this case you need Sandbox mode.

Sandbox mode allows you to use full feature set while it has restrictions in parsing and executing a template:

1. A `SecurityManager` is loaded to prevent access to certain system resources at runtime including
    * Access to threads that are not the sandbox executing thread
    * Thread groups access
    * `Runtime.exit()` call
    * `Runtime.exec()` call
    * `Runtime.load()` and `Runtime.loadLibrary()` call
    * File IO operations
    * Socket operations
    * `System.getProperties()` call
    * `System.getProperty()` call with the property key not found in the [allowed system properties](configuration.md#sandbox_allowed_system_properties) configuration
    * AWT window operations
    * Print jobs
    * System clipboard access
    * Package definition (via class loader's `loadClass()` method call)
    * `System.setSecurityManager()` call
2. If the template source contains strings matches the [sandbox.restricted_class](configuration.md#sandbox_restricted_class) configuration, then a parsing exception will be thrown out.
3. When the template executing time exceed the [sandbox.timeout](configuration.md#sandbox_timeout) configuration, the sandbox executing thread will be interrupted and the executing is abort.

To force Rythm run template in Sandbox mode, use the `sandbox()` API defined in both engine and singleton:

```lang-java
result = engine.sandbox().render(...);
result = Rythm.sandbox().render(...);
```

APIs:

* [http://rythmengine.org/api/org/rythmengine/Rythm.html#sandbox()](http://rythmengine.org/api/org/rythmengine/Rythm.html#sandbox())
* [http://rythmengine.org/api/org/rythmengine/Rythm.html#sandbox(java.util.Map)](http://rythmengine.org/api/org/rythmengine/Rythm.html#sandbox(java.util.Map))
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#sandbox()](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#sandbox())
* [http://rythmengine.org/api/org/rythmengine/RythmEngine.html#sandbox(java.util.Map)](http://rythmengine.org/api/org/rythmengine/RythmEngine.html#sandbox(java.util.Map))

### [extension]Extends Rythm

This section describes how to extend Rythm to make it more cater to your application context. It includes the following topics:

* [User Defined Transformer](#user_defined_transformer)

#### [user_defined_transformer]User Defined Transfomer

Rythm provide a set of useful [built-in transformers](builtin_transformer.md) for commonly used string manipulations as well as varieties of format operations. However when certain processing on output is not covered by those built in transformers, you can still enjoy the simplicity and elegancy of Rythm transform syntax by implementing your own transformer.

The first step is to define a class with transformer logic implemented in a public static method with return value:

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

The `Transformer` annotation could also be used on a single method instead of the entire class, which makes it really easy to add specific transformer to your domain model classes. For example, in a sample [Order.java](https://github.com/greenlaw110/rythmfiddle/blob/master/app/demo/Order.java) class we defined a transform method `asCurrency`
  
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

Once you defined the transform logic, you need to register the relevant class to the Rythm engine:

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
<a href="http://rythmengine.org/api/org/rythmengine/RythmEngine.html#registerTransformer(java.lang.Class...)"><code>RythmEngine.registerTransformer(Class...)</code></a>
</li>
<li>
<a href="http://rythmengine.org/api/org/rythmengine/RythmEngine.html#registerTransformer(java.lang.String, java.lang.String, java.lang.Class...)"><code>RythmEngine.registerTransformer(String, Class...)</code></a>
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

<div class="alert alert-info">
<i class="icon-info-sign"></i> You can also register transformer class by setting the <a href="configuration.md#transformer_udt">transformer.udt</a> configuration
</div>