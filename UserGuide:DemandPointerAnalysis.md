---
title: UserGuide:DemandPointerAnalysis
permalink: /UserGuide:DemandPointerAnalysis/
---

[wala.core technical overview](/wala.core_technical_overview "wikilink")

WALA's demand-driven pointer analysis, exposed through the
[`DemandRefinementPointsTo`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/DemandRefinementPointsTo.html)
class, implements the analysis presented in the paper ["Refinement-Based
Context-Sensitive Points-To Analysis for
Java"](http://doi.acm.org/10.1145/1133255.1134027) and in [Manu
Sridharan's
dissertation](http://www.eecs.berkeley.edu/Pubs/TechRpts/2007/EECS-2007-125.html).

Initialization
--------------

There are several parameters that must be set during initialization of
the `DemandRefinementPointsTo` object, documented below. Basic
initialization code can be seen in the
`AbstractPtrTest.makeDemandPointerAnalysis()` method in the
`trunk.tests.demandpa` project.

### Call Graph

The analysis requires a pre-computed [call
graph](/UserGuide:CallGraph "wikilink") (though it can be configured to
refine virtual call targets on-the-fly). Most testing has been done with
a 0-CFA call graph (see
[UserGuide:PointerAnalysis](/UserGuide:PointerAnalysis "wikilink")), but
more precise call graphs should work fine. Using a call graph built by
rapid type analysis is not supported and may lead to unsoundness or
crashes (see note in
[UserGuide:CallGraph](/UserGuide:CallGraph "wikilink")).

### Heap Model

The analysis also requires a
[`HeapModel`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/HeapModel.html)
to determine how to abstract pointers and heap locations (see
[UserGuide:PointerAnalysis](/UserGuide:PointerAnalysis "wikilink")). The
analysis has mostly been tested with a `HeapModel` obtained from calling
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/impl/Util.html#makeVanillaZeroOneCFABuilder(com.ibm.wala.ipa.callgraph.AnalysisOptions,%20com.ibm.wala.ipa.callgraph.AnalysisCache,%20com.ibm.wala.ipa.cha.IClassHierarchy,%20com.ibm.wala.ipa.callgraph.AnalysisScope>)
`Util.makeVanillaZeroOneCFABuilder()`\], but other heap models should
work.

### State Machine

The analysis can be extended to have more precision or to track other
properties using a
[`StateMachine`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/statemachine/StateMachine.html).
For example, context sensitivity is implemented through the use of a
[`ContextSensitiveStateMachine`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/ContextSensitiveStateMachine.html).
The `DemandRefinementPointsTo` constructor takes a
[`StateMachineFactory`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/statemachine/StateMachineFactory.html)
as a parameter; the factory is used to create a new `StateMachine`
object for each query and for each refinement pass. For
context-insensitive analysis, pass in a
[`DummyStateMachine.Factory`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/statemachine/DummyStateMachine.Factory.html)
object.

### Refinement Policy

A
[`RefinementPolicy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/refinepolicy/RefinementPolicy.html)
dictates how the analysis handles field accesses and method calls, and
also specifies the budgets for each refinement pass. By default, the
analysis uses a
[`SinglePassRefinementPolicy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/refinepolicy/SinglePassRefinementPolicy.html)
that leads to a single analysis pass with no field sensitivity or
on-the-fly call graph refinement. Other useful refinement policies
include the
[`ManualRefinementPolicy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/refinepolicy/ManualRefinementPolicy.html),
which refines for certain hand-specified fields (mostly `java.util`
collections classes), and the
[`TunedRefinementPolicy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/refinepolicy/TunedRefinementPolicy.html),
which attempts to discover which fields and method calls are important
to treat precisely.

Similar to state machines, the refinement policy is changed through
setting a
[`RefinementPolicyFactory`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/refinepolicy/RefinementPolicyFactory.html)
with a call to
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/DemandRefinementPointsTo.html#setRefinementPolicyFactory(com.ibm.wala.demandpa.alg.refinepolicy.RefinementPolicyFactory>)
`DemandRefinementPointsTo.setRefinementPolicyFactory()`\].

Running The Analysis
--------------------

The most straightforward way to run the analysis is to call the
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/DemandRefinementPointsTo.html#getPointsTo(com.ibm.wala.ipa.callgraph.propagation.PointerKey>)
`getPointsTo()`\] method that just takes a
[`PointerKey`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/callgraph/propagation/PointerKey.html)
as an argument. (The `PointerKey` should be obtained from the same
`HeapModel` used to initialize the analysis.) If the analysis is able to
compute a result in the budget specified by the `RefinementPolicy`, that
result is returned; otherwise, the analysis returns `null`.

For more control over the analysis, call the
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/DemandRefinementPointsTo.html#getPointsTo(com.ibm.wala.ipa.callgraph.propagation.PointerKey,%20com.ibm.wala.demandpa.genericutil.Predicate>)
`getPointsTo()`\] method that takes both a `PointerKey` and a
[`Predicate`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/genericutil/Predicate.html)
as arguments. The `Predicate` specifies the condition that the result
points-to set should ideally satisfy. (For example, if the points-to set
is used for checking downcast safety, the predicate would check that no
instance key in the set would cause a downcast failure.) The method
returns both the points-to set (again possibly `null`) and a
[`PointsToResult`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/demandpa/alg/DemandRefinementPointsTo.PointsToResult.html)
indicating whether the (1) predicate was satisfied, (2) no more
refinement was possible, or (3) the analysis budget was exceeded.