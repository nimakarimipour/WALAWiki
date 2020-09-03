---
title: GettingStarted:StringAnalysis
permalink: /GettingStarted:StringAnalysis/
---

Getting started with WALA String Analysis
-----------------------------------------

*Note for developers*: the string analysis functionality is a WALA
*incubator* project; this code is not-quite-yet ready for prime time.

WALA now includes a module for performing string analysis, analysis that
models the set of strings that different program variables may take on,
based on the same approach as one described in the paper "Static
approximation of dynamically generated Web pages" (WWW'05). This is
essentially a generalization of analysis for string constants; the
current analysis models the set of strings of a given variables as a
context-free language and supports queries regarding whether this
language does or does not contain particular strings. Due to
undecidability issues, the current interface supports queries whether a
given variable may take on strings from a given regular language rather
than a context-free one. The code on WALA is currently working only for
JavaScript, and you need several additional projects to use this
analysis (These projects live in the `incubator` branch of subversion,
and not `trunk`):

` * com.ibm.wala.automaton (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.automaton)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.automaton)`)`
`   o A generic library of automaton and language functionality`
` * com.ibm.wala.stringAnalysis (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.stringAnalysis)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.stringAnalysis)`)`
`   o The core of string analysis functionality`
` * com.ibm.wala.stringAnalysis.js (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.stringAnalysis.js)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.stringAnalysis.js)`)`
`   o JavaScript-specific aspects of string analysis`

In addition, you should get the test projects, which contain both tests
that you can use to verify the code is working for you and also example
that you can use to get started with learning how the analysis is used.

` * com.ibm.wala.automaton.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.automaton.test)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.automaton.test)`)`
`   o Tests of automaton and language functionality`
` * com.ibm.wala.stringAnalysis.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.stringAnalysis.test)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.stringAnalysis.test)`)`
`   o Tests of core string analysis functionality`
` * com.ibm.wala.stringAnalysis.js.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.stringAnalysis.js.test)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.stringAnalysis.js.test)`)`
`   o Tests of JavaScript-specific string analysis`

The place to start looking at this code is the
`com.ibm.wala.stringAnalysis.js.examples.SimpleStringAnalysisDriver`.
This driver simply analyzes a JavaScript program and executes a single
query as to whether a given variable contains all the strings in a given
pattern. To use this example driver, first try the run configuration for
it that is part of the com.ibm.wala.stringAnalysis.js.test project. Once
that example is working for you, try loading your favorite JavaScript
program and asking a similar question about it.

You can also look at the
`com.ibm.wala.stringAnalysis.js.test.translator.TestGR2CFG` class to see
further examples of queries that are run against the `test.js` script in
`com.ibm.wala.stringAnalysis.js.test.example`.