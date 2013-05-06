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

### [pattern]Common pattern

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

### [params]Passing parameters

The sample code in the [common pattern](#pattern) section shows how to pass data from user application to a template using `java.util.Map<String, Object>`. The key of the entry in the map corresponds to the template argument name. 

Specifically in the sample code, the template has declared an argument `who` with `@args String who`. Thus if you want to pass a value bind to the `who` argument to the template, in the application you need to do this:

```lang-java
Map<String, Object> conf = ...
conf.put("who", "World");
```

In simple cases you can also use a much light way to pass parameters, i.e. passing parameters by location as follows:

```lang-java
Rythm.render("@args String who\nHello @who!", "World");
Rythm.render("@args String first, int second\n1: @first\n2: @second", "foo", 234);
```

So in the above sample code, we pass parameter's by the position in the declaration sequence. This pattern could save you some typings in ver simple cases, but generally you should stick to the passing by name way to make it clear and less error prone.

 