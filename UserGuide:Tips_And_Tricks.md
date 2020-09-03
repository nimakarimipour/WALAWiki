---
title: UserGuide:Tips And Tricks
permalink: /UserGuide:Tips_And_Tricks/
---

Performance Tips
----------------

### Heap Size

Use `-Xmx`<n>`M` to set the heap size to `n` megabytes. Use
`-verbose:gc` to see if it looks like the VM is in the GC *region of
discontent*.

Want to get an idea where memory is going? Check out the
[HeapTracer](http://wala.svn.sourceforge.net/viewvc/wala/trunk/com.ibm.wala.core/src/com/ibm/wala/util/heapTrace/HeapTracer.java?view=markup)
utility. It's slow, but I find it useful.

### Libraries

Many analyses (e.g. call graph construction) may run dramatically faster
when analyzing earlier versions (e.g. 1.4) of the JDK, since 5.0+
libraries introduce various sorts of pollution that we haven't tracked
down yet. You may want to prototype your analysis against a 1.4 JDK, and
then move up to a later version, perhaps using an Exclusions file later
to recover lost performance.

### Exclusions File

WALA supports an XML-based **exclusions file**, which tells WALA to
*ignore certain classes or packages*. Using an exclusions file can
drastically speed up analysis, although of course introduces potential
unsoundness.

For example, you can use the following code (from
[`CallGraphTestUtil`](http://wala.svn.sourceforge.net/viewvc/wala/trunk/com.ibm.wala.core.tests/src/com/ibm/wala/core/tests/callGraph/CallGraphTestUtil.java?view=log))
to build an analysis scope that excludes classes based on an exclusions
file.

` public static AnalysisScope makeJ2SEAnalysisScope(String scopeFile, String exclusionsFile) {`
`   AnalysisScope scope = AnalysisScopeReader.read(scopeFile, exclusionsFile, MY_CLASSLOADER);`
`   return scope;`
` }`

For a client that uses this, see the
\[<http://wala.sourceforge.net/javadocs/com.ibm.wala.core.tests/com/ibm/wala/core/tests/callGraph/CallGraphTest.html#testPrimordial>()
`CallGraphTest.testPrimordial()`\] unit test.

Here are the contents of
[GUIExclusions.txt](http://wala.svn.sourceforge.net/viewvc/wala/trunk/com.ibm.wala.core.tests/dat/GUIExclusions.txt?view=log),
an exclusions file which tells WALA to ignore
[AWT](http://java.sun.com/products/jdk/awt/)-related classes. When using
these exclusions, WALA pretends that classes from `java.awt` don't
exist.

`  java\/awt\/.*`
`  javax\/swing\/.*`
`  sun\/awt\/.*`
`  sun\/swing\/.*`

### Other Issues

Something seem too slow? [Let
us](mailto:wala-wala@lists.sourceforge.net) know how to reproduce the
problem. Maybe we'll fix it.