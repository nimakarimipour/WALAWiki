---
title: CAst CallGraph Details
permalink: /CAst_CallGraph_Details/
---

Here, we give some technical details for those interested in the
implementation of the call graph builder for CAst-based languages, i.e.,
the Java source front ends and JavaScript.

Lexical Scoping
---------------

CAst allows for languages with nested functions and accesses from nested
functions to variables declared in enclosing functions. We call such an
access a *lexical access*, and we say a variable accessed from a nested
function is *exposed*. Here we discuss how lexical accesses are handled
during WALA's pointer analysis. Note that while lexical accesses from
anonymous classes are translated away in Java bytecodes, such accesses
are handled directly when analyzing Java source code using CAst.

### Original approach

Originally, WALA went to great lengths to try to maintain some kind of
SSA representation even for exposed variables, to improve precision with
some level of flow sensitivity. In this approach, lexical reads from
calls to nested functions must access the correct SSA name for an
exposed variable, and lexical writes from such calls must be treated as
a "def" and create a new SSA name. Unfortunately, this approach becomes
rather complex since the analysis must compute call graph information to
determine when nested functions are being invoked. Hence, SSA renaming
and call graph construction must be performed simultaneously, a tricky
proposition. Issues are further complicated in JavaScript, since nested
functions may be returned as closures, in which case lexical writes
operate on the closure rather than a local variable on the call stack.
While the code to implement this technique lives on (e.g., see
[SSAConversion](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast/source/java/com/ibm/wala/cast/ir/ssa/SSAConversion.java)),
we have transitioned to a simpler approach that we believe should have
little precision loss in practice.

### New approach

Our new approach essentially works by heap allocating any exposed
variable and treating lexical accesses appropriately. Specifically:

-   Lexical accesses are determined by building symbol tables while
    generating the CAst from a language-specific AST. The relevant code
    is in
    [`AstTranslator`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast/source/java/com/ibm/wala/cast/ir/translator/AstTranslator.java),
    e.g., the `doLexicallyScopedRead` and `doLexicallyScopedWrite`
    methods. Note that with this scheme, we expect the implementation of
    `AstTranslator.useLocalValuesForLexicalVars()` to return `false`.
-   Conceptually, the pointer analysis maintains a copy of an exposed
    variable for each `CGNode` representing the declaring function (see
    the `UpwardFunargPointerKey` inner class in
    [`AstSSAPropagationCallGraphBuilder`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast/source/java/com/ibm/wala/cast/ipa/callgraph/AstSSAPropagationCallGraphBuilder.java)).
-   Given a lexical access of *v* declared in function *f* from `CGNode`
    *n*, the `AstSSAPropagationCallGraphBuilder.getLexicalDefiners()`
    method discovers the relevant `CGNode`s for *f*. First, it computes
    the current points-to set of `v1` for *n*, which holds the
    "receiver" for *n*. (In JavaScript, `v1` holds the abstract
    location(s) for the invoked function, *not* the `this` argument.)
    The receiver / function is assumed to be of type
    [`ScopeMappingInstanceKey`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast/source/java/com/ibm/wala/cast/ipa/callgraph/ScopeMappingInstanceKeys.java#L74),
    an `InstanceKey` that maintains information about lexical parents of
    the method allocating the function. For each
    `ScopeMappingInstanceKey`, we invoke `getFunargNodes` to find the
    appropriate `CGNode`s for *f*. NOTE: for JavaScript,
    `getFunargNodes` relies on the (synthetic) contructor allocating the
    function to be analyzed with a caller or call-string context, in
    order to determine which function invoked the constructor.
-   Once we have the appropriate CGNodes for *f*, we create
    corresponding `UpwardFunargPointerKey` objects for *v*, and then add
    the appropriate constraints on the key depending on whether the
    access is a read or a write.

Context Sensitivity
-------------------