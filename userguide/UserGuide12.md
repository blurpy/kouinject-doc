---
title: How to use KouInject v1.2 (current release)
---


# About KouInject

KouInject is a [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) framework based on the specification [JSR-330: Dependency injection for Java](http://jcp.org/en/jsr/detail?id=330).

JSR-330 gives you a standard way of annotating injections in your beans (objects handled by the framework), which should make your application more portable between different dependency injection frameworks. However the specification does not define how to configure the beans themselves. KouInject uses annotations and classpath scanning for configuration, but could just as well have used XML or a plain Java API.


# What's new

Changes since v1.1:

  * [Added support for custom @Components](#Custom_@Components.md)
  * [Added support for profiles with @Profile](#Profiles.md)
  * [Added support for generics](#Generics.md)
  * [Improved support for running with a SecurityManager](#SecurityManager.md)


# Getting started

The best way to get started is to read the [JSR-330 API](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/package-summary.html). That should give you an understanding of the capabilities of the framework. The [KouInject API](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/index.html) is also available, but most of the API is not meant for public use. The public part of the API is shown in this guide. If you want to see complete examples you should check out the [KouInject example code](http://kouinject.googlecode.com/svn/examples/kouinject-1.2-examples/).


## The injector

To use KouInject you need to create an instance of an [Injector](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/Injector.html). There is only one implementation, and that is the [DefaultInjector](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/DefaultInjector.html). It's used like this:

```
package some.basepackage;

import net.usikkert.kouinject.DefaultInjector;
import net.usikkert.kouinject.Injector;

public class ApplicationMain {

    public static void main(String[] args) {
        Injector injector = new DefaultInjector("some.basepackage");
    }
}
```

The injector will now scan the classpath for any classes marked with the annotation [@Component](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/annotation/Component.html) in the package `some.basepackage` and any sub-packages.

Here is an example of a bean in its simplest form:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

}
```

Beans are created on demand. So even though the injector scans `SomeBean`, it's not instantiated until you ask for it, like this:

```
SomeBean someBean = injector.getBean(SomeBean.class);
```


### Custom @Components

You can create your own annotations to use instead of the default `@Component` if you like. This gives you a few advantages:

  * You get to annotate your beans with names that are more meaningful. Like `@Model`, `@View`, or `@Controller`.
  * You can combine `@Component` and `@Profile` in one annotation for increased conciseness.
  * Improved portability, as you can use the same annotation across different frameworks.

A custom `@Component` is defined like this:

```
package some.basepackage;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import net.usikkert.kouinject.annotation.Component;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Component
public @interface Controller {

}
```

The usage of `@Component` can then be replaced on your controller beans with `@Controller` instead.


### Multiple base packages

If using a single base package is too limiting, or you want more control over which classes are scanned, then you can specify each package manually:

```
Injector injector = new DefaultInjector("some.basepackage",
                                        "some.different.basepackage",
                                        "a.third.basepackage");
```

You can specify an unlimited number of packages this way. Sub-packages of each base package is also scanned. It's also possible to use a string array of base packages, instead of varargs.


## Injections

The [@Inject](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Inject.html) annotation is used to mark where to inject in beans. Constructors, fields and methods are supported, and injection will happen in that order. If a bean inherits from another class, then the members in the superclass is injected first.


### Constructor injection

If you want to inject parameters into a constructor you must mark the constructor using the `@Inject` annotation. Any number of parameters can be injected, but at most one constructor can be marked for injection. If no constructor is marked, then the (parameterless) default constructor is used.

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeOtherBean {

}
```

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

    @Inject
    public SomeBean(SomeOtherBean someOtherBean) {

    }
}
```


### Field injection

Any number of fields can be injected, as long as they are marked with `@Inject`.
The fields can be of any visibility, but not `static` or `final`.

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

    @Inject
    private SomeOtherBean someOtherBean;
}
```


### Method injection

Any number of methods can be injected with any number of parameters, as long as the methods are marked with `@Inject`. The methods can be of any visibility, but not `static`, can be named anything, and can have a return value (that will be ignored).

The annotation is not inherited in overridden methods, so if a method marked for injection in a superclass is overridden by a method not marked for injection, the method will not be injected.

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

    @Inject
    public void setSomeOtherBean(SomeOtherBean someOtherBean) {
        
    }
}
```


## Qualifiers

Sometimes you need to create different implementations of an interface that you need to use for different purposes in the application. With a qualifier you can choose between different implementations of a bean to inject using an annotation instead of specifying the exact class. The qualifier is specified both on the bean to inject, and on the field or parameter to inject into. A bean can have zero or one qualifier, and the same with an injection.

When an injection has a qualifier then only implementations with the exact same qualifier is considered. When an injection does not have a qualifier then only implementations without a qualifier is considered, with the exception being if the implementation is the same class as the class requested in the injection.

The [@Qualifier](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Qualifier.html) annotation can not be used directly, but may be used either through the included [@Named](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Named.html) annotation, or by creating your own custom annotations. Qualifiers are not inherited in overridden methods.


### Custom qualifiers

The recommended way to use qualifiers is to create custom qualifiers. A custom qualifier may look like this:

```
package some.basepackage;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import javax.inject.Qualifier;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Green {

}
```

This gives you a qualifier annotation `@Green` with the value `Green`. The advantage you get with using a custom annotation is that it's easy to find where it's used, and it's easier to refactor.

This is the bean to inject:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Green
@Component
public class GreenColor implements Color {

}
```

And this is where the injection occurs:

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class Car {

    @Inject
    @Green
    private Color color;
}
```


Or it may look like this if injecting into a constructor:

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class Car {

    @Inject
    public Car(@Green Color color) {

    }
}
```

Each parameter in the constructor may have it's own qualifier. Using qualifiers on methods works the same way.


### @Named

`@Named` is a string based qualifier. That means that you specify the qualifier as the value of the annotation, instead of the class name. The example from above would look like this using `@Named`:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Named("Green")
@Component
public class GreenColor implements Color {

}
```

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class Car {

    @Inject
    @Named("Green")
    private Color color;
}
```


## Scope

KouInject supports two scopes: `prototype` and `singleton`. It is not possible to define new scopes. Scope is not inherited.


### Prototype

The default scope is prototype. If you do not declare the scope of a bean it will be registered with the prototype scope. This scope creates a new instance of a bean on each request. All beans using a bean with prototype scope will get unique instances.


### Singleton

Singleton scope creates a new instance of a bean on first request, and then caches the instance for later requests. All beans using a bean with singleton scope will get the same instance.

To set the scope to singleton on a bean, use the [@Singleton](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Singleton.html) annotation.

```
package some.basepackage;

import javax.inject.Singleton;
import net.usikkert.kouinject.annotation.Component;

@Singleton
@Component
public class SomeBean {

}
```


## Providers

[Provider](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Provider.html) is an interface you can inject with a bean as a generic type argument.

It's easier to explain with an example:

```
package some.basepackage;

import javax.inject.Inject;
import javax.inject.Provider;
import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

    @Inject
    public void setSomeOtherBean(Provider<SomeOtherBean> someOtherBeanProvider) {
        SomeOtherBean someOtherBean = someOtherBeanProvider.get();
    }
}
```

Here you can see that a provider with `SomeOtherBean` as generic argument is injected, and that the provider can be used to get instances of `SomeOtherBean`. If `SomeOtherBean` uses the prototype scope then several unique instances can be retrieved by calling `get()` for each. Provider may also be useful for lazy-loading beans, or breaking circular dependencies, and may be injected in constructors, fields and methods just like regular beans.


## Collections

If you have several implementations of an interface you might want to inject all of them at the same time. This is possible by injecting a collection of beans. Only the `Collection` interface is accepted. `List`, `Set` and `Queue` is unsupported.

The same rules for qualifiers apply to collections as well. So if you specify a qualifier on the collection then only beans having that qualifier will be injected. If you don't specify a qualifier then only beans without a qualifier (or is of the exact same class) will be injected. If this is too restrictive, you can use the special [@Any](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/annotation/Any.html) qualifier on the collection. `@Any` effectively means that the qualifier check is ignored, so you get all matching bean implementations even if they have a qualifier or not.

Example:

```
package some.basepackage;

import javax.inject.Named;
import net.usikkert.kouinject.annotation.Component;

@Named("casual")
@Component
public class CasualRadioListener implements RadioListener {}
```

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class RegularRadioListener implements RadioListener {}
```

```
package some.basepackage;

import java.util.Collection;
import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Any;
import net.usikkert.kouinject.annotation.Component;

@Component
public class Radio {

    @Inject @Any
    private Collection<RadioListener> radioListeners;

    public Collection<RadioListener> getRadioListeners() {
        return radioListeners;
    }
}
```

Here we can see one listener with a qualifier, and one without. To get an instance of both, the `@Any` qualifier is used when injecting the collection.


## CollectionProviders

[CollectionProvider](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/CollectionProvider.html) is like a regular `Provider`, except it returns a collection of beans instead of a single bean. Useful if you need to lazy-load beans or need multiple instances of the beans in the collection.

Example:

```
package some.basepackage;

import java.util.Collection;
import javax.inject.Inject;
import net.usikkert.kouinject.CollectionProvider;
import net.usikkert.kouinject.annotation.Any;
import net.usikkert.kouinject.annotation.Component;

@Component
public class LazyRadio {

    @Inject
    public LazyRadio(@Any CollectionProvider<RadioListener> radioListenersCollectionProvider) {
        Collection<RadioListener> radioListeners = radioListenersCollectionProvider.get();
    }
}
```

The result of this injection after calling `get()` is exactly the same as shown in the Radio example above. Qualifiers and scope works like normal.


## Factories

The [@Produces](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/annotation/Produces.html) annotation is used to mark factory methods in beans. The functionality is based on the [JSR-299](http://docs.jboss.org/weld/reference/1.0.0/en-US/html/producermethods.html) specification, but adapted to match the capabilities of a JSR-330 based injector.

A factory method (also called a factory point) is simply a method in a bean that has the `@Produces` annotation, and a return value. The method may be of any visibility, but can not be static, or overridden (as annotations aren't inherited), and must never return `null`. The method may also specify the scope and qualifier of the bean it creates instances of, using annotations.

The injector will treat beans created by factories the same way as any other beans, meaning that a factory method will be called whenever a new bean with matching class and qualifier is requested for injection. Any parameters on the factory method will be injected, just like regular method injection. The returned bean can be used in a `Provider`, `Collection`, `CollectionProvider` or by itself. There are no restrictions on the number of factory methods in a single bean.

Compared to annotating beans with `@Component`, using a factory gives you the ability to:

  * Create beans based on runtime configuration.
  * Create different instances of the same bean with different qualifiers and scope.
  * Create instances of classes that you can't modify (like a JDK class).
  * Create beans that require additional setup beyond running the constructor.

Example:

```
package some.basepackage;

import java.util.Date;
import javax.inject.Named;
import javax.inject.Singleton;
import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Produces;

@Component
public class Factory {

    @Produces @Singleton @Named("startup")
    public Date createStartupDate() {
        return new Date();
    }
}
```

```
package some.basepackage;

import java.util.Date;
import javax.inject.Inject;
import javax.inject.Named;
import net.usikkert.kouinject.annotation.Component;

@Component
public class DateUser {

    @Inject @Named("startup")
    private Date startupDate;
}
```

We see that the factory creates a singleton `Date` object with the qualifier "startup", and that another bean injects that `Date`.


### A note on Collections

Usually when injecting into a `Collection` the injector will lookup all the beans matching the generic parameter. Injecting `Collection<Animal>` will get all the beans implementing `Animal` (assuming `Animal` is an interface). If a factory returning `Collection<Animal>` is detected however, then that factory will be used instead of the built-in `Collection` support.


### FactoryContext

[FactoryContext](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/factory/FactoryContext.html) is an interface that can be injected into a factory method to get the qualifier of the parameter or field that requested the injection. This is useful when combined with the [@Any](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/annotation/Any.html) qualifier.

Using `@Any` on the factory method effectively means that the qualifier check is ignored, so that the factory method will be asked to produce bean instances for all requested injections matching only the return class of the method.

Example:

```
package some.basepackage;

import java.util.Properties;
import net.usikkert.kouinject.annotation.Any;
import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Produces;
import net.usikkert.kouinject.factory.FactoryContext;

@Component
public class StringPropertyFactory {

    @Produces @Any
    public String getProperty(FactoryContext factoryContext) {
        String qualifier = factoryContext.getQualifier();
        Properties properties = loadPropertyFile("database.properties"); // Not included in example

        return properties.getProperty(qualifier);
    }
}
```

```
package some.basepackage;

import javax.inject.Inject;
import javax.inject.Named;
import net.usikkert.kouinject.annotation.Component;

@Component
public class DatabaseConnection {

    @Inject @Named("database.url")
    private String url;

    @Inject @Named("database.username")
    private String username;

    @Inject @Named("database.password")
    private String password;
}
```

Here we have a factory method that injects the `FactoryContext`, and returns a property based on the qualifier on the fields in the `DatabaseConnection` bean.

Note that injecting a `Collection` or `CollectionProvider` with the `@Any` qualifier will not include beans created from factory methods with the `@Any` qualifier.


## Profiles

With profiles you can enable and disable certain beans at runtime. This is useful in several use-cases. For example when:

  * Using different beans for configuring resources in different environments.
  * Only enabling certain functionality based on requirements of the environment or capabilities of the platform.
  * Selecting between different modes, like running with a gui, as a console application, or as a Windows service or Unix daemon.

The functionality is based on profiles in [Spring 3.1](http://blog.springsource.com/2011/02/14/spring-3-1-m1-introducing-profile/).

To start using profiles you have to annotate the beans with the profiles you want them to have, and then tell the injector which profiles you want to be active. The [@Profile](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/annotation/Profile.html) annotation is used to create profiles. An example is seen here:

```
package some.basepackage;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import net.usikkert.kouinject.annotation.Profile;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Profile
public @interface Production {

}
```

Profiles are annotations annotated with `@Profile`. The name of the annotation becomes the name of the profile, in this case "production" (case insensitive). Example of using the profile:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
@Production
public class FtpFileStorage implements FileStorage {

}
```

The `FtpFileStorage` bean is now part of the production profile. If the production profile is active then this bean will be available for injection when we request a `FileStorage` bean.
During development we might not care about storing files on FTP. Instead we want to store files in a local temporary directory, and create an alternative `FileStorage` bean:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
@Development
public class TemporaryDirectoryFileStorage implements FileStorage {

}
```

When the "development" profile is active we will get an instance of this `TemporaryDirectoryFileStorage` bean instead.

Any number of beans may be part of a profile, and any number of profiles may be used on a single bean. Having several profiles on a bean means it will be available when any (not all) of those profiles are active. Beans without any profiles are always active. Profiles are not inherited.
To choose which profiles are active you have to use an alternative constructor on the `DefaultInjector`, which accepts a collection of active profiles in addition to the packages to scan.

Example:

```
package some.basepackage;

import java.util.Arrays;
import net.usikkert.kouinject.DefaultInjector;
import net.usikkert.kouinject.Injector;

public class ApplicationMain {

    public static void main(String[] args) {
        Injector injector = new DefaultInjector(Arrays.asList("production", "swing"), "some.basepackage");
    }
}
```

The profiles "production" and "swing" are active profiles, and "some.basepackage" is the package to scan for beans.
To make activating profiles more dynamic you could get the list from a property with a couple of lines of code:

```
String activeProfilesProperty = System.getProperty("com.something.active.profiles");
List<String> activeProfiles = Arrays.asList(activeProfilesProperty.split(","));
```

To set the active profiles just pass the parameter to the application on startup using:

```
java -Dcom.something.active.profiles=production,swing -jar YourApplication.jar
```


## Assignability

Assignability describes the relationship between a point of injection and a bean. A point of injection can be a field, or a parameter in a constructor or a method, with `@Inject`. A bean is an instance of a class with `@Component`, or an instance from a factory method with `@Produces`.

In it's simplest form it can look like this:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class Apple {

}
```

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class FruitBasket {

    @Inject
    private Apple apple;
}
```

The injection point here is `Apple`, and the bean to inject is also `Apple`. The injector knows that these two are a match because the class of the injection point is assignable from the class of the bean.

An injection point is also assignable from a subclass or an implemented interface. Example:

```
package some.basepackage;

public interface Fruit {

}
```

```
package some.basepackage;

public abstract class RoundFruit implements Fruit {

}
```

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class Apple extends RoundFruit {

}
```

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class FruitBasket {

    @Inject
    private Apple apple;

    @Inject
    private RoundFruit roundFruit;

    @Inject
    private Fruit fruit;
}
```

  * `Apple` is assignable from `Apple`.
  * `RoundFruit` is assignable from `RoundFruit` and `Apple`.
  * `Fruit` is assignable from `Fruit`, `RoundFruit` and `Apple`.

In the example above, only `Apple` is a bean, so an instance of `Apple` will be injected in all three fields.


### Arrays

Arrays follow the same assignability rules as regular classes, as long as they have the same dimensions.

Example:

  * `Apple[]` is assignable from `Apple[]`.
  * `RoundFruit[]` is assignable from `RoundFruit[]` and `Apple[]`.
  * `Fruit[]` is assignable from `Fruit[]`, `RoundFruit[]` and `Apple[]`.

However, these are not valid:

  * `Apple[]` is **not** assignable from `Apple[][]`.
  * `Apple[][]` is **not** assignable from `Apple[]`.

Array beans can only be created from factories. Be carefull with arrays and inheritance though, as they are not as type safe (at compile time) as generics are.


### Generics

Generics have their own special rules when it comes to assignability. Lets look at an example of a couple of generic classes:

```
package some.basepackage;

public interface Box<T> {

}
```

```
package some.basepackage;

public class PlasticBox<T> implements Box<T> {

}
```

These classes can be used like this:

```
Box<Apple> boxOfApple = new PlasticBox<Apple>();
```

This works fine. `Box` is assignable from `PlasticBox`.
This, however, does not even compile:

```
PlasticBox<Fruit> boxOfFruit = new PlasticBox<Apple>();
```

One would think that since `Fruit` is assignable from `Apple`, then `PlasticBox<Fruit>` is assignable from `PlasticBox<Apple>`. That is not the case, and the injector will only see a match where the generic class (Box, PlasticBox) of the injection point is assignable from the generic class of the bean, and the parameters of both are exactly the same.

Here is an example of these classes being used by the injector:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class AppleBox extends PlasticBox<Apple> {

}
```

```
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class StorageRoom {

    @Inject
    private AppleBox appleBox;

    @Inject
    private PlasticBox<Apple> plasticBoxOfApple;

    @Inject
    private Box<Apple> boxOfApple;

    @Inject
    private Box<? extends Fruit> boxOfFruit;
}
```

These are all legal injection points, and they all get an instance of `AppleBox`. The last injection point is using a wildcard, which means that it accepts any kind of `Box` as long as it contains a subclass of `Fruit`.

Generics can also be used in factories. The subclass `AppleBox` could be replaced by the following factory:

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Produces;

@Component
public class BoxFactory {

    @Produces
    public PlasticBox<Apple> createBoxOfApple() {
        return new PlasticBox<Apple>();
    }
}
```

The `StorageRoom` bean would then get an instance from `createBoxOfApple()` for the last three injection points. The first would fail, as `AppleBox` no longer exists.

Other notes on generics:

  * A factory point is allowed to use wildcards in the return type.
  * A generic class is allowed to use the type parameters (`<T>`) in injection points and factory points.
  * An injection point where a generic class is used with parameters is not assignable from a bean where it is used without parameters. `Box<Apple>` may not inject `Box`.
  * The opposite is however valid. `Box` may inject `Box<Apple>`.
  * A generic class is allowed to use an array as a parameter. This is valid: `Box<Apple[]>`.
  * A generic class can itself not be an array. This is not valid: `Box<Apple>[]`.


#### Collections and Providers

The built-in support for `Collection`, `Provider` and `CollectionProvider` behave a bit different because they are wrappers around beans, instead of actual beans.

`PlasticBox<Apple>` is a bean that can be injected standalone, or in any of the mentioned wrappers. Example:

```
package some.basepackage;

import java.util.Collection;
import javax.inject.Inject;
import javax.inject.Provider;
import net.usikkert.kouinject.annotation.Component;

@Component
public class AlternativeStorageRoom {

    @Inject
    private PlasticBox<Apple> plasticBoxOfApple;

    @Inject
    private Provider<PlasticBox<Apple>> plasticBoxOfAppleProvider;

    @Inject
    private Collection<PlasticBox<Apple>> plasticBoxOfAppleCollection;
}
```

The lookup of `PlasticBox<Apple>` is performed the exact same way in all three injection points, even though two of them are wrapped by a `Provider` or a `Collection`.


#### TypeLiteral

Because of a limitation in the language, it is not possible to get the class reference to a generic class with parameters.
That means you can **not** do this to get a generic bean directly from the injector:

```
Box<Apple> boxOfApple = injector.getBean(Box<Apple>.class);
```

As a workaround, the injector offers an alternative approach, where you use the class [TypeLiteral](http://kouinject.googlecode.com/svn/javadoc/kouinject-1.2/net/usikkert/kouinject/generics/TypeLiteral.html) instead:

```
Box<Apple> boxOfApple = injector.getBean(new TypeLiteral<Box<Apple>>() {});
```

Note the extra curly braces, they are required. When you use `TypeLiteral` like this, you are actually creating a subclass of `TypeLiteral`.


# SecurityManager

When used with a security manager you need to give KouInject access to read classes to detect beans, and access to allow injection into non-public members. You can give that access by modifying the policy file used by the application.

Here is an example of how to do that with a standalone application. Assuming you have a file structure like this:

  * YourApplication.jar
  * config/YourApplication.policy
  * libs/kouinject.jar (+ other required libraries)

And `config/YourApplication.policy` looks like this:

```
grant codebase "file:${user.dir}/libs/kouinject.jar" {
  permission java.io.FilePermission "${user.dir}/YourApplication.jar", "read";
  permission java.lang.reflect.ReflectPermission "suppressAccessChecks";
};
```

Then you can run your application with a security manager like this:

```
java -Djava.security.manager -Djava.security.policy=config/YourApplication.policy -jar YourApplication.jar
```

If your beans are not getting detected you may have configured the policy file wrong. To see what is going on you can enable logging from the security manager by adding the argument "`-Djava.security.debug=access,failure`" to the command as well.


# Runtime dependencies

You need these jar-files on the classpath to use KouInject:

  * [Apache Commons Lang v2.6](http://commons.apache.org/lang/)
  * [javax.inject v1](http://atinject.googlecode.com/)

If you use Maven then this is handled automatically for you.


# Upgrading from v1.1

Upgrading from v1.1 should be straight forward, unless you have used generic classes in an invalid way by taking advantage of the fact that KouInject did not care about generics earlier.

An overview of changes is available here: [CHANGES](http://kouinject.googlecode.com/svn/tags/kouinject-1.2/CHANGES)
