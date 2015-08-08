---
title: KouInject - News
layout: default
---

# News


## 20.04.2011 - KouInject v1.2 is released

New in this release:

  * Profiles
  * Custom @Component annotations
  * Full support for generics
  * Improved compatibility with security managers

**Profiles**. I started wondering a while back about how I could get KouInject to handle different setups for different environments. Like when you have a production environment with all its complexities, and a development environment where you try to keep it as simple as possible. Maybe you want to mock out a mail service, or any other integration service.

With `v0.7` came support for multiple base packages, allowing you to potentially cherry pick beans by separating beans for different environments into their own packages. Not the most elegant solution, but still possible. With `v1.1` came support for factories. A factory could create different instances of beans based on runtime configuration. That was a bit more elegant, but still not quite what I wanted for this use case. Then I saw the blog post  [introducing @Profile](http://blog.springsource.com/2011/02/14/spring-3-1-m1-introducing-profile/) for Spring 3.1. That was exactly what I needed. So I "borrowed" the idea, and it's available now, in `v1.2`. And that's before Spring 3.1 has been even released ;)

To use profiles, you create your own annotations that are based on `@Profile`, and annotate your beans with them. And then you select the active profiles when initializing the injector. Beans with no profiles, and beans with an active profile will be loaded. Simple as that.

**Custom @Component annotations** allows you to annotate beans with more meaningful names, like `@Controller`, `@Service` and so on. Create any names you like, or keep using `@Component`.

**Full support for generics**. The injector now knows the difference between `List<Monkey>` and `List<Beer>`, can handle wildcards, and resolves `<T>` (as long as it's resolvable from a subclass). Generics can be used as a return type in a factory method (`public Basket<Fruit> createFruitBasket()`), or using inheritance on classes (`public class FruitBasket implements Basket<Fruit>`), and injected anywhere, including providers and collections.

I actually thought that adding support for generics would be quite simple. Oh, was I wrong. It seems generics has an infinite amount of complexity and possibilities. I have never spent so much time on a single feature before. And I still don't know if I have covered every possibility. Probably not, but I'm very happy with what I've got now. Please create an issue if you stumble upon something that does not work like you would expect.

**Improved compatibility with security managers**. The injector now isolates its requirements for security permissions. That means you can give KouInject the permissions it needs, without also giving those permissions to the client code.

Hope you enjoy this release! I'm taking a break again. So tired of generics...

Check the [user guide]({{ site.baseurl }}/userguide/1.2/) for more examples and details.

-- Christian Ihle



---



## 02.03.2011 - KouInject v1.1 is released

For this release I have added support for factory methods. With a factory method you can create instances of beans that are difficult to setup automatically. For example if you want to expose a JDK class as a bean, a property from a property-file, or beans that require additional setup after injecting the dependencies before it's ready.

A factory method is defined by using the `@Produces` annotation. The implementation is inspired by [Producer Methods](http://docs.jboss.org/weld/reference/1.0.0/en-US/html/producermethods.html) in JSR-299.

Check the [user guide]({{ site.baseurl }}/userguide/1.1/) for more details.

-- Christian Ihle



---



## 20.12.2010 - KouInject v1.0 is released

Here it is! The magic version 1.0 :)

What's so special about v1.0 you may wonder? It's been over a year of testing, tweaking and improvements since the "pre-release" called v0.5, and now I believe KouInject is ready for prime time.

The hot new feature in this release is support for `CollectionProvider`. This is like a `Provider`, except it works with collections. Check the [user guide]({{ site.baseurl }}/userguide/1.0/) for more details.

Merry Xmas everybody! I'm taking some time off now.

-- Christian Ihle



---



## 12.12.2010 - KouInject v0.7 is released

KouInject v0.7 is now available.

There are two new features since last time. You can now inject collections of beans, and you can scan multiple base packages for beans. Check the [user guide]({{ site.baseurl }}/userguide/0.7/) for more details.

I was originally going to wait until [issue 3](https://github.com/blurpy/kouinject/issues/3) was done and release it as v1.0. But then a user wanted to know when [issue 8](https://github.com/blurpy/kouinject/issues/8) would be completed, so I decided to add this feature and make another release first. It's been a while since v0.6, so it was about time for a release anyway.

Enjoy!

-- Christian Ihle



---



## 13.06.2010 - KouInject v0.6 is finally released

I made KouInject v0.6 available for download yesterday :)
Both for Maven users, and here on the download page. Go get it while it's hot!

The new feature list shouldn't come as a surprise - it's mostly the transition to the JSR-330 API. Check out the [user guide]({{ site.baseurl }}/userguide/0.6/), which has been updated with examples of the new features.

-- Christian Ihle



---



## 24.05.2010 - KouInject v0.6 kind of done

KouInject v0.6 has passed the JSR-330 TCK since 20.02.2010. The reason v0.6 has not been released yet is that I'm waiting for the TCK to be added to the maven central repository. I can't put v0.6 into maven central before all dependencies are there as well, and the TCK is the only missing dependency. The JIRA ticket is here: [MAVENUPLOAD-2738](http://jira.codehaus.org/browse/MAVENUPLOAD-2738).

While I'm waiting I have deployed a snapshot of KouInject v0.6 to Nexus. Check the [user guide]({{ site.baseurl }}/userguide/0.6/) for details on how to use the snapshot.

-- Christian Ihle



---



## 14.02.2010 - KouInject v0.6 to implement JSR-330

If you're interested in dependency injection you might have heard about [JSR-330](http://jcp.org/en/jsr/detail?id=330),
or _Dependency Injection for Java_ as it's also called. This specification tries to standardize the way dependency injection works across different implementations, both in JavaSE and JavaEE. Take a look at [atinject](http://atinject.googlecode.com/) for details.

I've decided to implement that specification in KouInject v0.6. The specification comes with a pretty good test suite (or Technology Compatibility Kit), and at the time of writing I'm failing 8 out of 61 tests. To get this far I had to add support for the [Provider](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Provider.html) interface, [Qualifiers](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Qualifier.html) and inheritance. What's missing now is [Scope](http://atinject.googlecode.com/svn/tags/1/javadoc/javax/inject/Scope.html) and some tweaking to the way inheritance is handled.

I'm looking forward to finishing the rest so that KouInject can become an official JSR-330 implementation.

-- Christian Ihle



---



## 19.01.2010 - KouInject added to Maven Central Repository

Good news for Maven users! KouInject v0.5 is now available in the main Maven repository.

This is thanks to [Sonatype](http://www.sonatype.com/), which offers free repository hosting for open source projects in their repository manager [Nexus](http://nexus.sonatype.org/). The repository even synchronizes with Maven Central, so any releases added to Nexus is added to Maven Central as well. Excellent :)

Check the [user guide]({{ site.baseurl }}/userguide/0.5/) for details on how to use KouInject in your Maven project.


-- Christian Ihle



---



## 15.11.2009 - Version 0.5 - the first public release is out

So, dependency injection huh? Yep!

The idea came to me when I was thinking about using [Spring](http://www.springsource.org/) in [KouChat](http://www.kouchat.net/). But I didn't like the thought of bundling 20MB of dependencies with the 450KB application. There are other options of course, but I still like keeping KouChat in one small executable jar file.

Creating my own framework inside KouChat would make that possible, and keep me entertained for a while. After a period of working on this code I noticed a few things. First, it worked! Second, it was completely self contained. Third, the code for testing the framework grew quite alot.

These points were enough to convince me to create a new project for the framework.

The first release is basic, but fully usable. You can use annotations to create beans and mark constructors, fields and methods for dependency injection. All beans become singletons.

Planned for later releases are using annotations to mark methods that run on initialization and on shutdown for better life-cycle control. And support for a different life-cycle for beans, called the prototype-scope. This means that a bean is not a singleton, and a new unique instance of the bean will be injected on each request for that bean.

I'm considering supporting circular dependencies in beans if it's in methods or fields. And I'm wondering how to handle initializing Swing GUIs, since it's advised using the event dispatcher thread for that.

But, we'll see. The goal is to create a framework I can use in KouChat. And when that happens I will release version 1.0.


-- Christian Ihle
