This is a streamlined guide to getting started with WALA, based on use of [Gradle](https://gradle.org/) and [the WALA jars in Maven Central](https://mvnrepository.com/artifact/com.ibm.wala).  With this approach, you never need to clone the WALA GitHub repository, and you can more easily use tools other than Eclipse for development.  This guide is a work in progress; feedback / edits welcome.

## Prerequisites

* [Git](https://git-scm.com/)
* [Java](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) version 7 or higher

On Mac OS, if you have [Homebrew](http://brew.sh/) installed, you can get the above dependencies by running:
```
brew cask install java
```
TODO add instructions for Ubuntu

## Getting the code

We are going to use the [WALA-start](https://github.com/msridhar/WALA-start) repository, which contains some sample analyses and an appropriate Gradle build file to automatically pull in the relevant WALA jars.  First, clone the repository:
```
git clone https://github.com/msridhar/WALA-start
```
Then, compile the Java code:
```
cd WALA-start
./gradlew compileJava
```
The first time you run this command, gradle will download the WALA jars and their dependencies, which will take some time.

That's basically it!  You can use the sample analyses under `src/main/java` to get started with your own analysis.  E.g., check out [`ScopeFileCallGraph`](https://github.com/msridhar/WALA-start/blob/master/src/main/java/com/ibm/wala/examples/analysis/ScopeFileCallGraph.java) to see how to build a call graph given a [scope file](https://github.com/wala/WALA/wiki/Analysis-Scope).

## Using Eclipse

If you'd like to use Eclipse for development, first run the following command in the `WALA-start` directory:
```
./gradlew eclipse
```
This generates the appropriate Eclipse metadata.  Once you've done that, you should be able to import `WALA-start` as an Eclipse project.
