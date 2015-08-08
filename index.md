---
title: KouInject
layout: default
---

# KouInject

KouInject is a simple but powerful dependency injection framework for Java, with the goal of being easy to use. The framework is built on the standard for dependency injection in Java ([JSR-330](http://jcp.org/en/jsr/detail?id=330)), with many enhancements.


## Latest news:

  * **20.04.2011:** KouInject v1.2 is released [Read more](news/#kouinject-v12-is-released).
  * **02.03.2011:** KouInject v1.1 is released [Read more](news/#kouinject-v11-is-released).
  * **20.12.2010:** KouInject v1.0 is released [Read more](news/#kouinject-v10-is-released).


## Features:

  * Configuration using annotations.
  * Classpath scanning for finding beans.
  * Injection into fields, methods and constructors, of any visibility.
  * Qualifiers for selecting between different beans of the same class or interface.
  * Two different bean scopes - singleton and prototype (new instance for each injection).
  * Providers for getting multiple instances of beans, or lazy-loading beans.
  * Factories for creating beans from classes that you can't annotate, or that require complex setup.
  * Profiles for loading different implementations of beans in different environments.
  * Collections for injecting all beans of a certain type.
  * Full support for generics.
  * And more :)


## Sneak peak

_This is just a quick preview of some of the capabilities of KouInject. Head over to the [user guide](userguide/1.2/) for all the details._

You need an injector to load the beans. It's created like this:

```java
Injector injector = new DefaultInjector("com.zoo.app");
```

The injector will scan the package _com.zoo.app_ and all sub-packages for beans. All beans annotated with `@Component` will be available for injection. Like this:

```java
@Component
public class MonkeyService {

}
```

To get `MonkeyService` injected somewhere you have to use `@Inject`:

```java
@Component
public class MonkeyController {

    @Inject
    private MonkeyService monkeyService;
}
```

Perhaps that `MonkeyController` also needs a `MonkeyValidator`, but the validator requires extra runtime configuration to be setup correctly. In that case, you can use a factory, like this:

```java
@Component
public class Factory {

    @Produces
    public MonkeyValidator createMonkeyValidator() {
        MonkeyValidator monkeyValidator = new MonkeyValidator();
        // Do advanced configuration ...
        return monkeyValidator;
    }
}
```

`MonkeyValidator` will be available for injection, just like `MonkeyService`.

Beans are loaded on demand, so nothing happens until you ask the injector for at least one bean. You would typically ask for a bean that initializes the application, like creating the user interface and starting services. Like this:

```java
ZooClient zooClient = injector.getBean(ZooClient.class);
```


## Requirements:

  * Java SE 6 ([http://java.oracle.com](http://java.oracle.com))


## Maven

KouInject is available in Maven Central Repository. The only thing you need to do to use KouInject in a Maven project is add the KouInject dependency.

Put this in the `<dependencies>` section of your pom.xml and you are ready to go:

```xml
<dependency>
  <groupId>net.usikkert.kouinject</groupId>
  <artifactId>kouinject</artifactId>
  <version>1.2</version>
</dependency>
```


### Using a snapshot version

To try the latest version of KouInject you can use a snapshot.

Put this in the `<repositories>` section of your pom.xml:

```xml
<repository>
  <id>sonatype-nexus-snapshots</id>
  <name>Sonatype Nexus Snapshot Repository</name>
  <url>http://oss.sonatype.org/content/repositories/snapshots</url>
</repository>
```

And then you can use `1.3-SNAPSHOT` as the version, instead of `1.2` like shown above.
