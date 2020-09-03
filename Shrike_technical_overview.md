---
title: Shrike technical overview
permalink: /Shrike_technical_overview/
---

This is a barebones technical overview of the Shrike toolkit for
reading, modifying, and writing Java bytecodes.

Key Features
------------

Shrike has a patch-based API for instrumenting bytecode. The bytecodes
of a method are represented with an array of immutable
[instruction](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/IInstruction.html)
objects (see
[MethodData](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodData.html)).
Bytecode modification is done by adding patches to a
[MethodEditor](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodEditor.html)
and then applying those patches. (Applying the patches mutates the
underlying
[MethodData](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodData.html)
object to have a new instruction array.) Shrike automatically takes care
of updating branch targets and exception handler ranges to account for
the applied patches. The key advantage of this approach is that patches
are all applied to the <em>original</em> method bytecodes in one
pass—the problem of inadvertently adding instrumentation to other
instrumentation is avoided.

Shrike is also designed to be very efficient. In particular:

-   If only some parts of a class are being modified, the unmodified
    methods and class data can be copied directly to the instrumented
    output class without any parsing.
-   Parsing bytecode into arrays of
    [IInstruction](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/IInstruction.html)
    objects is fast and space-efficient; since the instructions are
    immutable, many constant instructions can be represented with a
    single object shared between methods.
-   Other key operations like patch application have been tuned for
    performance.

Other features of note include:

-   JSRs are automatically inlined, so instrumentation passes can ignore
    them.
-   Exception handlers are represented as an explicit handler list for
    each instruction (rather than ranges of indices covered by handler),
    easing manipulation.
-   Instrumented methods larger than the 64k JVM limit are automatically
    split up.
-   Simple dataflow analysis and a [bytecode
    verifier](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/analysis/Verifier.html)
    for sanity checking of instrumented code.

Key Classes
-----------

Key
["ShrikeCT"](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/package-summary.html)
classes for reading and writing .class files:

-   [ClassReader](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/ClassReader.html):
    provides an immutable view of .class file data. Reads the data
    lazily.
-   [ClassWriter](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/ClassWriter.html):
    generates the JVM representation of a class.

Key "ShrikeBT" classes for manipulating bytecodes:

-   [MethodData](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodData.html):
    information about a method
-   [MethodEditor](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodEditor.html):
    core class for transforming bytecodes via patches
-   [ClassInstrumenter](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html):
    convenient way to instrument an existing class. You can grab methods
    via a ClassReader from
    \[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html#getReader>()
    getReader()\], instrument them, and then call
    \[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html#emitClass>()
    emitClass()\] to obtain a ClassWriter populated with the modified
    class.
-   [CTCompiler](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/CTCompiler.html):
    for compiling a ShrikeBT method into bytecodes. Useful if you're
    creating your own method from scratch. Also see
    `CTUtils.compileAndAddMethodToClassWriter()` and
    `MethodData.makeWithDefaultHandlersAndInstToBytecodes()`.

Examples
--------

Check out the
[com.ibm.wala.shrike.Bench](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrike/bench/Bench.html)
class for a basic example of instrumenting existing methods.
[Dila](/GettingStarted:wala.dila "wikilink") has some examples of
creating new methods, fields, etc.; see, e.g.,
`com.ibm.wala.dila.instrumentation.cg.CallGraphInstrumentation.instrument()`.
(Note that Dila's use of Shrike is not optimized for performance; in
particular, it generates the JVM representation of the instrumented
class more times than necessary.)