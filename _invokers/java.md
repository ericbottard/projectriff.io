---
name: java
title: "Java Invoker"
author: projectriff
href: https://github.com/projectriff/java-function-invoker
development:
  href: https://github.com/projectriff/java-function-invoker/raw/master/java-invoker.yaml
releases:
- version: v0.0.5-sr.1
  href: https://github.com/projectriff/java-function-invoker/raw/v0.0.5-sr.1/java-invoker.yaml
  date: '2018-04-06'
- version: v0.0.6
  href: https://github.com/projectriff/java-function-invoker/raw/v0.0.6/java-invoker.yaml
  date: '2018-04-18'
- version: v0.0.7
  href: https://github.com/projectriff/java-function-invoker/raw/v0.0.7/java-invoker.yaml
  date: '2018-05-23'
---

*by [{{page.author}}]({{ page.href }})*

The Java Invoker hosts riff functions written in Java.

## Usage

With the `riff` CLI you need to provide an archive location with `-a`
and a handler specification with `--handler`. The archive is a jar
file, which can be shaded with all required dependencies, or it can be
a Spring Boot fat jar (with dependencies nested in
`BOOT-INF/lib`). Simple functions that do not require any dependencies
work just fine. The simplest form of the handler is a class name that
can be instantiated (with a default constructor). More complex creation 
scenarios can be handled by giving the handler in the form `<bean>[&main=<main>]` where

* `<bean>` is a class name or bean name, and
* `<main` is a Spring `@Configuration` class to create an application context

If you provide a `main` parameter you need to include Spring Boot and
all the other dependencies of the context in your archive. A Spring
Boot jar file does not need an explicit `main` (there is one in the
`MANIFEST.MF`), but if you supply one it will be used. If the
`Function` has POJO (i.e. not generic JDK classes) as parameter types,
then you also need to depend on `spring-cloud-function-context` (and
include that in the archive).

Example:

```
riff init java -i greetings -a target/greeter-1.0.0.jar --handler=functions.Greeter
```

Example with Spring Boot application:

```
riff init java -i greetings -a target/greeter-1.0.0.jar --handler=greeter
```

Example with Spring application context and an explicit main (Note: you need to add quotes around the handler value since it contains an `&`)

```
riff init java -i greetings -a target/greeter-1.0.0.jar --handler='greeter&main=functions.FunctionApp'
```

As long as the dependencies are included in the archive correctly, you
can supply a `Function` with a wide range of input and output
types. The input or output types can be plain JDK classes, or POJOs
defined in your archive, or `Message` (from `spring-messaging`) or
`Publisher` (from `reactive-streams`) or `Flux` or `Mono` (from
`reactor-core`). The `Message` type will give you access to header
metadata in the incoming and outgoing messages. If the input or output
is either a `Publisher` or a `Message` (or a `Publisher<Message>`)
then both input and output must be in the same form, with possibly
different payload types, obviously. POJOs are converted from incoming
messages assuming the payload is JSON ans using the GSON library.

## How it Works

The invoker is a [Spring Boot](https://projects.spring.io/spring-boot)
application with a configuration key `function.uri` that can be used
to point to a `java.util.function.Function`. Because of the way Spring
Boot works you can use an environment variable `FUNCTION_URI` or a
System property `function.uri` or a command line argument
`--function.uri` (amongst other options). Its value points to a
classpath archive, which can be a jar file or a directory, together
with parameters:

* `handler`: the class name of a `Function` to execute, or (when
  `main` is also used) a bean name of a `Function`. Can also be a
  comma, or pipe-separated list of functions, which are composed
  together at runtime.
* `main`: (optional) the class name of a Spring `@Configuration` that
  can be used to create a Spring application context, in which the
  `handler` is defined.

The jar archive can also be a comma-separated list of archive
locations, in case you need to compose things together.

Examples:

* A jar file

```
file:target/app.jar&handler=functions.Greeter
```

* A Spring app (with `spring-cloud-function-context`) and function specified by bean class

```
file:target/app.jar&handler=functions.Greeter&main=functions.Application
```

* A Spring app and function specified by bean name

```
file:target/app.jar&handler=greeter&main=functions.Application
```

* A Spring app split between 2 jar files

```
file:target/app.jar,file:target/lib.jar&handler=greeter&main=functions.Application
```
