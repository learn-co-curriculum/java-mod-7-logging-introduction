# Logging Introduction

## Learning Goals

- Explain the disadvantage to print statements.
- Explain why logging is important.
- Explain the different types of log levels.

## Why not `System.out`

While the `System.out` method has allowed us to very easily display text
information to our application's console, it has several important limitations:

- All print statements are the same, which means the only way we can make them
  conditional is by putting each statement in a conditional block. This presents
  the following disadvantages:
  - a) It is time-consuming.
  - b) It costs a lot to maintain as it would require us to change every single
    conditional statement every time we wanted to change the way our application
    prints information to the console.
- `System.out` print statements are printed directly to the console, but not all
  applications have an actual console that someone can read, and most
  importantly the console output scrolls over time and is generally not
  recoverable later.
- Print statements are "local", so their ability to scale with distributed
  systems is not existent

## Default logging in Spring

The alternative to `System.out` statements is to use a logging framework.
**Loggers** solve the disadvantages of using `System.out` considering they are
easily configurable and can be saved and viewed later on. The Spring Framework
comes with logging capabilities, and the Spring Boot bundles we have been using
gives us access to these logging libraries out of the box.

Consider our `JokeService` class that we worked with at the end of the last
module:

```java
package com.example.springrestdemo.service;

import com.example.springrestdemo.dto.JokeDTO;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class JokeService {

    private static final String JOKE_URL = "https://icanhazdadjoke.com/";
    private final RestTemplate restTemplate;
    
    @Autowired
    public JokeService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public JokeDTO getJoke() {
        // Request a joke from the JOKE_URI and maps it to the JokeDTO
        return restTemplate.getForObject(JOKE_URL, JokeDTO.class);
    }
}
```

We can add simple logging to our `JokeService` class as follows:

```java
package com.example.springrestdemo.service;

import com.example.springrestdemo.dto.JokeDTO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class JokeService {

    private static final String JOKE_URI = "https://icanhazdadjoke.com/";
    private final RestTemplate restTemplate;
    private final Logger logger = LoggerFactory.getLogger(JokeService.class);

    @Autowired
    public JokeService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public JokeDTO getJoke() {
        logger.info("Using logger: Retrieving the joke from the external source");
        System.out.println("Using System.out: Retrieving the joke from the external source");
        return restTemplate.getForObject(JOKE_URI, JokeDTO.class);
    }
}
```

Note that there might be several choices of packages to import when trying to
tell the IDE which `Logger` class to import. Here are the proper import
statements for the code above:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

Now let's break down this code:

1. "SLF4J" stands for "Simple Logging Facade for Java." It offers a generic API
   to abstract the actual underlying implementation.
2. In the default Spring configuration, the underlying implementation is
   Logback, which is a popular, full-featured and well-supported logging
   utility.
3. SLF4J comes with a factory that will return an instance of the `Logger` class
4. We request an instance of `Logger` by passing the `LoggerFactory` the name of
   the class for which we wish to log statements.
5. We then use the `logger.info()` method like we would use the
   `System.out.println()` method. More on the significance of the `info()`
   method in a bit.
6. We also include the same output using the `System.out.println()` method, so
   we can compare the two results.

Let's look at the output of the application when we send the GET request now
that we've added this log statement:

```text
2022-12-15 11:17:18.252  INFO 9424 --- [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-12-15 11:17:18.253  INFO 9424 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-12-15 11:17:18.254  INFO 9424 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
2022-12-15 11:17:18.405  INFO 9424 --- [nio-8080-exec-2] c.e.springrestdemo.service.JokeService    : Using logger: Retrieving the joke from the external source
Using System.out: Retrieving the joke from the external source
```

Notice the last two lines of the output above:

1. The logger automatically outputs the date and time on the server at the time
   that a given statement is output to the console: `2022-12-15 11:17:18.405`.
2. The logger also outputs the log level at which the statement is being
   displayed: `INFO`. This is related to the configuration options that the
   logger gives us, which allows us to control what statements get written out.
   More on this below.
