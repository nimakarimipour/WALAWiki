**NOTE: the guide has only has only been partially migrated to the github wiki -- see [the original](http://wala.sourceforge.net/wiki/index.php/UserGuide:Getting_Started) for any missing links**

This page steps through the process to download, build, and run a core subset of WALA for standard Java analysis. Later we will describe other available components, in addition to this core subset.

Prerequisites
-------------

#### Prerequisites for WALA 1.3.5 and later

WALA relies on Java 7 to run. In order to compile all the test code from source, Java 8 is required.

Parts of the WALA framework rely on Eclipse, and in general development using WALA is likely to be easiest from Eclipse. However, compiling WALA using [Apache Maven](http://maven.apache.org/) is also supported. You are encouraged to begin exploring WALA from within a fresh Eclipse 3.7 or 4.2 workspace. You can download Eclipse from <http://www.eclipse.org/downloads/>. WALA is packaged as a bunch of Eclipse plug-ins, so you will need a version of Eclipse that includes the Plugin Development Environment (PDE). To get PDE out of the box, you can download the "Eclipse for RCP and RAP Developers" version. Or, you can download any of the Eclipse variants for Java development and then install PDE from the Eclipse Marketplace, as described [here](http://stackoverflow.com/a/21058382/1126796). Also, if you'd like to use the support for integrating with [Eclipse JSDT](http://www.eclipse.org/webtools/jsdt/) (the `com.ibm.wala.ide.jsdt.*` projects), you'll need to install the JSDT plugin from the repository for your Eclipse release.

- [[Prerequisites for earlier WALA versions]]

Getting the code
----------------

WALA has moved its main source control to [GitHub](http://www.github.com), and source access via Git and Subversion are supported.

#### Access via Git

From the command-line, simply run:

`git clone https://github.com/wala/WALA.git`

This provides you with the latest WALA code. If you prefer, you can stick to some release by checking out the appropriate tag, e.g. (after cloning):

`git checkout R_1.3.5`

See [[Accessing WALA Using Git]] for details on accessing WALA via Git from within Eclipse.

#### Access via Subversion

The root Subversion URL for the WALA repository is:

`https://github.com/wala/WALA  `

You can work out of the `trunk` branch, or stick with a previous release under `branches`.

We recommend you install SubClipse ([install instructions](http://subclipse.tigris.org/servlets/ProjectProcess?pageID=p4wYuA)) to provide Subversion support for Eclipse.

#### Getting WALA 1.3.3 and earlier

WALA releases 1.3.3 and earlier are archived in the Sourceforge Subversion repository at this URL:

`https://wala.svn.sourceforge.net/svnroot/wala/tags`

WALA 1.3.4 is available both in the Sourceforge and GitHub repositories.

Building the code
-----------------

To get started, we concentrate on a core subset of WALA for standard Java analysis:

- `com.ibm.wala.core` -- core WALA framework support
- `com.ibm.wala.shrike` -- Shrike bytecode manipulation library
- `com.ibm.wala.core.tests` -- basic WALA example programs
- `com.ibm.wala.ide` -- WALA support that relies on Eclipse IDE or SWT functionality
- `com.ibm.wala.ide.tests` -- tests for WALA support that relies on Eclipse IDE functionality
- `com.ibm.wala.util` -- general utilities (post WALA 1.3.2)

When you initially import the WALA projects into Eclipse, you may see some compile errors (due to missing external dependencies), but the above projects should compile, which is enough to get started.

To fix the above compile errors, or if you prefer building without Eclipse, you can use our maven build scripts. First, manually download dx.jar and place it in the com.ibm.wala.dalvik.test/lib folder, as described [here](https://groups.google.com/d/msg/wala-sourceforge-net/cBYsfEvYVG0/Ua52dyQQU-YJ). (Unfortunately, we cannot easily download this jar automatically.) Then, in your root directory (the parent directory where you cloned WALA, or your workspace root), run the following command to download external dependencies and compile all the WALA projects:

`mvn clean verify -DskipTests=true -q`

Be sure you have a recent version of Maven, at least version 3.0. You also need to have the `svn` executable in your path, as an `svn checkout` is performed during the build. The first Maven build may take a while, as it must download many dependencies. For Eclipse users, once the Maven build is finished, refresh / rebuild your workspace, and the compile errors should disappear. (If you didn't install the JSDT plugin, as described [[above|#Prerequisites for WALA 1.3.5 and later]], you may still see compile errors in the `com.ibm.wala.ide.jsdt.*` projects.)

Alternately, if you don't want to use Maven, there are instructions below for fixing the compile errors by running various Ant build.xml files.

Configuring WALA properties
---------------------------

You will need to set up a few Java properties files before you can run the WALA code.

In the `com.ibm.wala.core` project, you need to copy the file [`dat/wala.properties.sample`](https://github.com/wala/WALA/tree/master/com.ibm.wala.core/dat/wala.properties.sample?view=markup) to `dat/wala.properties`. You need to then edit `wala.properties` to reflect your environment. See the properties file for the detailed instructions on what properties you must set. For beginners, we recommend you set the `java_runtime_dir` property (which is mandatory) and the `output_dir` property (required for some tests below like `PDFTypeHierarchy`). Note that the directory specified for `output_dir` must exist on the filesystem; WALA will not create it.

In the `com.ibm.wala.core.tests` project, you need to copy the file [`dat/wala.examples.properties.sample`](https://github.com/wala/WALA/tree/master/com.ibm.wala.core.tests/dat/wala.examples.properties.sample?view=markup) to `dat/wala.examples.properties`. For the moment, there are no mandatory settings in this file that you need to worry about, so move on. We will describe a few of the other optional properties shortly.

Note that on Windows all paths must be specified using '/' and not '\\'!

Running WALA Example programs
-----------------------------

We will now step through a few example programs, which will analyze the JLex program from Princeton University. First, you will need a file `JLex.jar` holding the contents of this program:

1.  Download `Main.java` <http://www.cs.princeton.edu/~appel/modern/java/JLex/Archive/1.2.6/Main.java> to a directory named JLex. *Note that case matters ... the L is uppercase.*
2.  Compile the file: from the parent directory of JLex, run `javac JLex/Main.java` (ignore any compile warnings)
3.  Create a `JLex.jar` file: `jar cvf JLex.jar JLex/*.class` (again from parent of JLex director)
4.  Copy JLex.jar to the root of the Eclipse workspace containing WALA.

-   **Tip**: We recommend you use the [launchers](/Eclipse:Launcher "wikilink") we have provided for each example program. If you create your own launch configuration, be sure to specify an adequate heap size, such as 800MB via VM argument `-Xmx800MB`.

### Example 1: [SWTTypeHierarchy](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/SWTTypeHierarchy.html)

Our first example program will do the following:

1.  Invoke WALA to build a Java type hierarchy
2.  Spawn an [SWT](http://www.eclipse.org/swt/) [TreeViewer](http://help.eclipse.org/help30/index.jsp?topic=/org.eclipse.platform.doc.isv/reference/api/org/eclipse/jface/viewers/TreeViewer.html) to visualize the type hierarchy

Use the [launcher](/Eclipse:Launcher "wikilink") `SWTTypeHierarchy`, found within `com.ibm.wala.ide.tests`. View and edit launchers via Eclipe's `Run -> Run Configurations...` drop-down menu. The launcher should already be listed under "Java Applications" in the list of launchers. Click the "Run" button to run the program; this should just work if you copied JLex.jar to the workspace as indicated above. If successful, you should see a new window pop up with a tree view of the class hierarchy of JLex.

Problems? See [UserGuide:Troubleshooting](/UserGuide:Troubleshooting "wikilink").

### Example 2: [PDFTypeHierarchy](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/PDFTypeHierarchy.html)

This example builds a Java type hierarchy, renders a vizualization of the tree using [dot](http://www.graphviz.org/), and visualizes it using a PDF viewer.

To run this example, first install the AT&T `dot` tool from <http://www.graphviz.org> . Also install a PDF viewer if you do not already have one.

Next, edit the `com.ibm.wala.core.tests/dat/wala.examples.properties` file to have the correct paths to the `dot` and PDF viewer executables; see suggestions in the file.

Now run the `PDFTypeHierarchy` [launcher](/Eclipse:Launcher "wikilink"). This program should soon launch a viewer for a PDF file representing the type hierarchy.

Problems? See [UserGuide:Troubleshooting](/UserGuide:Troubleshooting "wikilink").

-   **Tip**: dot will choke on large graphs. Don't do it.

### Other basic examples

The `com.ibm.wala.core.tests` project contains a number of other simple driver programs, in the package [`com.ibm.wala.examples.drivers`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/package-frame.html). You should now be able to figure out how to run any of them, using the steps documented above in Examples 1-2.

As of this writing, in addition to Examples 1-2, available example drivers include:

-   [`ClassPrinter`](http://wala.sourceforge.net/javadocs/com.ibm.wala.shrike/com/ibm/wala/shrikeBT/shrikeCT/tools/ClassPrinter.html) : the WALA (Shrike) equivalent of `javap` (in the shrike project)
-   [`PDFCallGraph`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/PDFCallGraph.html): builds a call graph and shows a PDF represention generated with dot
-   [`PDFWalaIR`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/PDFWalaIR.html): builds a WALA IR for a method and shows a PDF representation generated with dot
-   [`SWTCallGraph`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/SWTCallGraph.html): builds a call graph and spawns an SWT TreeViewer to browse it
-   [`SWTPointsTo`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/examples/drivers/SWTPointsTo.html): performs pointer analysis and spawns an SWT TreeViewer to browse the results

Getting started with WALA CAst, the front end for multiple source languages
---------------------------------------------------------------------------

WALA now includes the WALA Common Abstract Syntax Tree (CAst) System, a front end for generating IR from program source. Currently there are front ends for Java and JavaScript on the WALA site. Both use third-party libraries to actually parse the source, from which CAst ASTs are generated.

Due to these dependencies and the fact that we cannot distribute these prerequisites directly, there are a couple of build steps needed to get CAst working for you. For both front ends, you must obtain the core CAst projects below from the `trunk` of WALA's subversion; you also need the WALA core projects described above. You can omit all the test projects, but this is highly discouraged.

` * com.ibm.wala.cast (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast)`)`
`   o The core CAst System machinery`
` * com.ibm.wala.cast.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.test)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.test)`)`
`   o Test support for the CAst`

### The Java Source Front End

The Java front end has two implementations: one makes use of [`polyglot`](http://www.cs.cornell.edu/projects/polyglot/) to parse Java and generate Abstract Syntax Trees (ASTs); the other is based on the Eclipse JDT and was contributed by Evan Battaglia from Berkeley. For either one, you need the following Java source language support projects, along with the CAst core and WALA core projects.

` * com.ibm.wala.cast.java (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.java)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.java)`)`
`   o The CAst-based Java front end`
` * com.ibm.wala.cast.java.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.java.test)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.java.test)`)`
`   o Tests for the CAst-based Java front end`
` * com.ibm.wala.cast.java.test.data (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.java.test.data)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.java.test.data)`)`
`   o Test data for the CAst-based Java front end`

The test data project requires some steps to build. For legal reasons, we do not include some test program source in the project, so you must obtain that separately. We do provide some support scripts to help. The following ought to work:

1) 'cd' to the root directory of the com.ibm.wala.cast.java.test.data project

2) run ant (or right-click on the build.xml from Eclipse and select "Run as Ant Build." The build file should export the entire project as a zip file to com.ibm.wala.ide.jdt.test/testdata/test_project.zip.

The Eclipse JDT based front end is in the projects given below. Note that due to the architecture of Eclipse, you currently can use the Eclipse ASTs that this front end needs only in the context of a running Eclipse. Thus, this front end is suitable only for plugins, i.e. for use in Eclipse-based applications. If you know how to get around this limitation, I would love to hear about it. We test this code with Eclipse 3.6 and 3.7.

Note that the regression tests in com.ibm.wala.cast.java.test run as JUnit Plugin tests. If you followed step 3 in the instructions above to export the zip file, the tests should be able to construct the needed workspace automatically. Hence, you should just be able to run them as plugin tests without any further setup.

` * com.ibm.wala.ide.jdt (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.ide.jdt)`)`
`   o The JDT- and CAst-based Java front end`
` * com.ibm.wala.ide.jdt.test (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.ide.jdt.test)`)`
`   o Tests for the JDT front end`

The Polyglot-based front end supports only up to Java 1.4, since that is what Polyglot handles. Hence, it is not suitable for more-recent Java applications that use Java 5 features such as generic types. (Java 6 is supported, but the new constructs in Java 7 are not.) This front end is in the projects below.

` * com.ibm.wala.cast.java.polyglot (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.java.polyglot)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.java.polyglot)`)`
`   o The Polyglot- and CAst-based Java front end`
` * com.ibm.wala.cast.java.polyglot.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.java.polyglot.test)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.java.polyglot.test)`)`
`   o Tests for the Polyglot- and CAst-based Java front end`

These Polyglot-based projects require libraries that are not included in the repository at this site for legal reasons. To obtain the libraries, you can just run ant on the build.xml file in com.ibm.wala.cast.java.polyglot, and it should fetch the polyglot source and build the necessary jars.

### The JavaScript Source Front End

For more details on the JavaScript front end, see [Getting Started:JavaScript frontend](/Getting_Started:JavaScript_frontend "wikilink").

Getting started on a MAC
------------------------

On Mac OS X 10.4.9, JRE 5 is available by default. Standard Eclipse for Mac works fine. You need to install the SVN, JST, WST, etc. plugins to download and compile Wala. This is no different from Windows. In wala.properties, the Java runtime directory is at the following location:

` * java_runtime_dir = /System/Library/Frameworks/JavaVM.framework/Classes`

Otherwise, the general instructions above should work; let us know if they don't.

Various build.xml files in Wala distribution on sourceforge have Windows paths in them. One might need to generate a build.xml from the project's MANIFEST.MF file (right click and see the menu). *Let me know which ones and we can fix them*. [Sjfink](/User:Sjfink "wikilink")

Incubator projects
------------------

In addition to the functionality listed above, WALA has a number of *incubator* projects. These projects live in the `incubator` branch of subversion, and not `trunk`. Incubator projects are not-yet quite ready for prime time, and will likely be less stable, documented, or supported than the rest of WALA.

### Eclipse Plugin for WALA

-   If you would like to use WALA as a plugin within Eclipse, you should take a look at [our Eclipse integration incubator project](/GettingStarted:wala.eclipse "wikilink").

### Dynamic Load-time Instrumentation Library for Java ([Dila](http://wala.sourceforge.net/wiki/index.php/GettingStarted:wala.dila))

-   Dila uses customized class loading for instrumenting the byte code of a program before it is executed. During the execution of an instrumented program dynamic program representations, such as call graphs, are created, which can be used for various program analyses. [Getting started with Dila.](http://wala.sourceforge.net/wiki/index.php/GettingStarted:wala.dila)