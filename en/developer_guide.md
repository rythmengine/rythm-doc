# Developer's Guide

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

The following code demonstrate how to using `com.greenlaw110.rythm.Rythm` class, the singleton pattern to render a template:

```lang-java
import com.greenlaw110.rythm.*;
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
import com.greenlaw110.rythm.*;
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

1. Passing the configuration (a `Map<String, Object>` instance) to the Rythm engine [constructor](http://rythmengine.org/api/com/greenlaw110/rythm/RythmEngine.html#RythmEngine(java.util.Map)) or the Rythm facade [init method](http://rythmengine.org/api/com/greenlaw110/rythm/Rythm.html#init(java.util.Map))
2. Passing the configuration (a `java.io.File` instance) to the Rythm engine [constructor](http://rythmengine.org/api/com/greenlaw110/rythm/RythmEngine.html#RythmEngine(java.io.File)) or the Rythm facade [init method](http://rythmengine.org/api/com/greenlaw110/rythm/Rythm.html#init(java.io.File))
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

If a configuration item is of a specific implementation, e.g. the [cache.service.impl](http://rythmengine.org/api/com/greenlaw110/rythm/conf/RythmConfigurationKey.html#CACHE_SERVICE_IMPL), the key might or might not be end with "`.impl`". So the following configuration settings are exactly the same:

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
* [Force loading inline template](#force_inline_template)

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

An immediate question is how Rythm locate the file `helloWorld.txt`. Rythm provides a default file [resource loader](http://rythmengine.org/api/com/greenlaw110/rythm/extension/ITemplateResourceLoader.html) to support resource locating and loading. Besides the default file resource loader, the application developer can configure a customized [resource loader](http://rythmengine.org/api/com/greenlaw110/rythm/extension/ITemplateResourceLoader.html) to plugin logic to load resources fit different environment, e.g. to load template resource from a database or from a [in-memory resource caching system](https://github.com/freewind/rythmfiddle/blob/master/app/models/InMemoryResourceLoader.java) which is exactly the case for the [rythm fiddle tool](http://fiddle.rythmengine.org/).

In case the [resource loader](configuration.md#resource_loader_impl) option has not being configured, or the configured resource loader failed to load the resource by name `helloWorld.txt`, the default file resource loader will pick up the job. It will look for the file based on the [template root directory](configuration.md#home_template_dir). If file resource loader also failed to locate the file, then a class path resource loader will try to load the file based on the class path root. In case non of those resource loader locates the `helloWorld.txt` file, it will be treated as inline template, thus the final output will become "helloWorld.txt" instead of the expected "Hello Rythm!".

#### [force_inline_template]Force rythm to load template as an inline template

So everytime you invoke the `RythmEngine.render()` or `Rythm.render()` method, it will always try to load the template as an external resource and until all resource loaders fail, Rythm will fall back to treat it as an inline template. If you already know the template content is an inline string content, you can force Rythm to load it as inline template directly without bothering the external resource loaders:

```lang-java
engine.renderString("@args String who\nHello @who!", "Rythm");
// or the alias of renderString
engine.renderStr("@args String who\nHello @who!", "Rythm");
```

#### [force_file_template]Force rythm to load template as an external file

No suprising Rythm also provides API to force load template as an external file:

```lang-java
engine.render(new File("path/to/helloWorld.txt"), "Rythm");
```

### [render_setting]Render Settings

TBD

### [miscs]More features

* [Substitute mode](#substitute)
* Type inference

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

Fair enough, right? So here is the rule leads to base of **substritute mode** of Rythm engine: when all your template arguments are referenced in simple way you can save argument declaration. By simple way it means you reference a Java instance ONLY by it's `toString()` method, no other fields/methods referenced. 