3. Before the actual content of the `info()` statement is printed out, the
   logger prints out the class that the statement comes from:
   `springrestdemo.service.JokeService`. This allows us to trace back where in
   the code the statement is being logged and can be helpful for debugging
   purposes.
4. Compare that to the output of the `System.out.println()` and we can see that
   logging with the logger gives us much more contextual information, with very
   little additional code required.

Note that the `springrestdemo.service.JokeService` comes from the `class` that
was passed into the `LoggerFactory` when the `Logger` instance was requested.
We could pass in another class, and the corresponding statements would be
prepended with the name of that class, even though they actually came from a
different class at runtime. However, doing so is highly **not** recommended.

## Lombok Logging Annotation

Now that we understand a little more about what is happening, we can also use
the Lombok dependency again to help instantiate a logger for us!

The Lombok dependency comes with different annotations, depending on the logging
framework we are using. In our case, we are using the SLF4J logger library.
Therefore, we could add the `@Sl4j` annotation at the class level to generate
`private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(MyClass.class);`
at runtime.

Let's go ahead and rewrite the code above to use the `@Sl4j` annotation:

```java
package com.example.springrestdemo.service;

import com.example.springrestdemo.dto.JokeDTO;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
@Sl4j
public class JokeService {

    private static final String JOKE_URI = "https://icanhazdadjoke.com/";
    private final RestTemplate restTemplate;

    @Autowired
    public JokeService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public JokeDTO getJoke() {
        log.info("Using logger: Retrieving the joke from the external source");
        System.out.println("Using System.out: Retrieving the joke from the external source");
        return restTemplate.getForObject(JOKE_URI, JokeDTO.class);
    }
}
```

Notice that when using the `@Sl4j` annotation, our `Logger` instance is called
`log`.

## Log Levels

One of the great features of a logging framework is that it gives us access to
specific logging methods that correspond to specific logging levels.

**Logging levels** are a way for us to tell the logging framework _when_ we
want specific log statements to be displayed. The logging levels allow us to
control how verbose we want our application to be. The higher the logging level,
the less information it should output. This is by convention, so we are
responsible for calling the logger with a logging level that is appropriate for
the information we're trying to display.

When we use one of the logger's output methods (e.g. `logger.info()`), the
logger will only output the statement if the current log level is at the
corresponding level or lower. In our example, we're using the `info()` method,
so the logger will only output our message if its current logging level is
INFO or lower. We will go over how to configure the logger's logging level in
detail later in this section, but first let's look at all the logging levels
supported by SLF4J:

1. TRACE: This is the most fine-grained level of logging and should be used to
   display very detailed information about what the application is doing.
   Expect all code used by the application to be extremely verbose when this
   logging level is enabled.
2. DEBUG: This is one level up from trace, so it should be less fine-grained,
   but still show a more detailed output. This level is used to get high level
   debugging information that falls short of including every single step that
   the application is going through.
3. INFO: This is the level that SLF4J is configured to log at by default, and
   should include high level information about what the application is doing,
   e.g. a request was just processed, but not any of the details of how the
   result was achieved.
4. WARN: Used to indicate that something went wrong, but wasn't fatal to the
   application. This is a log level that is almost always enabled because it's
   not expected to be used frequently.
5. ERROR: Used to indicate an issue that is preventing the application from
   performing one of its functions.

Note that in order to ask the logger to log a statement at a particular logging
level, we can use the logging method that has the same name as the logging
level, but with Java's method naming convention:

1. TRACE -> `logger.trace("tracing message")`
2. DEBUG -> `logger.debug("debugging message")`
3. INFO -> `logger.info("informational message")`
4. WARN -> `logger.warn("warning message")`
5. ERROR -> `logger.error("error message")`

## Summary

Logging provides several advantages to `System.out.println()` statements. It
provides more contextual information with little additional code, like a
timestamp, log level, and an indication to what class wrote out the log message.
The logging framework also provides logging levels, so we can specify when
certain information should be written out to the log. In the next lesson, we'll
learn how to configure the log with these levels.

## References

-[Spring Logging Documentation](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-logging.html)
