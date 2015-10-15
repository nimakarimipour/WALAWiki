This is a barebones technical overview of the Shrike toolkit for reading, modifying, and writing Java bytecodes.

Key Features
------------

Shrike has a patch-based API for instrumenting bytecode. The bytecodes of a method are represented with an array of immutable [instruction] objects (see [MethodData]). Bytecode modification is done by adding patches to a [MethodEditor] and then applying those patches. (Applying the patches mutates the underlying [MethodData] object to have a new instruction array.) Shrike automatically takes care of updating branch targets and exception handler ranges to account for the applied patches. The key advantage of this approach is that patches are all applied to the <em>original</em> method bytecodes in one pass—the problem of inadvertently adding instrumentation to other instrumentation is avoided.

Shrike is also designed to be very efficient. In particular:

-   If only some parts of a class are being modified, the unmodified methods and class data can be copied directly to the instrumented output class without any parsing.
-   Parsing bytecode into arrays of [IInstruction][instruction] objects is fast and space-efficient; since the instructions are immutable, many constant instructions can be represented with a single object shared between methods.
-   Other key operations like patch application have been tuned for performance.

Other features of note include:

-   JSRs are automatically inlined, so instrumentation passes can ignore them.
-   Exception handlers are represented as an explicit handler list for each instruction (rather than ranges of indices covered by handler), easing manipulation.
-   Instrumented methods larger than the 64k JVM limit are automatically split up.
-   Simple dataflow analysis and a [bytecode verifier] for sanity checking of instrumented code.

Key Classes
-----------

Key [“ShrikeCT”] classes for reading and writing .class files:

-   [ClassReader]: provides an immutable view of .class file data. Reads the data lazily.
-   [ClassWriter]: generates the JVM representation of a class.

Key “ShrikeBT” classes for manipulating bytecodes:

-   [MethodData]: information about a method
-   [MethodEditor]: core class for transforming bytecodes via patches
-   [ClassInstrumenter]: convenient way to instrument an existing class. You can grab methods via a ClassReader from [ClassInstrumenter#getReader()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html#getReader()), instrument them, and then call [ClassInstrumenter#emitClass()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html#emitClass()) to obtain a ClassWriter populated with the modified class.
-   [CTCompiler]: for compiling a ShrikeBT method into bytecodes. Useful if you're creating your own method from scratch. Also see `CTUtils.compileAndAddMethodToClassWriter()` and `MethodData.makeWithDefaultHandlersAndInstToBytecodes()`.

Examples
--------

Check out the [com.ibm.wala.shrike.Bench] class for a basic example of instrumenting existing methods. [Dila] has some examples of creating new methods, fields, etc.; see, e.g., `com.ibm.wala.dila.instrumentation.cg.CallGraphInstrumentation.instrument()`. (Note that Dila's use of Shrike is not optimized for performance; in particular, it generates the JVM representation of the instrumented class more times than necessary.)

  [“ShrikeCT”]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/package-summary.html
  [ClassReader]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/ClassReader.html
  [ClassWriter]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeCT/ClassWriter.html
  [MethodData]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodData.html
  [MethodEditor]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodEditor.html
  [ClassInstrumenter]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/ClassInstrumenter.html
  [CTCompiler]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/shrikeCT/CTCompiler.html
  [com.ibm.wala.shrike.Bench]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrike/bench/Bench.html
  [Dila]: GettingStarted:wala.dila "wikilink"
  [instruction]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/IInstruction.html
  [MethodData]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodData.html
  [MethodEditor]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/MethodEditor.html
  [bytecode verifier]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/shrikeBT/analysis/Verifier.html