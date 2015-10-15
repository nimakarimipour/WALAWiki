WALA provides a framework for flow-insensitive Andersen-style pointer
analysis. You can customize the pointer analysis context-sensitivity
policy in various ways. We often find that for a particular client
application, it is useful to customize the pointer analysis policy to
focus analysis effort where it matters.

Currently all pointer analysis implementations perform on-the-fly call
graph construction.

A pointer analysis context-sensitivity policy can vary in two
dimensions:

-   [`HeapModel`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/HeapModel.html):
    how does the pointer analysis model abstract pointers and heap
    locations?
-   [`ContextSelector`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/ContextSelector.html):
    how does the call graph construction clone methods based on context?

You can define your own pointer analysis policy by customizing a heap
model and context selector. Also WALA provides a number of built-in
policies.

Heap Model
----------

A `HeapModel` tells the WALA pointer analysis how to abstract pointers
and heap locations. The key classes are

-   [`PointerKey`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/PointerKey.html):
    a representation of an abstract pointer
-   [`InstanceKey`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/InstanceKey.html):
    a representation of an abstract heap location.

For example, a `PointerKey` may represent a local variable, or a static
field, or an instance field of objects from a particular allocation
site, or other variants. More technically, a `PointerKey` is the name
for an equivalence class of pointers from the concrete program, which
are collected into a single representative in the abstraction.

An `InstanceKey` may represent all objects of a particular type, or all
objects from a particular allocation site, or all objects from a
particular allocation site in a particular context, or other variants.

A `HeapModel` provides call-backs for the pointer analysis to create
`PointerKeys` and `InstanceKeys` during analysis. You can customize the
policy by providing your own `HeapModel`.

Heap Graph
----------

A `HeapGraph` provides a convenient way to navigate the results of a
pointer analysis. The nodes in a `HeapGraph` are `PointerKeys` and
`InstanceKeys`. There is an edge from a `PointerKey` *P* to an
`InstanceKey` *I* if and only if the pointer analysis indicates that *P*
may point to *I*. There is an edge from an `InstanceKey` *I* to a
`PointerKey` *P* if and only if *P* represents a field of an object
instance modelled by *I*, or *P* represents the array contents of array
instance *I*.

For example, given a `HeapGraph` *h*, suppose you want to know all the
`PointerKey`s that may alias a particular `PointerKey` *p*. You first
use `h.getSuccNodes(p)` to find all `InstanceKeys` *p* may point to, and
for each such `InstanceKey` *i*, use `h.getPredNodes(i)` to find other
`PointerKeys` that may alias *p*.

Context Selector
----------------

Recall that each call graph node represents a method in a particular
`Context`. The
[`ContextSelector`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/ContextSelector.html)
object controls the policy by which the call graph builds contexts.

