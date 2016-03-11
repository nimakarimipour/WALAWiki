# Tips and Tricks

Sometimes running WALA for quick and accurate results takes some finesse. These tips and tricks should help.

## Performance Tips

### Heap Size

Use `-Xmx<n>M` to set the heap size to `n` megabytes. Use `-verbose:gc` to see if it looks like the VM is in the GC _region of discontent_.

Want to get an idea where memory is going? Check out the [HeapTracer](https://github.com/wala/WALA/blob/95fde985336f6e6d6c72a18418b7f82535544ea4/com.ibm.wala.util/src/com/ibm/wala/util/heapTrace/HeapTracer.java) utility. It's slow, but I find it useful.

### Libraries

Many analyses (e.g. call graph construction) may run dramatically faster when analyzing earlier versions (e.g. 1.4) of the JDK, since 5.0+ libraries introduce various sorts of pollution that we haven't tracked down yet. You may want to prototype your analysis against a 1.4 JDK, and then move up to a later version, perhaps using an Exclusions file later to recover lost performance.

### Exclusions File

WALA supports an XML-based **exclusions file**, which tells WALA to _ignore certain classes or packages_. Using an exclusions file can drastically speed up analysis, although of course introduces potential unsoundness.

For example, you can use the following code (from [CallGraphTestUtil](https://github.com/wala/WALA/blob/master/com.ibm.wala.core.tests/src/com/ibm/wala/core/tests/callGraph/CallGraphTestUtil.java)) to build an analysis scope that excludes classes based on an exclusions file.

    public static AnalysisScope makeJ2SEAnalysisScope(String scopeFile, String exclusionsFile) {
      AnalysisScope scope = AnalysisScopeReader.read(scopeFile, exclusionsFile, MY_CLASSLOADER);
      return scope;
    }

For a client that uses this, see the [CallGraphTest.testPrimordial()](https://github.com/wala/WALA/blob/95fde985336f6e6d6c72a18418b7f82535544ea4/com.ibm.wala.core.tests/src/com/ibm/wala/core/tests/callGraph/CallGraphTest.java#L246) unit test.

Here are the contents of [GUIExclusions.txt](https://github.com/wala/WALA/blob/master/com.ibm.wala.core.tests/dat/GUIExclusions.txt), an exclusions file which tells WALA to ignore [AWT](http://java.sun.com/products/jdk/awt/)-related classes. When using these exclusions, WALA pretends that classes from `java.awt` don't exist.

    java\/awt\/.*
    javax\/swing\/.*
    sun\/awt\/.*
    sun\/swing\/.*

### Other Issues

Something seem too slow? [Let us know](mailto:wala-wala@lists.sourceforge.net) how to reproduce the problem. Maybe we'll fix it.