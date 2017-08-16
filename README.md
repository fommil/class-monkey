| Windows | GNU / Linux | OS X |
|---------|-------------|------|
| [![Build status](https://ci.appveyor.com/api/projects/status/9if66b9ymuddko8a?svg=true)](https://ci.appveyor.com/project/fommil/class-monkey) | [![Build Status](https://ci.fommil.com/api/badges/fommil/class-monkey/status.svg)](https://ci.fommil.com/fommil/class-monkey) | [![Build Status](https://travis-ci.org/fommil/class-monkey.svg?branch=master)](https://travis-ci.org/fommil/class-monkey) |

There is a (reasonably) well-known bug in the JVM, that the URL classloader does not release its jars

- [Classloaders Keeping JarFiles Open](http://management-platform.blogspot.co.uk/2009/01/classloaders-keeping-jar-files-open.html)
- [apache-cxf/JDKBugHacks.java](https://github.com/ancoron/apache-cxf/blob/master/common/common/src/main/java/org/apache/cxf/common/logging/JDKBugHacks.java)

the various attempts to workaround the bug are limited.

This project provides a monkey patch to `java.net.URLClassLoader` which fixes the problem at its heart: by avoiding the use of persistent `JarFile`s which hold onto file handles.

## How?

We use a [Java Agent](https://docs.oracle.com/javase/7/docs/api/java/lang/instrument/package-summary.html) to `Retransform` `java/net/URLClassLoader` so that all new instances of `sun/misc/URLClassPath` instead create our `fommil/URLClassPath` - a clean implementation of the same API.

A caveat is that the system `URLClassLoader` will use the `sun/misc` implementation, however any user-created `URLClassLoader`s will use the `fommil` implementation.

Whereas the `sun/misc` implementation is overly general in the kinds of URLs that it can load (it was designed for the [infobahn](https://en.wikipedia.org/wiki/Information_superhighway) and makes unsubstantiated readability / performance trade-offs), the `fommil` implementation is stateless and simple.

## Why?

I personally encounter problems in the context of Scala development:

- https://issues.scala-lang.org/browse/SI-9682
- https://github.com/sbt/sbt/issues/2496

and there is a somewhat related resource management bug in `scalac`:

- https://issues.scala-lang.org/browse/SI-9632

This manifests as a memory leak in `sbt` on all platforms, but on Windows it is particularly bad because the OS uses read-write locks on file handles and the file cannot be deleted, moved or changed. **A catastrophic consequence is that compilations silently fail on Windows when using `exportJars`.**

## Install

Download the latest stable release from

- https://repo1.maven.org/maven2/com/fommil/class-monkey/1.7.1/class-monkey-1.7.1-assembly.jar

and add it to your `java` process as `-javaagent:/path/to/class-monkey-1.7.1-assembly.jar`. You must not rename the file or it will not work.

Specifically for scala usage, you can add this flag to your `SBT_OPTS` or add this to your project's `.sbtopts`

```
-J-javaagent:/home/fommil/.sbt/class-monkey-1.7.1-assembly.jar
```
