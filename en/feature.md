# Features

Rythm is yet another Java Template Engine. It provides an easy to use, super fast and general purpose template engine to Java programmer. On the other side Rythm could be used in any Java program and does not require a specific framework.

### [developer-friendly]Developer friendly

Rythm is created by developer for developers and the user experience is the number one concern of the product. The beauty of simplicity is been built into the product, from API to the template syntax.

Code is better than word. Let's take a look at a sample template first:

```lang-html,fid-cc5533f277d64ed69678738f97b91227
@// template line comment

@**
 * template block comment 
 * can across lines
 *@

@// declare the arguments used in this template
@args List<Order> orders, int maxLines, User user

<h1>Order Manager</h1>

@ifNot (user.hasRole("order-manager")) {
    <p>You don't have access to this page</p>
    @return @// break the template execution with @return
}

<div id="order-list">
@for(Order order : orders) {
    @if(order_index >= maxLines) {
        <a href="#" id="load-more">Click here to load more order...</a>
        @break @// break the loop when maximum line reached
    }
    @**
     * loop variables used here: 
     * _parity: "odd" or "even" depends on current loop index
     * _isFirst: true if this is the first element
     * _isLast: true if this is the last element in the loop
     *@
    <div class='order @order_parity @(order_isFirst ? "first" : "") @(order_isLast ? "last" : "")'>
        <h3>@order.getName()</h3>
        @if (order.closed()) {
            <div>closed</code>
        } else {
            <div>...</div>
        }
    </div>
} else {
    @// the else block is executed while the iterable is empty
    <div class="alert alert-info">Order not found</a>
}
</div>
```

As shown above Rythm use a single special character `@` to introduce all syntax elements, including comments, flow control, variable evaluation. The idea is borrowed from [.Net Razor](http://weblogs.asp.net/scottgu/archive/2010/07/02/introducing-razor.aspx), and yes Razor inspired me to create Rythm because of this simple elegant syntax.

Another design philosophy of Rythm is to make it native to Java programmer as much as possible. Generally speaking, an experienced Java programmer shouldn't have any difficulty to read and understand a Rythm template even if he/she has never known Rythm before, and they should be able to start working on Rythm template in a few minutes.

And here are some code demonstrate how to use Rythm API in Java code:

```java
// inline template pass the template content directly
String s = Rythm.render("Hello @who", "world");

// file template use the same API but pass in the file name
String s = Rythm.render("hello.txt", "world");

// pass render arguments by position
String s = Rythm.render("Hello @1", "world");

// pass render arguments by name
Map<String, Object> params = new HashMap<String, Object>();
params.put("who", "world");
String s = Rythm.render("Hello @who", params);

// or pass by name with a bit more fancy
NamedParams np = NamedParams.instance;
String s = Rythm.render("hello @who", np.from(np.pair("who", "world")));
```

As shown above, it uses the same set of API to overload different intention and keep it still simple and transparent. This is another design principle of Rythm.

### [high-performance]High performance

Being different from dynamic engines like [Velocity](http://velocity.apache.org/) and [FreeMarker](http://freemarker.sourceforge.net/), Rythm is a static typed engine and compile your template source into java byte code, and thus very fast. Based on the result of [this benchmark test](https://github.com/greenlaw110/template-engine-benchmarks), Rythm is one of the fastest template engines in Java world.

![benchmark-image](../img/benchmark.png)

### [general-purpose]General purpose

Rythm is designed as a general purpose template engine. It allows you to generate html page, xml file, source code, SQL script, email content and any other kind of **text based artifacts**.

### Template auto reload

Rythm can be started in 2 different modes: **prod**(Product) and **dev**(Development). 

When running in development mode, Rythm monitors template source files, and automatically reload the content once one changes. This process is very fast, usually you won't be able to notice the delay. 

Auto reload feature is disabled when Rythm is running in product mode to maximize the performance gain.

### [extensibility]Extensibility and reusability

Rythm provides varieties of ways for template author to reuse and extend their template library, including:

* **Template invocation**: Most common and powerful way to extend template library. Rythm allows it to invoke another template using Java method notation. E.g. If you want to call a template file `${tmpl_home}/bar/foo.html` from within another template, you simply issue 

    ```java
    @bar.foo()
    ``` 

    If there are arguments defined in `foo.html`:

    ```java
    @args String user, int age
    ...
    ```
    
    Then you can pass parameters in template invocation:
    
    ```java
    // pass params by position
    @bar.foo("Rythm", 1)
    
    // or pass params by name
    @bar.foo(user: "Rythm", age: 1)
    
    // or even JS style
    @bar.foo({
        user: "Rythm",
        age: 1
    })
    ```
     
* **Inline methods**: Defining an internal tag is as simple as defining a method of a class
* **Macro**: The fastest way to reuse code block inside one template file
* **Include directive**: Including other template content in place at parsing time, and reuse internal tags defined in the including template.
* **Java Extension**: A mechanism to attach new methods to Java types in expression evaluation

### [security]Security

By default Rythm runs template without any restriction, this is good as long as the template code is under the control of the project. However, when you want to accept and execute template source code from untrusted source, e.g. end user input, you need to make sure the code won't break through your environment. 

Rythm provides a restricted environment named "Substitute Mode" to removed the security concern raised when you needs to process templates coming from untrusted source. Note this mode is a very restricted and most features are tailored due to security concern, name a few of them: _Expression Evaluation_, _Scripting_ and _Free style looping_. All these means you have absolute security to run whatever templates without worrying about your system get broken. To render templates in "Substitute Mode" simply use `substitute` instead of `render` API:

```java
String s = Rythm.substitute(unTrustedTemplate, ...)
```

In case you really want to provide features beyond the limits of "Substitute Mode" to external template provider, Rythm provides sandbox mode to make sure the untrusted template code is running in a secure way. The sandbox mode also prevent infinite loop from eating up your resources. As usual Rythm provide a simple API to enter Sandbox:

```java
String s = Rythm.sandbox().render(unTrustedTemplate, ...);
```
    
Please be noted that sandbox is not a free lunch. When Rythm is running in sandbox mode, it takes about 40% more time to render the same template with the same input compare to normal mode.   

### [i18n]I18N and I10N

Things can be easier than Rythm for I18N and I10N:

```java
<h1>@i18n("main.title")</h1>
...
<p>@i18n("template", "planet", 7, new Date())</p>
```

Might generate

```html
<h1>Top News</h1>
...
<p>At 3:53 PM on 11 March 2013, we detected 7 spaceships on the planet Mars</p>
```

or 

```html
<h1>头条新闻>
<p>我们于2013年3月11日的下午3:53在火星上发现了7艘宇宙飞船</p>
```

depending on the locale context when running the template. And let's take a look at localization case:

```java
@args Date date, Double amount
@i18n("date"): @date.format()
@i18n("amount"): @amount.formatCurrency()
```

Based on the locale context, the above code generates 

```
Date: 11 March 2013
Amount: $20000.00
```

or

```
日期：2013年3月11日
金额：￥20000.00
```

Easy, isn't?
    
### [rich-function]Rich functionality

Rythm provides wide range of templating features includes

* Template layout management
* Call other template and callback
* Define inline method
* Transformer
* Template cache
* ... 

Check all of them out at [Document Center](index.md)
