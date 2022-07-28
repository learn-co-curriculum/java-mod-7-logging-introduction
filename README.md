# Logging Intro

## Learning Goals

- Explain why logging is important.
- Explain the different types of log levels.

## Why not `System.out`

While the `System.out` method has allowed us to very easily display text
information to our application's console, it has several important limitations:

- All print statements are the same, which means the only way we can make them
  conditional is by putting each statement in a conditional block, which is a)
  time consuming and b) would require us to change every single conditional
  statement every time we wanted to change the way our application prints
  information to the console.
- `System.out` print statements are printed directly to the console, but not all
  applications have an actual console that someone can read, and most
  importantly the console output scrolls over time and is generally not
  recoverable later.
- Print statements are "local", so their ability to scale with distributed
  systems is not existent

The Spring Framework gives us access to logging libraries that solve all these
issues. Let's dive in!

## Default logging in Spring

As mentioned earlier, the Spring Framework comes with logging capabilities, and
the Spring Boot bundles we have been using give us access to these logging
capabilities out of the box.

We can add simple logging to our `HelloController` as follows:

```java
    Logger logger = LoggerFactory.getLogger(HelloController.class);

    @GetMapping("/api/hello")
    public String hello(@RequestParam(name = "targetName", defaultValue = "Stephanie") String name) {
        System.out.println("In HelloController.hello() method");
        logger.info("In HelloController.hello() method");
        String greeting = "Hello " + name;
        greeting += "<br/>";
        greeting += "Dad joke of the moment: " + jokeService.getDadJoke();
        return greeting;
    }
```

Note that there might be several choices of packages to import from when you try
to tell your IDE which `Logger` class to import. Here are the proper import
statements for the code above:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

Now let's break down this code:

1. "SLF4J" is Simple Logging Facade for Java and it offers a generic API to
   abstract the actual underlying implementation.
2. In the default Spring configuration, the underlying implementation is
   Logback, which is a popular, full featured and well-supported logging
   utility.
3. SLF4J comes with a factory that will return an instance of the `Logger` class
4. We request an instance of `Logger` by passing the `LoggerFactory` the name of
   the class for which we wish to log statements. This enables functionality
   that we will cover later in this module.
5. We then use the `logger.info()` method like we would use the
   `System.out.println()` method. More on the significance of the `info()`
   method later in this section.
6. We also include the same output using the `System.out.println()` method, so
   we can contract the 2 results

Let's look at the output of the `HelloController` now that we've added this log
statement:

```java
2022-07-10 00:59:11.147  INFO 25285 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-07-10 00:59:11.147  INFO 25285 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-07-10 00:59:11.148  INFO 25285 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
In HelloController.hello() method
2022-07-10 00:59:12.058  INFO 25285 --- [nio-8080-exec-4] c.f.s.FlatironSpring.HelloController     : In HelloController.hello() method
```

You should get output similar to the last 2 lines in the output above. A few
things to note:

1. The logger automatically outputs the date and time on the server at the time
   that a given statement is output to the console: `2022-07-10 00:59:12.058`
2. Next, the logger outputs the "log level" at which your statement is being
   displayed: `INFO`. This is related to the configuration options that the
   logger gives us, which allows us to control what statements get outputted.
   More on this below.
3. Finally, before the actual content of the `info()` statement is printed out,
   the logger prints out the class that the statement comes from:
   `c.f.s.FlatironSpring.HelloController`
4. Compare that to the output of the `System.out.println()` and you can see that
   logging with the logger gives us much more contextual information, with very
   little additional code required.

Note that the `c.f.s.FlatironSpring.HelloController` comes from the `class` that
was passed into the `LoggerFactory` when the `Logger` instance was requested.
You could pass in another class, and the corresponding statements would be
prepended with the name of that class, even they actually came from a different
class at runtime.

## Log Levels

One of the great features of a logging framework is that it gives us access to
specific logging methods that correspond to specific logging levels.

Logging levels are a way for us to tell the logging framework "when" we want
specific log statements to be displayed. The logging levels allow us to control
how "verbose" we want our application to be. The higher the logging level, the
less information it should output. This is by convention, so you are responsible
for calling your logger with a logging level that is appropriate for the
information you're trying to display.

When you use one of the logger's output method (e.g. `logger.info()`), the
logger will only output the corresponding if the current log level is at the
corresponding level or lower. In our example, we're using the `info()` method,
so the logger will only output our information is its current logging level is
INFO or lower. We will go over how to configure the logger's logging level in
detail later in this section, but first let's look at all the logging levels
supported by SLF4J:

1. TRACE: this is the most fine-grained level of logging and should be used to
   display very detailed information about what your application is doing.
   Expect all code used by your application to be extremely verbose when this
   logging level is enabled.
2. DEBUG: this is one level up from trace, so it should be less fine-grained,
   but still more output than the application should put out under "normal"
   circumstances. This level is used to get high level debugging information
   that falls short of including every single step that your application is
   going through.
3. INFO: this is the level that SLF4J is configured to log at by default, and
   should include high level information about what your application is going,
   e.g. a request was just processed, but not any of the details of how the
   result was achieved.
4. WARN: used to indicate that "something" went wrong, but wasn't fatal to the
   application. This is a log level that is almost always enabled because it's
   not expected to be used frequently.
5. ERROR: used to indicate an issue that is preventing the application from
   performing one of its functions.

Note that in order to ask the logger to log a statement at particular logging
level, you use the logging method that has the same name as the logging level,
but with Java's method naming convention:

1. TRACE -> `logger.trace("tracing message")`
2. DEBUG -> `logger.debug("debugging message")`
3. INFO -> `logger.info("informational message")`
4. WARN -> `logger.warn("warning message")`
5. ERROR -> `logger.error("error message")`
