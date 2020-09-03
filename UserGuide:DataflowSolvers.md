---
title: UserGuide:DataflowSolvers
permalink: /UserGuide:DataflowSolvers/
---

WALA includes several types of solvers applicable for various styles of
dataflow analysis. You may find one of these helpful for setting up and
solving a particular problem.

Simple Fixed-point
------------------

The simplest solver is the fixed point solver,
`DefaultFixedPointSolver`. This solver simply manages a worklist of
"statements", and "evaluates" statements until there are no further
changes to the solution.

The main implementation benefits of using the WALA solver are a
space-efficient representation of dependencies between statements, and
the ability to evaluate statements based on various approximations to
topological order. The latter feature is crucial for rapid convergence
in some problems.

WALA's default IR construction and flow-insensitive pointer analysis,
along with several other analyses, use the common fixed-point solver at
the lowest level.

Killdall-Style Frameworks
-------------------------

The `com.ibm.wala.dataflow` package provides a layer on top of the
fixed-point solver to facilitate implementation of Killdall-style
dataflow frameworks. The abstractions here help set up a dataflow system
induced over nodes and/or edges of a graph.

To solve backwards dataflow problems, simply reverse graph edges using
`GraphInverter.invert()`, and set up flow functions appropriately.

RHS solver
----------

The `TabulationSolver` provides an implementation of the RHS algorithm
for Sharir-Pneuli functional context-sensitive dataflow analysis,
including IFDS. The `TabulationSolver` implementation has been tuned for
space efficiency, and includes various extensions to help with features
such as exceptional control flow.

To solve backwards dataflow problems, start with the
`BackwardsSupergraph`.