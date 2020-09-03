---
title: UserGuide:CallGraph
permalink: /UserGuide:CallGraph/
---

[wala.core technical overview](/wala.core_technical_overview "wikilink")

Call Graph Basics
-----------------

The
[CallGraph](http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/CallGraph.html)
class represents potentially *context-sensitive* call graphs, via a
logical *cloning* of methods. Each call graph node
([`CGNode`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/CGNode.html))
represents a method `IMethod` in a `Context`.

So what's a
[`Context`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/Context.html)?
Basically, a `Context` is just a name for a clone of an `IMethod`. For
*context-insensitive* call graphs, note the special `Context`
[`Everywhere.EVERYWHERE`](http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/impl/Everywhere.html).
This will be used as a default context in context-insensitive
algorithms.

Note that for a given `IMethod`, a context-sensitive call graph may have
many nodes (contexts) representing the method. You can get all these
nodes using the method
\[<http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/CallGraph.html#getNodes(com.ibm.wala.types.MethodReference>)
`CallGraph.getNodes(MethodReference m)]`

WALA supports a family of on-the-fly call graph construction algorithms,
integrated with flow-insensitive pointer analysis. See
[UserGuide:PointerAnalysis](/UserGuide:PointerAnalysis "wikilink") for
more details. WALA also has an implementation of call-graph construction
via rapid type analysis (see
\[<http://wala.sourceforge.net/javadocs/com.ibm.wala.core/com/ibm/wala/ipa/callgraph/impl/Util.html#makeRTABuilder(com.ibm.wala.ipa.callgraph.AnalysisOptions,%20com.ibm.wala.ipa.cha.IClassHierarchy,%20com.ibm.wala.ipa.callgraph.AnalysisScope,%20com.ibm.wala.util.warnings.WarningSet>)
`Util.makeRTABuilder()`\]), but its use is strongly discouraged. The
included 0-CFA analysis (see
[UserGuide:PointerAnalysis](/UserGuide:PointerAnalysis "wikilink")) is
almost always faster and more precise. Furthermore, RTA operates on
bytecodes, some of which may be proved dead during SSA construction and
hence not represented in the SSA IR. This can lead to undesirable
behaviors like call sites returned by the RTA call graph that do not
appear in the SSA IR. Finally, note that models of native methods like
`Object.clone()` in the RTA call graph are "sound enough" to ensure that
the call graph includes all reachable code but may be unsound otherwise.