The simplest Context is
[`Everywhere.EVERYWHERE`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/impl/Everywhere.html#EVERYWHERE),
which represents a single, global context for a method.

Other context-policies can represent call-string contexts, contexts
naming the receiver object to implement object-sensitivity, or other
variants.

You can customize a context-sensitivity policy by providing a custom
`ContextSelector` object.

Entrypoints
-----------

Suppose we have a public entrypoint with the following signature:

`    public static void foo(Object x, Object y)`

What will pointer analysis do with the parameters to this entrypoint? In
particular, what `InstanceKey` abstractions will the pointer analysis
instantiate as the initial contents of the actual parameters `x` and
`y`?

WALA's pointer analysis uses the `Entrypoint` interface to direct this
policy. The crucial method, in the `Entrypoint` interface, is:

`   public abstract TypeReference[] getParameterTypes(int i);`

`getParameterTypes(i)` gives a set of types which will be instantiated
for the `i`th parameter to the entrypoint.

For example, suppose we build an `Entrypoint` implementation for `foo`
where

`   ``getParameterTypes(0) = { java.lang.Object, java.lang.String } `
`   ``getParameterTypes(1) = { java.lang.Integer } `

Then logically, the pointer analysis models the invocation of `foo` as
follows:

`   ``Object o1 = new java.lang.Object();
    Object o2 = new java.lang.String();
    Object o3 = non-determinstic-choice ? o1 : o2;
    Object o4 = new java.lang.Integer();
    foo(o3,o4); `

So, the pointer analysis will create `InstanceKey` abstractions
according to the governing `HeapModel`, as if the program encounters
allocations of the specified types in the `FakeRootMethod`.

By default, most WALA pointer analysis implementation examples use the
`DefaultEntrypoint` implementation of `Entrypoint`. In this
implementation, `getParameterTypes(i)` returns an array with a single
element, which is the declared type of the ith parameter. Most
production-level clients of WALA provide custom implementations of
`Entrypoint`, specialized to the types of frameworks and clients being
analyzed.

Built-in policies
-----------------

Let's look at a few of WALA's built-in pointer analysis policies.

### ZeroCFA

The ZeroCFA policy provides a simple, cheap, context-insensitive pointer
analysis. You can get a call graph builder using the ZeroCFA policy from
[`Util.makeZeroCFABuilder()`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/impl/Util.html#makeZeroCFABuilder(com.ibm.wala.ipa.callgraph.AnalysisOptions,%20com.ibm.wala.ipa.cha.ClassHierarchy,%20com.ibm.wala.ipa.callgraph.AnalysisScope,%20com.ibm.wala.util.warnings.WarningSet))

-   The `HeapModel` introduces a single `InstanceKey` for every
    concrete type. That is, all objects of a particular type are
    represented by a single abstract object.
-   The `ContextSelector` uses a single global context for each method,
    except for some special cases dealing with reflection,
    described below.

### ZeroOneCFA

The ZeroOneCFA policy provides an approximation of "standard" Andersen's
pointer analysis, using allocation sites to name abstract objects. By
default,

-   The `HeapModel` introduces a single `InstanceKey` for every
    allocation site.

However, there are variants of ZeroOneCFA, depending on some
optimizations to the heap model provided by the
[`ZeroXInstanceKeys`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/cfa/ZeroXInstanceKeys.html)
class. See the source for more details. Briefly:

-   VanillaZeroOneCFA (see
    [`Util.makeVanillaZeroOneCFA`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/impl/Util.html#makeVanillaZeroOneContainerCFABuilder(com.ibm.wala.ipa.callgraph.AnalysisOptions,%20com.ibm.wala.ipa.cha.ClassHierarchy,%20com.ibm.wala.ipa.callgraph.AnalysisScope,%20com.ibm.wala.util.warnings.WarningSet)))
    turns off all optimizations; each allocation is handled separately.
-   ZeroOneCFA (see
    [`Util.makeZeroOneCFABuilder`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/impl/Util.html#makeZeroOneCFABuilder(com.ibm.wala.ipa.callgraph.AnalysisOptions,%20com.ibm.wala.ipa.cha.ClassHierarchy,%20com.ibm.wala.ipa.callgraph.AnalysisScope,%20com.ibm.wala.util.warnings.WarningSet)))
    turns on the following optimizations
    -   [SMUSH_STRINGS](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/cfa/ZeroXInstanceKeys.html#SMUSH_STRINGS):
        individual `String` or `StringBuffer` allocation sites are
        not disambiguated. A single `InstanceKey` represents all
        `String` objects, and a single `InstanceKey` represents all
        `StringBuffer` objects.
    -   [SMUSH_THROWABLES](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/cfa/ZeroXInstanceKeys.html#SMUSH_THROWABLES):
        individual exception objects are disambiguated by type, and not
        by allocation site.
    -   [SMUSH_PRIMITIVE_HOLDERS](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/cfa/ZeroXInstanceKeys.html#SMUSH_PRIMITIVE_HOLDERS):
        if a class has no reference fields, then all objects of that
        type are represented by a single `InstanceKey`
    -   [SMUSH_MANY](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/cfa/ZeroXInstanceKeys.html#SMUSH_MANY):
        if there are more than *k* (current *k=25*) allocation sites of
        a particular type in a single method, then all these sites are
        represented by a single `InstanceKey`. For example, a library
        class initializer method might allocate 10,000 `Font` objects;
        this optimization will not disambiguate the 10,000 separate
        allocation sites.

You can customize the policies in similar ways, depending on your
client's needs.

### ZeroOneContainerCFA

The ZeroOneContainerCFA policy (see
[`Util.makeZeroOneContainerCFABuilder`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/impl/Util.java#L418))
extends the ZeroOneCFA policy with unlimited object-sensitivity for
collection objects. For any allocation sites in collection objects, the
allocation site is named by a tuple of allocation sites extending to the
outermost enclosing collection allocation. This policy can be relatively
expensive, but can be effective in disambiguating contents of standard
collection classes. For this to be effective, note that ZeroOneContainer
modifies the `ContextSelector` to clone methods based on the receiver
object's object-sensitive name. The
[`ContainerContextSelector`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/propagation/cfa/ContainerContextSelector.java)
class manages this logic, and the method
[`ContainerUtil.isContainer()`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/propagation/ContainerUtil.java#L45)
determines which classes are considered containers.

ZeroOneContainer also uses one level of call-string sensitivity for
certain methods that are either factories or are known to require such
precision, e.g., `System.arraycopy()`.
[`ContainerContextSelector.isWellKnownStaticFactory()`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/propagation/cfa/ContainerContextSelector.java#L135)
determines which methods get this treatment.

### Contexts for Reflection

To deal with reflection, WALA uses context-sensitivity policies, even if
using a context-insensitive base policy like ZeroCFA.

Currently, all built-in pointer analysis policies treat calls to
`Object.clone()` context sensitively, using the concrete type of the
receiver class as the context (see
[`CloneContextSelector`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/CloneContextSelector.html)).
Also, `clone()` has a different IR in each context, to properly model
the semantics of the method for the corresponding receiver type (see
[`CloneInterpreter`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/analysis/reflection/CloneInterpreter.html)).

There is also significant support for other reflective constructs like
`Class.forName()`, `Class.newInstance()`, `Method.invoke()`, etc. See
[`ReflectionContextSelector`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/analysis/reflection/ReflectionContextSelector.java)
and
[`ReflectionContextInterpreter`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/analysis/reflection/ReflectionContextInterpreter.java)
for the code that manages this reflection handling, and
[`AnalysisOptions.setReflectionOptions()`](https://github.com/wala/WALA/blob/3e78ff7dc6ae03337404aff53c5aa9635cef0152/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/AnalysisOptions.java#L388)
for tuning the handling.

Improving Scalability
---------------------

Reflection usage and the size of modern libraries / frameworks make it
very difficult to scale flow-insensitive points-to analysis to modern
Java programs. For example, with default settings, WALA's pointer
analyses cannot handle any program linked against the Java 6 standard
libraries, due to extensive reflection in the libraries. To improve
scalability (at the cost of some soundness in some cases), try the
following:

-   Use `AnalysisScope` exclusions to exclude classes / packages you
    know are irrelevant to the application. Exclusions are specified in
    an exclusions file passed to the appropriate method of
    `AnalysisScopeReader`. For an example, see
    [`Java60RegressionExclusions.txt`](https://github.com/wala/WALA/blob/master/com.ibm.wala.core.tests/dat/Java60RegressionExclusions.txt).
-   Analyze an older version of the standard libraries, e.g., from
    Java 1.4.
-   Reduce the context sensitivity of the pointer analysis (see above).
-   Reduce WALA's reflection handling by calling
    `setReflectionOptions()` on the
    [`AnalysisOptions`](https://github.com/wala/WALA/blob/master/com.ibm.wala.core/src/com/ibm/wala/ipa/callgraph/AnalysisOptions.java)
    object used for pointer analysis. You may have to tinker to see what
    setting enables scalability for your application.

Demand-Driven Pointer Analysis
------------------------------

As of version 1.0.04, WALA also includes a demand-driven pointer
analysis. See the [[demand pointer analysis]] page for details.