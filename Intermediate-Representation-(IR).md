Overview
--------

The WALA IR (Intermediate Representation) is the central data structure
that represents the instructions of a particular method. The IR
represents a method's instructions in a language close to JVM bytecode,
but in an SSA-based register transfer language which eliminates the
stack abstraction, relying instead on a set of symbolic registers. The
IR organizes instructions in a control-flow graph of basic blocks, as
typical in compiler textbooks.

The IR is immutable; it does not support program transformation, and
WALA does not provide support for code generation from an IR. The
philosophy is that the IR could be used in a pass that generates
analysis information, to be used in support of some other tool, such as
an IDE or compiler. (It is possible to do program transformation at the
bytecode level in WALA using the shrike package, but not directly from
the IR discussed here). Typically, an analysis will set up various
structures and maps from IR constructs to relevant analysis information,
such as abstractions and dataflow facts.

IR Basics
---------

The WALA
[IR](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/IR.html)
class provides an [[SSA]] based
intermediate representation of a particular method. The IR consists of a
[control-flow
graph](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/cfg/ControlFlowGraph.html)
of [basic
blocks](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/cfg/IBasicBlock.html),
along with a set of
[instructions](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSAInstruction.html).
Try the
[`PDFWalaIR`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/examples/drivers/PDFWalaIR.html)
example program to see what an IR looks like.

The current IR implementation is somewhat convoluted, due to historical
back-breaking for space efficiency. (it may change in the future).
Rather than deal with the underlying implementation, you should try to
access the IR via its public interfaces. For example, the following
should be most useful:

