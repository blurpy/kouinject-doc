---
title: How to use KouInject v1.0
layout: default
permalink: userguide/1.0/
---


## About KouInject


KouInject is a [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) framework based on the specification [JSR-330: Dependency injection for Java](http://jcp.org/en/jsr/detail?id=330).

JSR-330 gives you a standard way of annotating injections in your beans (objects handled by the framework), which should make your application more portable between different dependency injection frameworks. However the specification does not define how to configure the beans themselves. KouInject uses annotations and classpath scanning for configuration, but could just as well have used XML or a plain Java API.


## Changelog

Changes since v0.7:

  * [Added CollectionProvider for lazy-loading of collections of beans](#collectionprovider)
  * Fix for singletons in some cases getting instantiated too many times when used with a qualifier
  * Improved logging


## Getting started

The best way to get started is to read the [JSR-330 API](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/package-summary.html). That should give you an understanding of the capabilities of the framework. The [KouInject API]({{ site.baseurl }}/javadoc/kouinject-1.0/index.html) is also available, but most of the API is not meant for public use. The public part of the API is shown in this guide. If you want to see complete examples you should check out the [KouInject example code](https://github.com/blurpy/kouinject-examples/tree/master/kouinject-1.0-examples).


To use KouInject you need to create an instance of an [Injector]({{ site.baseurl }}/javadoc/kouinject-1.0/net/usikkert/kouinject/Injector.html). There is only one implementation, and that is the [DefaultInjector]({{ site.baseurl }}/javadoc/kouinject-1.0/net/usikkert/kouinject/DefaultInjector.html). It's used like this:

```java
Injector injector = new DefaultInjector("some.basepackage");
```

The injector will now scan the classpath for any classes marked with the annotation [@Component]({{ site.baseurl }}/javadoc/kouinject-1.0/net/usikkert/kouinject/annotation/Component.html) in the package `some.basepackage` and any sub-packages.

Here is an example of a bean in its simplest form:

```java
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

}
```

Beans are created on demand. So even though the injector scans SomeBean, it's not instantiated until you ask for it, like this:

```java
SomeBean someBean = injector.getBean(SomeBean.class);
```


### Multiple base packages

If using a single base package is too limiting, or you want more control over which classes are scanned, then you can specify each package manually:

```java
Injector injector = new DefaultInjector("some.basepackage",
                                        "some.different.basepackage",
                                        "a.third.basepackage");
```

You can specify an unlimited number of packages this way. Sub-packages of each base package is also scanned. It's also possible to use a string array of base packages, instead of varargs.

**Tip**: If you have many base packages and want to avoid outdated paths or miss-spelled paths then you can get the packages from classes you know reside in them:

```java
Injector injector = new DefaultInjector(ClassInSomeBasePackage.class.getPackage().getName(),
                                        ClassInSomeDifferentBasePackage.class.getPackage().getName(),
                                        ClassInAThirdBasePackage.class.getPackage().getName());
```


### @Inject

The [@Inject](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Inject.html) annotation is used to mark where to inject in beans. Constructors, fields and methods are supported, and injection will happen in that order. If a bean inherits from another class, then the members in the superclass is injected first.


#### Constructor injection

If you want to inject parameters into a constructor you must mark the constructor using the `@Inject` annotation. Any number of parameters can be injected, but at most one constructor can be marked for injection. If no constructor is marked, then the (parameterless) default constructor is used.

```java
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


#### Field injection


Any number of fields can be injected, as long as they are marked with `@Inject`.
The fields can be of any visibility, but not `static` or `final`.

```java
package some.basepackage;

import javax.inject.Inject;
import net.usikkert.kouinject.annotation.Component;

@Component
public class SomeBean {

    @Inject
    private SomeOtherBean someOtherBean;
}
```


#### Method injection

Any number of methods can be injected with any number of parameters, as long as the methods are marked with `@Inject`. The methods can be of any visibility, but not `static`, can be named anything, and can have a return value (that will be ignored).

The annotation is not inherited in overridden methods, so if a method marked for injection in a superclass is overridden by a method not marked for injection, the method will not be injected.

```java
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


### @Qualifier

Sometimes you need to create different implementations of an interface that you need to use for different purposes in the application. With a qualifier you can choose between different implementations of a bean to inject using an annotation instead of specifying the exact class. The qualifier is specified both on the bean to inject, and on the field or parameter to inject into. A bean can have zero or one qualifier, and the same with an injection.

When an injection has a qualifier then only implementations with the exact same qualifier is considered. When an injection does not have a qualifier then only implementations without a qualifier is considered, with the exception being if the implementation is the same class as the class requested in the injection.

The [@Qualifier](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Qualifier.html) annotation can not be used directly, but may be used either through the included [@Named](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Named.html) annotation, or by creating your own custom annotations. Qualifiers are not inherited in overridden methods.


#### Custom qualifiers

The recommended way to use qualifiers is to create custom qualifiers. A custom qualifier may look like this:

```java
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

```java
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Green
@Component
public class GreenColor implements Color {

}
```

And this is where the injection occurs:

```java
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

```java
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


#### @Named

`@Named` is a string based qualifier. That means that you specify the qualifier as the value of the annotation, instead of the class name. The example from above would look like this using `@Named`:

```java
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Named("Green")
@Component
public class GreenColor implements Color {

}
```

```java
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


### @Scope

KouInject supports two scopes: `prototype` and `singleton`. It is not possible to define new scopes. Scope is not inherited.


#### Prototype

The default scope is prototype. If you do not declare the scope of a bean it will be registered with the prototype scope. This scope creates a new instance of a bean on each request. All beans using a bean with prototype scope will get unique instances.


#### Singleton

Singleton scope creates a new instance of a bean on first request, and then caches the instance for later requests. All beans using a bean with singleton scope will get the same instance.

To set the scope to singleton on a bean, use the [@Singleton](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Singleton.html) annotation.

```java
package some.basepackage;

import javax.inject.Singleton;
import net.usikkert.kouinject.annotation.Component;

@Singleton
@Component
public class SomeBean {

}
```


### Provider

[Provider](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Provider.html) is an interface you can inject with a bean as a generic type argument.

It's easier to explain with an example:

```java
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

Here you can see that a provider with SomeOtherBean as generic argument is injected, and that the provider can be used to get instances of SomeOtherBean. If SomeOtherBean uses the prototype scope then several unique instances can be retrieved by calling `get()` for each. Provider may also be useful for lazy-loading beans, or breaking circular dependencies, and may be injected in constructors, fields and methods just like regular beans.


### Collection

If you have several implementations of an interface you might want to inject all of them at the same time. This is possible by injecting a collection of beans. Only the collection interface is accepted. List, Set and Queue is unsupported.

The same rules for qualifiers apply to collections as well. So if you specify a qualifier on the collection then only beans having that qualifier will be injected. If you don't specify a qualifier then only beans without a qualifier (or is of the exact same class) will be injected. If this is too restrictive, you can use the special [@Any]({{ site.baseurl }}/javadoc/kouinject-1.0/net/usikkert/kouinject/annotation/Any.html) qualifier on the collection. `@Any` effectively means that the qualifier check is ignored, so you get all matching bean implementations even if they have a qualifier or not.

Example:

```java
package some.basepackage;

import javax.inject.Named;
import net.usikkert.kouinject.annotation.Component;

@Named("casual")
@Component
public class CasualRadioListener implements RadioListener {}
```

```java
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class RegularRadioListener implements RadioListener {}
```

```java
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


### CollectionProvider

[CollectionProvider]({{ site.baseurl }}/javadoc/kouinject-1.0/net/usikkert/kouinject/CollectionProvider.html) is like a regular [Provider](#provider), except it returns a [Collection](#collection) of beans instead of a single bean. Useful if you need to lazy-load beans or need multiple instances of the beans in the collection.

Example:

```java
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


## Runtime dependencies

You need these jar-files on the classpath to use KouInject:

  * [Apache Commons Lang v2.5](http://commons.apache.org/lang/)
  * [javax.inject v1](http://atinject.googlecode.com/)

If you use Maven then this is handled automatically for you.


## Upgrading from v0.7

There are no breaking changes to the official API described in the user guide for version 0.7, so upgrading should be straight forward.
