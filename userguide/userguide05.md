---
title: How to use KouInject v0.5
layout: default
permalink: userguide/0.5/
---


## About KouInject

KouInject is a [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) framework.

Version 0.5 was the first public release, and was not based on any real specification. The API is deprecated, but still available here for those who need it.

  * Javadoc for the API is available here: [KouInject v0.5 API](http://kouinject.googlecode.com/svn/javadoc/kouinject-0.5/index.html)
  * Example code is available here: [KouInject v0.5 examples](http://kouinject.googlecode.com/svn/examples/kouinject-0.5-examples/)

Take a look at the code below for a full example that can run be without modification.


## Maven

KouInject is available in Maven Central Repository. The only thing you need to do to use KouInject in a Maven project is add the KouInject dependency.

Put this in the `<dependencies>` section of your pom.xml and you are ready to go:

```
<dependency>
  <groupId>net.usikkert.kouinject</groupId>
  <artifactId>kouinject</artifactId>
  <version>0.5</version>
</dependency>
```


## The container

The main component in the framework is the BeanLoader. The BeanLoader's responsibility is handling the life cycle of objects, and is called a container. An object in the container is called a bean.

KouInject works by scanning the classpath for annotated classes, then instantiates them one by one, and makes them available for use in other beans. A bean can not be instantiated before all the beans it depends on are also instantiated. The container makes sure the beans are created in the correct order. After all beans have been created then the application is ready for use.

To get the beans loaded you will have to tell the container about the base-package to use. The base-package is the top level package where the annotated classes are located. All classes in that package will be scanned, and all sub-packages as well. Any annotated class will become a bean.

```
package some;

import net.usikkert.kouinject.AnnotationBasedBeanDataHandler;
import net.usikkert.kouinject.BeanDataHandler;
import net.usikkert.kouinject.BeanLoader;
import net.usikkert.kouinject.ClassLocator;
import net.usikkert.kouinject.ClassPathScanner;
import net.usikkert.kouinject.DefaultBeanLoader;

public class MainClass {

    public static void main(String[] args) {
        ClassLocator classLocator = new ClassPathScanner();
        BeanDataHandler beanDataHandler = new AnnotationBasedBeanDataHandler("some.basepackage", classLocator);
        BeanLoader beanLoader = new DefaultBeanLoader(beanDataHandler);
        beanLoader.loadBeans();
    }
}
```


## To create a bean

In KouInject a bean is any class having the annotation `@Component`. Only those classes benefit from dependency injection. It is not possible to inject beans into anything other than beans managed by the container.

This class has no dependencies and will be instantiated by calling the default constructor. After the constructor has been called it is ready to use.

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;

@Component
public class FirstBean {

    public FirstBean() {
        System.out.println("FirstBean");
    }
}
```


## To mark a constructor for dependency injection

Sometimes it's useful to have all dependencies available in the constructor. This can be accomplished by creating a constructor with another bean as a parameter and marking the constructor with the `@Inject` annotation. The constructor can take any number of parameters. But all the parameters have to be other beans.

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Inject;

@Component
public class SecondBean {

    @Inject
    public SecondBean(FirstBean firstBean) {
        System.out.println("SecondBean");
    }
}
```


## To mark a field for dependency injection

If you do not want to expose a dependency, you can get the container to inject an instance of another bean directly into a field right after instantiation, without using a setter. Just annotate the field with the `@Inject` annotation. The field does not have to be public, and you can annotate as many fields as you like.

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Inject;

@Component
public class ThirdBean {

    @Inject
    private SecondBean secondBean;

    public ThirdBean() {
        System.out.println("ThirdBean");
    }
}
```

## To mark a method for dependency injection

If it's OK to expose the dependency then a setter injection can be useful. Just mark the setter with the `@Inject` annotation. The method can be called anything, and take any number of parameters. But all the parameters have to be other beans.

```
package some.basepackage;

import net.usikkert.kouinject.annotation.Component;
import net.usikkert.kouinject.annotation.Inject;

@Component
public class FourthBean {

    private ThirdBean thirdBean;

    public FourthBean() {
        System.out.println("FourthBean");
    }

    @Inject
    public void setThirdBean(ThirdBean thirdBean) {
        this.thirdBean = thirdBean;
    }
}
```
