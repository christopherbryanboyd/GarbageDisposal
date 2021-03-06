# GarbageDisposal ![GarbageDisposal](resources/recycle.png) 

**GarbageDisposal** is a Java library for registering [callbacks](https://en.wikipedia.org/wiki/Callback_(computer_programming)) when one or more specific objects are [Garbage Collected](https://www.cubrid.org/blog/understanding-java-garbage-collection), without incurring the [penalty](http://thefinestartist.com/effective-java/07) for implementing the [finalize](https://docs.oracle.com/javase/9/docs/api/java/lang/Object.html#finalize--) method. 
Usage of *finalize* is [discouraged](https://softwareengineering.stackexchange.com/questions/288715/is-overriding-object-finalize-really-bad), and now [deprecated as of Java 9](https://www.infoq.com/news/2017/03/Java-Finalize-Deprecated).

This library uses the [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern) to *decorate* an object, wrapping the specified callback in a [PhantomReference](https://docs.oracle.com/javase/9/docs/api/java/lang/ref/PhantomReference.html), and invoking the callback in its own thread. Optionally, you may specify an [ExecutorService](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/ExecutorService.html) to be used for the invocation of the callback.

Please see [here](https://stackoverflow.com/questions/2860121/why-do-finalizers-have-a-severe-performance-penalty) and [here](https://docs.oracle.com/javase/9/docs/api/java/lang/Object.html#finalize--) for more details on why it is problematic to implement the *finalize* method directly.

## Usage Examples

The standard usage pattern of [GarbageDisposal.java](src/main/java/club/wodencafe/decorators/GarbageDisposal.java) is to [decorate()](src/main/java/club/wodencafe/decorators/GarbageDisposal.java#L189) an object and provide a [Runnable](https://docs.oracle.com/javase/9/docs/api/java/lang/Runnable.html) callback:

```
import club.wodencafe.decorators.GarbageDisposal;
...
Object objectToWatch = new Object();
GarbageDisposal.decorate(objectToWatch, () -> 
    System.out.println("Object is eligible for Garbage Collection"));
```

This callback will be invoked when the [JVM Garbage Collection](https://www.dynatrace.com/resources/ebooks/javabook/how-garbage-collection-works/) cycle runs, and the object is [Phantom Reachable](https://docs.oracle.com/javase/7/docs/api/java/lang/ref/package-summary.html#reachability).

If for some reason you later decide to remove the callback, you may [undecorate()](/src/main/java/club/wodencafe/decorators/GarbageDisposal.java#L164
) the decorated object:

```
GarbageDisposal.undecorate(objectToWatch);
```

You can also use the [CompletableFuture](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/CompletableFuture.html) decorator methods to get a *CompletableFuture* handle:
```
Object objectToWatch = new Object();
GarbageDisposal.decorateAsync(objectToWatch).thenRunAsync(
    () -> System.out.println("Object is eligible for Garbage Collection"));
```
This *CompletableFuture* handle may be [cancelled](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/CompletableFuture.html#cancel-boolean-) if you choose, which will internally call undecorate.

## Getting Started

This project is in the process of being hosted on [Maven Central](https://search.maven.org/), when this is complete this artifact will be available and this section will be updated with the *Maven Coordinates*. 

### Maven
If you would like to start using this library in your [Maven](https://maven.apache.org/) projects, please add the following to your **pom.xml**:
```
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```
```
<!-- https://jitpack.io/#wodencafe/GarbageDisposal -->
<dependency>
    <groupId>com.github.wodencafe</groupId>
    <artifactId>GarbageDisposal</artifactId>
    <version>master-SNAPSHOT</version>
</dependency>
```

### Gradle
If you would like to start using this library in your [Gradle](https://gradle.org/) projects, please add the following to your **build.gradle**:
```
repositories {
    maven { url "https://jitpack.io" }
}
```
```
dependencies {
    // https://jitpack.io/#wodencafe/GarbageDisposal
    compile 'com.github.wodencafe:GarbageDisposal:master-SNAPSHOT'
}
```

For customizing and playing with the source for yourself, please see the **[Grab the source](#grab-the-source)** section.

## News
### 2018-01-06:
  * Version 0.3
  * Better logging, better background service for dequeing PhantomReferences.
  * Added additional decorator methods, utilizing CompletableFuture, and may be cancelled.
  * Added several new tests to test the new decorator methods.
  
### 2018-01-05:
  * Version 0.2
  * Added support for [undecorate()](/src/main/java/club/wodencafe/decorators/GarbageDisposal.java#L91).
  * Added license explicitly to individual source files.
            
### 2018-01-04: 
  * Version 0.1
  * GarbageDisposal GitHub Repository is created, initial commits.
  * Added initial support for hosting on [JitPack.io](https://jitpack.io/#wodencafe/GarbageDisposal).

## Grab the source

To grab a copy of this code for yourself, please run the following commands in your workspace or a directory of your choosing:
```
git clone https://github.com/wodencafe/GarbageDisposal
cd GarbageDisposal
./gradlew build
```

This will build the jar in:
`./build/libs/GarbageDisposal.jar`

You can then reference this jar for your own projects.

## Built With

* [Gradle](https://gradle.org/) - Dependency Management and Build System.
* [JitPack.io](https://jitpack.io/#wodencafe/GarbageDisposal) - Easy to use package repository for Git.
* [Guava](https://github.com/google/guava) - A useful a set of core libraries for Java, developed by Google.
* [SLF4J](https://www.slf4j.org/) - A simple facade or abstraction for various logging frameworks.
* [JUnit](http://junit.org) (**Unit Testing**) - The *de facto* unit testing framework for Java.
* [Awaitility](https://github.com/awaitility/awaitility) (**Unit Testing**) - A small Java DSL for synchronizing asynchronous operations.

## License

This project is licensed under the [BSD-3 License](https://opensource.org/licenses/BSD-3-Clause) - see the [LICENSE.md](LICENSE.md) file for details

## Further Reading
You can find more information about the deprecation of *Object.finalize()* [here](https://stuartmarks.wordpress.com/2017/04/17/deprecation-of-object-finalize/).