-   [IR.iterateAllInstructions()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/IR.html#iterateAllInstructions()): return all the instructions, in an
    undefined order. Useful for flow-insensitive analysis.
-   [IR.getControlFlowGraph()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/IR.html#getControlFlowGraph()): return the control-flow graph, a graph
    of IBasicBlock.
-   [IR.getControlFlowGraph().iterateNodes()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/util/graph/NodeManager.html#iterateNodes()): return an iteration of
    the nodes (basic blocks) in a control-flow graph.
-   [IBasicBlock.iterateAllInstructions()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/cfg/IBasicBlock.html#iterateAllInstructions()): iterate all the
    instructions in a particular basic block.

If you care about the order in which you iterate over the instructions,
you should loop over the basic blocks, and then iterate over the
instructions in each basic block.

Usually, you will build a callgraph, and then acquire the IR for a
particular node via CGNode.getIR(). (You can also directly build IR for
a particular IMethod using the method `AnalysisCache.getIR(IMethod)`.)  
Note that WALA might build different
IRs for a single IMethod, depending on the context represented by a
CGNode. For example, each `clone` node represents the `clone` IMethod in
a context which defines the receiver type. The IR for each `clone` node
is specialized with respect to this context.

Control Flow Graph
------------------

Exceptional edges are added fairly naively, using only declared types to
determine exceptional flow, and assuming every Potentially Excepting
Instruction (PEI) might throw exceptions. There is no smart optimization
to remove infeasible exceptional control flow.

Occasionally you may wish to tweak your view of the CFG. For example,
perhaps a client wants a view of the CFG that ignores exceptional edges.
For these purposes, check out the class `PrunedCFG`.

To do some basic navigation of a CFG, check out the `Util` class in
`com.ibm.wala.cfg`. This class gives utilities to find, for example, the
taken and not-taken successors of conditional branches.

Value Numbering
---------------

The variables in an IR each have a unique id, called the *value number*.
So you can think of the variables in an IR having the names **v1, v2,
v3**, .... .

By convention, the value numbers for a method start at 1. So for a
non-static method, v1 represents the `this` parameter. The parameters to
the method are next (v2, v3, etc). For a static method, v1 represents
the first parameter.

Most
[`SSAInstructions`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSAInstruction.html)
define at most one variable, and use some number of variables. Each
instruction carries the numbers of the variables it defs and uses,
accessible through
[getDef()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSAInstruction.html#getDef()) and
[`getUses()`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSAInstruction.html#getUse(int)).
Actually,
[`SSAInvokeInstructions`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSAInvokeInstruction.html)
may additionally def a second variable, representing the exceptional
return value.

Conversely, each variable (value number) will have exactly one def and
some number of uses. Use the
[`DefUse`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/DefUse.html)
class to find the instructions that def and use a particular value
number. If `DefUse` returns `null` as the def for a value number, that
value number represents either a parameter or a constant. See below for
information regarding constants.  Note that the definition may be a phi
statement; see below for more details.

During SSA translation, stack locations and locals are both translated
into symbolic virtual registers. For a given SSA value number and
instruction index, you can use
[`IR.getLocalNames()`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/IR.html#getLocalNames(int,%20int))
to find out the names of the corresponding locals if available. You can
look at
[SSABuilder.SSA2LocalMap](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SSABuilder.html)
to see how the IR keeps track of this information internally.

Phi statements
--------------

[As is standard in SSA form](https://en.wikipedia.org/wiki/Static_single_assignment_form#Converting_to_SSA), the WALA IR contains *phi statements* to handle cases where multiple definitions may reach some use.  These phi statements (`SSAPhiInstruction`s) are *not* stored as normal instructions in the IR, for historical reasons.  Instead, they are stored on `BasicBlock`s in the IR's control-flow graph.  To see all phi instructions in an IR, call `IR.iteratePhis()`.  To see the phi statements on a particular basic block, call `BasicBlock.iteratePhis()`.  Logically, the phi statements on a basic block execute **at the beginning** of the basic block, i.e., before any of the normal instructions in the block.  

Type inference
--------------

You can discover types intraprocedurally for value numbers in an IR using the `TypeInference` class. E.g.:

        IR ir = ...;
        boolean doPrimitives = ...; // infer types for primitive vars?
        TypeInference ti = TypeInference.make(ir, doPrimitives);
        TypeAbstraction type = ti.getType(vn);

Constant Values
---------------

Primitive and String constants are represented in the IR as follows.
Each constant has a corresponding variable, but there is no explicit
statement in the IR assigning the constant to the variable. To discover
constants, use the IR's
[`SymbolTable`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SymbolTable.html),
obtained through calling
[IR.getSymbolTable()](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/IR.html#getSymbolTable()).
[`SymbolTable.isConstant()`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SymbolTable.html#isConstant(int))
will tell you if a particular variable represents a constant (pass in
the value number of the variable as the parameter), and
[`SymbolTable.getConstantValue()`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ssa/SymbolTable.html#getConstantValue(int))
will give you the represented value.

Pi nodes (advanced)
-------------------

π assignments (referred to as "pi nodes" in WALA code) were introduced in [Bodik et al.'s ABCD work](http://dl.acm.org/citation.cfm?id=349342) as a technique to capture local path conditions directly in SSA form.  E.g., if we have the code `S1: c = v1 != null S2: if (c) { return v1; }`, we could rewrite it to `S1: c = v1 != null S2: if (c) { v2 = PI(v1, S1); return v2; }`.  With the new representation, an otherwise path-insensitive analysis can easily reason that v2 is never null.

To build IR with pi nodes, invoke `setPiNodePolicy()` on the corresponding `SSAOptions` object with a corresponding `SSAPiNodePolicy`, which governs how pi nodes are constructed.  WALA has two built-in pi-node policies: `NullTestPiPolicy`, which adds pi nodes for null checks (like the example above), and `InstanceOfPiPolicy` for instance-of type tests.

Like with phi instructions, pi instructions are stored on `BasicBlock`s and are not normal IR instructions.  To iterate all pi instructions in an IR, call `IR.iteratePis()`.  More usefully, you can call `BasicBlock.iteratePis()` to see the pi instructions stored on that block.  Note that pi instructions logically execute **at the end** of the basic block on which they are stored; this is the opposite of phi instructions.  More precisely, each `SSAPiInstruction` is associated with some edge *e* in the control-flow graph and is stored at source block of *e$; the instruction only (logically) executes when *e* is traversed.

For a given `SSAPiInstruction`, `getVal()` tells you which value number is being renamed, and `getDef()` tells you the new value number.  How can you tell what condition holds for the new name?  `getSuccessorBlock()` tells you the successor block number where the new name will be used.  Assuming you have the block `b` on which the `SSAPiInstruction` is stored, you can look at `b`'s successors in the control flow graph to figure out the `BasicBlock` `succ` corresponding to the successor block number.  Now, you can call `com.ibm.wala.cfg.Util.getTakenSuccessor()` or `com.ibm.wala.cfg.Util.getNotTakenSuccessor()` to figure out if `succ` is reached when the condition at the end of `b` is true or false.  To figure out what condition holds, you need to look at both the `SSAConditionalBranchInstruction` at the end of `b` and the "cause" instruction returned by `SSAPiInstruction.getCause()`; the specific condition will be specific to the pi-node policy.

From IR to bytecode?
--------------------

WALA is not a compiler; it does not have a code generating back end and
the IR is not designed for transformations. We do not know of any
implementation that generates bytecode directly from the WALA SSA IR.

A plausible option is to do the analysis on the IR, map the analysis
results back to the Shrike representation, and do bytecode
transformations in Shrike.