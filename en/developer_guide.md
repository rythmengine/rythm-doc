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

#### [conf_summary] Summary

This section explains the Rythm engine configuration and it's rules. For details about each configuration item, please refer to the [configuration reference](configuration.md)