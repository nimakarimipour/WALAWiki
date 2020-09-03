---
title: UserGuide:AnalysisScope
permalink: /UserGuide:AnalysisScope/
---

[wala.core technical overview](/wala.core_technical_overview "wikilink")

An `AnalysisScope` specifies the application and library code to be
analyzed.
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/util/config/AnalysisScopeReader.html#makeJavaBinaryAnalysisScope(java.lang.String,%20java.io.File>)
`AnalysisScopeReader.makeJavaBinaryAnalysisScope()`\] constructs an
`AnalysisScope` given a Java classpath String and an exclusions file
(discussed further below). For more control, use the
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/util/config/AnalysisScopeReader.html#readJavaScope(java.lang.String,%20java.io.File,%20java.lang.ClassLoader>)
`AnalysisScopeReader.readJavaScope()`\] method, whose first parameter is
the name of a text <em>scope file</em> specifying the analysis scope. A
scope file has lines of the following format:

> <em>Classloader,Language,Type,Location</em>

For Java, <em>Classloader</em> is one of `Primordial`, `Extension`, or
`Application` (see [Naming Java
Entities](/UserGuide:NamingJavaEntities "wikilink")). `Primordial` is
reserved for the Java standard libraries. If you've set up your
`wala.properties` file correctly (see [Getting
Started](/UserGuide:Getting_Started "wikilink")), the following two
lines for `Primordial` should do the right thing:

`Primordial,Java,stdlib,none`
`Primordial,Java,jarFile,primordial.jar.model`

`Extension` entries should be used for other libraries used by the
application, while `Application` entries should be used for the
application code itself. Valid values of <em>Type</em> are:

-   `classFile` for a single `.class` file
-   `binaryDir` for a directory containing class files (with the
    standard package-to-sub-directory correspondence)
-   `jarFile` for a `.jar` file
-   `sourceFile` and `sourceDir` for source files (for use with a Java
    source frontend)

<em>Location</em> should give the appropriate filesystem path. Here's a
full example:

`Primordial,Java,stdlib,none`
`Primordial,Java,jarFile,primordial.jar.model`
`Extension,Java,jarFile,/workspace/myapp/lib/someLib.jar`
`Application,Java,binaryDir,/workspace/myApp/bin`