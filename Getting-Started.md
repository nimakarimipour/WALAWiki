This page steps through the process to download, build, and run a core subset of WALA for standard Java analysis. Later we will describe other available components, in addition to this core subset.

Quick Start using Maven Central packages
----------------------------------------

The quickest way to start with WALA is to get the [packages from Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.ibm.wala%22) using your build tool of choice (Maven, Gradle, sbt, etc.).  For an example using Gradle, see the [WALA-start](https://github.com/wala/WALA-start) project, which also contains some example analyses and drivers.

Note that there are many code examples that only appear in `test` packages in the WALA repository, and you still need to clone the repo to get those.  We are working on adding more examples to WALA-start so this becomes unnecessary.

Also, if you want to use the WALA Dalvik front-end or Scandroid from Maven Central, you need to add the repository `https://raw.github.com/msridhar/maven-jars/mvn-repo/` to your build file, to get a third-party jar that is not yet in Maven Central (this is already done in WALA-start).  We're also working on fixing this.

For now, the remainder of this document describes how to pull WALA source from Github and build it.  As we migrate more examples to WALA-start, the need for building from source should become less and less.


Prerequisites
-------------

#### Prerequisites for WALA 1.3.5 and later

WALA relies on Java 7 to run. In order to compile all the test code from source, Java 8 is required.

Parts of the WALA framework rely on Eclipse, and in general development using WALA is likely to be easiest from Eclipse. However, [compiling WALA using Gradle](https://github.com/wala/WALA/blob/master/README-Gradle.md) is also supported. You are encouraged to begin exploring WALA from within a fresh Eclipse 3.7 or 4.2 workspace. You can download Eclipse from <http://www.eclipse.org/downloads/>. WALA is packaged as a bunch of Eclipse plug-ins, so you will need a version of Eclipse that includes the Plugin Development Environment (PDE). To get PDE out of the box, you can download the "Eclipse for RCP and RAP Developers" version. Or, you can download any of the Eclipse variants for Java development and then install PDE from the Eclipse Marketplace, as described [here](http://stackoverflow.com/a/21058382/1126796). Also, if you'd like to use the support for integrating with [Eclipse JSDT](http://www.eclipse.org/webtools/jsdt/) (the `com.ibm.wala.ide.jsdt.*` projects), you'll need to install the JSDT plugin from the repository for your Eclipse release.

Getting the code
----------------

WALA has moved its main source control to [GitHub](http://www.github.com), and source access via Git and Subversion are supported.

#### Access via Git

From the command-line, simply run:

`git clone https://github.com/wala/WALA.git`

This provides you with the latest WALA code. If you prefer, you can stick to some release by checking out the appropriate tag, e.g. (after cloning):

`git checkout R_1.3.5`

#### Getting WALA 1.3.3 and earlier

WALA releases 1.3.3 and earlier are archived in the Sourceforge Subversion repository at this URL:

`https://wala.svn.sourceforge.net/svnroot/wala/tags`

WALA 1.3.4 is available both in the Sourceforge and GitHub repositories.

Building the code
-----------------

To get started, follow [the Gradle build instructions](https://github.com/wala/WALA/blob/master/README-Gradle.md) for either [building in Eclipse](https://github.com/wala/WALA/blob/master/README-Gradle.md#eclipse), [building in IntelliJ IDEA](https://github.com/wala/WALA/blob/master/README-Gradle.md#intellij-idea), or [building using Gradle on the command line](https://github.com/wala/WALA/blob/master/README-Gradle.md#gradle-command-line). The first build may take a while, as it must download many dependencies.

Configuring WALA properties
---------------------------

You will need to set up a few Java properties files before you can run the WALA code.

In the `com.ibm.wala.core` project, you need to copy the file [`dat/wala.properties.sample`](https://github.com/wala/WALA/tree/master/com.ibm.wala.core/dat/wala.properties.sample?view=markup) to `dat/wala.properties`. You need to then edit `wala.properties` to reflect your environment. See the properties file for the detailed instructions on what properties you must set. For beginners, we recommend you set the `java_runtime_dir` property (which is mandatory) and the `output_dir` property (required for some tests below like `PDFTypeHierarchy`). Note that the directory specified for `output_dir` must exist on the filesystem; WALA will not create it.

In the `com.ibm.wala.core.tests` project, you need to copy the file [`dat/wala.examples.properties.sample`](https://github.com/wala/WALA/tree/master/com.ibm.wala.core.tests/dat/wala.examples.properties.sample?view=markup) to `dat/wala.examples.properties`. For the moment, there are no mandatory settings in this file that you need to worry about, so move on. We will describe a few of the other optional properties shortly.

Note that on Windows all paths must be specified using '/' and not '\\'!

Running WALA Example programs
-----------------------------

We will now step through a few example programs, which will analyze the JLex program from Princeton University.  The `JLex.jar` file should be present in `com.ibm.wala.core.testdata` if you ran an Eclipse, IntelliJ IDEA, or Gradle command-line build successfully as recommended above.

-   **Tip**: We recommend you use the [Eclipse Launcher](http://wala.sourceforge.net/wiki/index.php/Eclipse:Launcher) we have provided for each example program. If you create your own launch configuration, be sure to specify an adequate heap size, such as 800MB via VM argument `-Xmx800MB`.

### Example 1: [SWTTypeHierarchy](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/SWTTypeHierarchy.html)

Our first example program will do the following:

1.  Invoke WALA to build a Java type hierarchy
2.  Spawn an [SWT](http://www.eclipse.org/swt/) [TreeViewer](http://help.eclipse.org/help30/index.jsp?topic=/org.eclipse.platform.doc.isv/reference/api/org/eclipse/jface/viewers/TreeViewer.html) to visualize the type hierarchy

Use the [Eclipse Launcher](http://wala.sourceforge.net/wiki/index.php/Eclipse:Launcher) `SWTTypeHierarchy`, found within `com.ibm.wala.ide.tests`. View and edit launchers via Eclipe's `Run -> Run Configurations...` drop-down menu. The launcher should already be listed under "Java Applications" in the list of launchers. Click the "Run" button to run the program; this should just work if you copied JLex.jar to the workspace as indicated above. If successful, you should see a new window pop up with a tree view of the class hierarchy of JLex.

Problems? See [Troubleshooting](http://wala.sourceforge.net/wiki/index.php/UserGuide:Troubleshooting).

### Example 2: [PDFTypeHierarchy](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/PDFTypeHierarchy.html)

This example builds a Java type hierarchy, renders a vizualization of the tree using [dot](http://www.graphviz.org/), and visualizes it using a PDF viewer.

To run this example, first install the AT&T `dot` tool from <http://www.graphviz.org> . Also install a PDF viewer if you do not already have one.

Next, edit the `com.ibm.wala.core.tests/dat/wala.examples.properties` file to have the correct paths to the `dot` and PDF viewer executables; see suggestions in the file.

Now run the `PDFTypeHierarchy` [Eclipse Launcher](http://wala.sourceforge.net/wiki/index.php/Eclipse:Launcher). This program should soon launch a viewer for a PDF file representing the type hierarchy.

Problems? See [Troubleshooting](http://wala.sourceforge.net/wiki/index.php/UserGuide:Troubleshooting).

-   **Tip**: dot will choke on large graphs. Don't do it.

### Other basic examples

The `com.ibm.wala.core.tests` project contains a number of other simple driver programs, in the package [`com.ibm.wala.examples.drivers`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/package-frame.html). You should now be able to figure out how to run any of them, using the steps documented above in Examples 1-2.

As of this writing, in addition to Examples 1-2, available example drivers include:

-   [`ClassPrinter`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/tools/ClassPrinter.html) : the WALA (Shrike) equivalent of `javap` (in the shrike project)
-   [`PDFCallGraph`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/PDFCallGraph.html): builds a call graph and shows a PDF represention generated with dot
-   [`PDFWalaIR`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/PDFWalaIR.html): builds a WALA IR for a method and shows a PDF representation generated with dot
-   [`SWTCallGraph`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/SWTCallGraph.html): builds a call graph and spawns an SWT TreeViewer to browse it
-   [`SWTPointsTo`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/SWTPointsTo.html): performs pointer analysis and spawns an SWT TreeViewer to browse the results

Getting started with WALA CAst, the front end for multiple source languages
---------------------------------------------------------------------------

WALA now includes the WALA Common Abstract Syntax Tree (CAst) System, a front end for generating IR from program source. Currently there are front ends for Java and JavaScript on the WALA site. Both use third-party libraries to actually parse the source, from which CAst ASTs are generated.

Due to these dependencies and the fact that we cannot distribute these prerequisites directly, there are a couple of build steps needed to get CAst working for you. For both front ends, you must obtain the core CAst projects below from the `trunk` of WALA's subversion; you also need the WALA core projects described above. You can omit all the test projects, but this is highly discouraged.

- `com.ibm.wala.cast` -- The core CAst System machinery
- `com.ibm.wala.cast.test` -- Test support for the CAst

### The Java Source Front End

The Java front end has two implementations: one makes use of [`polyglot`](http://www.cs.cornell.edu/projects/polyglot/) to parse Java and generate Abstract Syntax Trees (ASTs); the other is based on the Eclipse JDT and was contributed by Evan Battaglia from Berkeley. For either one, you need the following Java source language support projects, along with the CAst core and WALA core projects.

- `com.ibm.wala.cast.java` -- The CAst-based Java front end
- `com.ibm.wala.cast.java.test` -- Tests for the CAst-based Java front end
- `com.ibm.wala.cast.java.test.data` -- Test data for the CAst-based Java front end

The test data project requires some steps to build. For legal reasons, we do not include some test program source in the project, so you must obtain that separately. We do provide some support scripts to help. The following ought to work:

1) 'cd' to the root directory of the com.ibm.wala.cast.java.test.data project

2) run ant (or right-click on the build.xml from Eclipse and select "Run as Ant Build." The build file should export the entire project as a zip file to com.ibm.wala.ide.jdt.test/testdata/test_project.zip.

The Eclipse JDT based front end is in the projects given below. Note that due to the architecture of Eclipse, you currently can use the Eclipse ASTs that this front end needs only in the context of a running Eclipse. Thus, this front end is suitable only for plugins, i.e. for use in Eclipse-based applications. If you know how to get around this limitation, I would love to hear about it. We test this code with Eclipse 3.6 and 3.7.

Note that the regression tests in com.ibm.wala.cast.java.test run as JUnit Plugin tests. If you followed step 3 in the instructions above to export the zip file, the tests should be able to construct the needed workspace automatically. Hence, you should just be able to run them as plugin tests without any further setup.

- `com.ibm.wala.ide.jdt` -- The JDT- and CAst-based Java front end
- `com.ibm.wala.ide.jdt.test` -- Tests for the JDT front end

The Polyglot-based front end supports only up to Java 1.4, since that is what Polyglot handles. Hence, it is not suitable for more-recent Java applications that use Java 5 features such as generic types. (Java 6 is supported, but the new constructs in Java 7 are not.) This front end is in the projects below.

- `com.ibm.wala.cast.java.polyglot` -- The Polyglot- and CAst-based Java front end
- `com.ibm.wala.cast.java.polyglot.test` -- Tests for the Polyglot- and CAst-based Java front end

These Polyglot-based projects require libraries that are not included in the repository at this site for legal reasons. To obtain the libraries, you can just run ant on the build.xml file in com.ibm.wala.cast.java.polyglot, and it should fetch the polyglot source and build the necessary jars.

### The JavaScript Source Front End

For more details on the JavaScript front end, see [JavaScript frontend](http://wala.sourceforge.net/wiki/index.php/Getting_Started:JavaScript_frontend).

Getting started on a MAC
------------------------

On Mac OS X 10.4.9, JRE 5 is available by default. Standard Eclipse for Mac works fine. You need to install the SVN, JST, WST, etc. plugins to download and compile Wala. This is no different from Windows. In wala.properties, the Java runtime directory is at the following location:

` * java_runtime_dir = /System/Library/Frameworks/JavaVM.framework/Classes`

Otherwise, the general instructions above should work; let us know if they don't.

Various build.xml files in Wala distribution on sourceforge have Windows paths in them. One might need to generate a build.xml from the project's MANIFEST.MF file (right click and see the menu). *Let me know which ones and we can fix them*. (Sjfink)

Incubator projects
------------------

In addition to the functionality listed above, WALA has a number of *incubator* projects. These projects live in the `incubator` branch of subversion, and not `trunk`. Incubator projects are not-yet quite ready for prime time, and will likely be less stable, documented, or supported than the rest of WALA.

### Eclipse Plugin for WALA

-   If you would like to use WALA as a plugin within Eclipse, you should take a look at [our Eclipse integration incubator project](/GettingStarted:wala.eclipse "wikilink").

### Dynamic Load-time Instrumentation Library for Java Dila

-   Dila uses customized class loading for instrumenting the byte code of a program before it is executed. During the execution of an instrumented program dynamic program representations, such as call graphs, are created, which can be used for various program analyses. [Getting started with Dila](http://wala.sourceforge.net/wiki/index.php/GettingStarted:wala.dila).