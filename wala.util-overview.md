This page provides high-level documentation for the `com.ibm.wala.util` sub-project of WALA.  This is a work in progress!

## Table of Contents
  * [Integer Sets](#integer-sets)
    * [Union-Find](#union-find)
    * [Mapping to integers](#mapping-to-integers)
  * [Graphs](#graphs)
    * [Data Structures](#data-structures)
    * [Graph Algorithms](#graph-algorithms)
  * [Fixed Point Solvers](#fixed-point-solvers)
    * [General Infrastructure](#general-infrastructure)
      * [Booleans and bit vectors](#booleans-and-bit-vectors)
    * [Dataflow Solver](#dataflow-solver)
  * [Tabulation Solver](#tabulation-solver)
  * [Other utilities](#other-utilities)
    * [Collections](#collections)
    * [Progress monitors](#progress-monitors)

## Integer Sets

### Union-Find

* IntegerUnionFind

### Mapping to integers


## Graphs

### Data Structures

### Graph Algorithms

## Fixed Point Solvers

For many dataflow analyses, WALA provides specialized solvers that can be used directly.  If you want to compute a dataflow analysis over a standard Killdall framework of an underlying flow graph and some transfer functions, use the specialized `DataflowSolver` type, which is documented below.  For tabulation-style inter-procedural dataflow analyses (e.g., IFDS problems), use the tabulation solver (TODO: link to documentation).  To use the WALA solver for other fixed-point problems, read on to learn about the more general infrastructure.  

### General Infrastructure

WALA's fixed-point solvers require a constraint system of `com.ibm.wala.fixpoint.IFixedPointStatement`s over `com.ibm.wala.fixpoint.IVariable`s.  An `IFixedPointStatement` has a left-hand side (LHS) `IVariable` and a list of right-hand side (RHS) `IVariable`s.  A statement can be "evaluated" via `com.ibm.wala.fixpoint.IFixedPointStatement.evaluate()`, which typically involves computing a new state for its LHS variable based on the state of its RHS variables.  (To this end, each `IVariable` must implement a `com.ibm.wala.fixpoint.IVariable.copyState(T)` method.)  A fixed-point solver repeatedly evaluates statements until there is no further change, i.e., a fixed-point has been reached.  Accordingly, statement evaluation must be monotonic.    

The statements for a problem are collected in an `com.ibm.wala.fixpoint.IFixedPointSystem`, which gets solved by an `com.ibm.wala.fixpoint.IFixedPointSolver`.  The outer logic for WALA's optimized fixed-point solver resides in `com.ibm.wala.fixedpoint.impl.AbstractFixedPointSolver`, and `com.ibm.wala.fixedpoint.impl.DefaultFixedPointSystem` provides a default implementation of `IFixedPointSystem`.  The two are connected by `com.ibm.wala.fixedpoint.impl.DefaultFixedPointSolver`, which is the superclass of all fixed-point solvers used in other WALA components.  To solve a fixed-point system, invoke `com.ibm.wala.fixpoint.IFixedPointSolver.solve(IProgressMonitor)`.

In WALA, the logic for `IFixedPointStatement` evaluation typically resides in an `com.ibm.wala.fixpoint.AbstractOperator` object.  WALA provides a `com.ibm.wala.fixedpoint.impl.NullaryOperator` type for statements with no RHS variables and a `com.ibm.wala.fixpoint.UnaryOperator` type for statements with a single RHS variable.  When using WALA's optimized solver, fixed-point statements should not be allocated and added to the fixed-point system directly.  Instead, use the appropriate variant of `AbstractFixedPointSolver.newStatement()`, depending on the number of RHS variables and arity of the operator.    

Note that while solving a fixed-point system, statement evaluation is allowed to add new statements to the system.  These statements are incorporated immediately and handled during the current solving phase.  This functionality is useful, e.g., for techniques like on-the-fly call graph construction during points-to analysis.

#### Booleans and bit vectors

WALA provides the `com.ibm.wala.fixpoint.BooleanVariable` type for `IVariable`s with boolean state.  WALA also provides the `com.ibm.wala.fixpoint.TrueOperator` and `com.ibm.wala.fixpoint.UnaryOr` operators for constraints involving `BooleanVariable`s.

The `com.ibm.wala.fixpoint.BitVectorVariable` type represents `IVariable`s with bit-vector state.  For efficiency, the state is represented using a `MutableSharedBitVectorIntSet` (see the integer set section).  A number of operators on `BitVectorVariables` are provided as part of WALA's dataflow analysis framework; see below.

### Dataflow Solver

## Tabulation Solver

## Other utilities

### Collections

* Heap 
* HashMapFactory / HashSetFactory
* Iterator2Collection, Iterator2Iterable

### Progress monitors


