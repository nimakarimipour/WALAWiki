The Big Picture
---------------

WALA provides a set of libraries for program analysis.

The typical WALA client will use the libraries to perform interprocedural analysis, via the following sequence:

1.  Build a [ClassHierarchy]; that is, read the program source (e.g. bytecode) into memory, and parse some basic information regarding the types represented. A ClassHierarchy object represents a universe of code to analyze.
2.  Build a [CallGraph]; that is, perform pointer analysis with on-the-fly call graph construction to resolve the targets of dynamic dispatch calls, and build a graph of CGNode objects representing the possible program calling structure.
3.  Perfom some analysis over the resulting call graph.

Many types of analysis are possible. Almost all clients will want to navigate the WALA [IR], which encodes the instructions and control flow of a particular method. The IR is an immutable control-flow graph of instructions in SSA form.

More Details
------------
the latest [javadoc]

### com.ibm.wala.core

This project holds WALA's core analysis utilities. To find out a small subset of what you need to know about these utilties, see the

-   [wala.core technical overview].

### com.ibm.wala.core.testdata

This project holds the source for some test cases used in WALA JUnit tests. These test cases are not WALA examples; they do not use any WALA code.

### com.ibm.wala.core.tests

This project holds the JUnit test cases for the com.ibm.wala.core project.

### com.ibm.wala.cast.js

This project holds the JavaScript frontend to WALA

-   [JavaScript frontend].

### com.ibm.wala.eclipse 

This project holds WALA code that contributes to an Eclipse UI and/or relies on services from an instance of the Eclipse platform. Annie Ying has contributed a call graph viewer that works inside JDT. See

-   [wala.eclipse].

### com.ibm.wala.shrike 

This project holds Rob O'Callahan's Shrike bytecode utilities.

Shrike consists of two main parts:

-   ShrikeCT : utilities to manipulate class file formats directly
-   ShrikeBT : a “sanitized” representation of code for the JVM stack machine, abstracting away the most irrelevant details.

WALA relies on Shrike to read class files. Shrike is also appropriate for bytecode manipulation such as instrumentation.

Check out the [Shrike\_technical\_overview] for more details.

  [ClassHierarchy]: UserGuide:ClassHierarchy "wikilink"
  [CallGraph]: UserGuide:CallGraph "wikilink"
  [IR]: UserGuide:IR "wikilink"
  [javadoc]: http://wala.sourceforge.net/javadocs/trunk/