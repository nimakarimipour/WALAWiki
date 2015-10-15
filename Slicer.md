WALA includes a slicer, based on context-sensitive tabulation of
reachability in the system dependence graph.

Driver
------

The
[`PDFSlice`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/PDFSlice.html)
[launcher](/Eclipse:Launcher "wikilink") provides a simple driver to
compute a slice and visualize it as a PDF.

API
---

To access the slicer programmatically, you will likely use
`computeForwardSlice` or `computeBackwardSlice` from the class `Slicer`.
These methods take as a parameter a `Statement` object from which to
slice. A `Statement` object identifies a node in the System Dependence
Graph (SDG). `Statement`s represent either instructions from the SSA IR
or extra nodes inserted to model parameter passing. To create a
statement representing a normal SSA IR instruction, use the
`NormalStatement` class; this class identifies an instruction via its
index into the IR `getInstructions()` array.

### Statement Types

`Statements` in a slice may be of the following types:

-   Normal statements represent instructions from the SSA IR.
-   PARAM_CALLER and PARAM_CALLEE statements are extra nodes inserted
    to model parameter passing. When a value flows from a caller to a
    callee via parameter passing, it flows from an SSA variable in the
    caller, to a PARAM_CALLER statement on the caller side, to a
    PARAM_CALLEE statement on the callee side, to a statement on the
    callee side.
-   RETURN_CALLER and RETURN_CALLEE statements provide analogous
    functionality for values that flow from callee to caller via a
    return value.
-   HEAP_\* statements model parameter passing and return value
    representing the statement of the heap at method calls and returns.

Options
-------

You can customize slices by choosing
[`DataDependenceOptions`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/slicer/Slicer.DataDependenceOptions.html)
and
[`ControlDependenceOptions`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/slicer/Slicer.ControlDependenceOptions.html).

Available `DataDependenceOptions` include:

-   `FULL` : track all data dependencies
-   `NO_BASE_PTRS` : like `FULL`, but ignore data dependence edges that
    define base pointers for indirect memory access
-   `NO_BASE_NO_HEAP` : like `NO_BASE_PTS`, and additionally ignore all
    data dependence edges to/from heap locations
-   `NO_HEAP` : like `FULL`, and additionally ignore all data dependence
    edges to/from heap locations
-   `NONE` : ignore all data dependences
-   `REFLECTION` : like `NO_BASE_NO_HEAP`, but also ignore data
    dependence edges originating from checkcast statements. This is the
    dependence algorithm used to unsoundly track from from newInstance
    to casts.

The available `ControlDependence` options are:

-   `FULL` : track all control dependence
-   `NONE` : ignore all control dependence.

Thin Slicing
------------

To execute context-sensitive **Thin Slicing**, as described by [this
paper](https://researcher.ibm.com/researcher/files/us-msridhar/pldi07.pdf),
we can use the `DataDependenceOptions.NO_BASE_PTRS` and
`ControlDependence.NONE` options. To perform optimized
context-<em>insensitive</em> thin slicing, see the
[CISlicer](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/slicer/thin/CISlicer.html)
class.

Example
-------

To compute a slice, we need a `CallGraph` for the program and a
calculated `PointerAnalysis`. We also need to somehow find the seed
statement. The following code snippet (partly from the PDFSlice example)
illustrates these necessary steps. The seed statement in this example is
the first instance of calling a `println()` function in the
`public static void main(String args[])` function.

Once the slice is calculated, we can analyze the statements more
(including determining the original source line numbers). See the
[IR](/UserGuide:IR "wikilink") page.

`<pre>
    public static CallGraphBuilder doSlicing(String appJar) throws WalaException {
      // create an analysis scope representing the appJar as a J2SE application
        AnalysisScope scope = AnalysisScopeReader.makeJavaBinaryAnalysisScope(appJar,CallGraphTestUtil.REGRESSION_EXCLUSIONS);
        ClassHierarchy cha = ClassHierarchy.make(scope);

        Iterable<Entrypoint> entrypoints = com.ibm.wala.ipa.callgraph.impl.Util.makeMainEntrypoints(scope, cha);
        AnalysisOptions options = new AnalysisOptions(scope, entrypoints);

        // build the call graph
        com.ibm.wala.ipa.callgraph.CallGraphBuilder cgb = Util.makeZeroCFABuilder(options, new AnalysisCache(),cha, scope, null, null);
        CallGraph cg = cgb.makeCallGraph(options);
        PointerAnalysis pa = cgb.getPointerAnalysis();

        // find seed statement
        Statement statement = findCallTo(findMainMethod(cg), "println");

        Collection<Statement> slice;

        // context-sensitive traditional slice
        slice = Slicer.computeBackwardSlice ( statement, cg, pa );
        dumpSlice(slice);

        // context-sensitive thin slice
        slice = Slicer.computeBackwardSlice(statement, cg, pa, DataDependenceOptions.NO_BASE_PTRS,
            ControlDependenceOptions.NONE);
        dumpSlice(slice);

        // context-insensitive slice
        ThinSlicer ts = new ThinSlicer(cg,pa);
        Collection<Statement> slice = ts.computeBackwardThinSlice ( statement );
        dumpSlice(slice);
    }

    public static CGNode findMainMethod(CallGraph cg) {
      Descriptor d = Descriptor.findOrCreateUTF8("([Ljava/lang/String;)V");
      Atom name = Atom.findOrCreateUnicodeAtom("main");
      for (Iterator<? extends CGNode> it = cg.getSuccNodes(cg.getFakeRootNode()); it.hasNext();) {
        CGNode n = it.next();
        if (n.getMethod().getName().equals(name) && n.getMethod().getDescriptor().equals(d)) {
          return n;
        }
      }
      Assertions.UNREACHABLE("failed to find main() method");
      return null;
    }

    public static Statement findCallTo(CGNode n, String methodName) {
      IR ir = n.getIR();
      for (Iterator<SSAInstruction> it = ir.iterateAllInstructions(); it.hasNext();) {
        SSAInstruction s = it.next();
        if (s instanceof com.ibm.wala.ssa.SSAAbstractInvokeInstruction) {
          com.ibm.wala.ssa.SSAAbstractInvokeInstruction call = (com.ibm.wala.ssa.SSAAbstractInvokeInstruction) s;
          if (call.getCallSite().getDeclaredTarget().getName().toString().equals(methodName)) {
            com.ibm.wala.util.intset.IntSet indices = ir.getCallInstructionIndices(call.getCallSite());
            com.ibm.wala.util.debug.Assertions.productionAssertion(indices.size() == 1, "expected 1 but got " + indices.size());
            return new com.ibm.wala.ipa.slicer.NormalStatement(n, indices.intIterator().next());
          }
        }
      }
      Assertions.UNREACHABLE("failed to find call to " + methodName + " in " + n);
      return null;
    }

    public static void dumpSlice(Collection<Statement> slice) {
      for (Statement s : slice) {
        System.err.println(s);
      }
    }
</pre>`

### Warning: exclusion of copy statements from slice

Because the SSA IR has already been somewhat optimized, some statements
such as simple assignments (`x=y`, `y=z`) do not appear in the IR, due
to copy propagation optimizations done automatically during SSA
construction by the SSABuilder class. In fact, there is no SSA
assignment instruction; additionally, a javac compiler is free to do
these optimizations, so the statements may not even appear in the
bytecode. Thus, these Java statements will never appear in the slice.

### Mapping slices to source code

See details at [this
page](/UserGuide:MappingToSourceCode#From_Slices_to_source_line_numbers "wikilink").