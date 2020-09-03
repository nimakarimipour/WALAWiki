---
title: GettingStarted:DOMAnalysis
permalink: /GettingStarted:DOMAnalysis/
---

Getting started with WALA DOM Analysis
--------------------------------------

*Note for developers*: the DOM analysis functionality is a WALA
*incubator* project; this code is not-quite-yet ready for prime time.

WALA now includes a module for performing DOM analysis in addition to
string analysis. This analyzer predicts a set of DOM instances arising
at run-time using similar techniques to the string analysis. The code on
WALA is currently working only for JavaScript, and you need the
following projects and [the string analysis
projects](/GettingStarted:StringAnalysis "wikilink") to use this
analysis. (These projects live in the `incubator` branch of subversion,
and not `trunk`):

` * com.ibm.wala.domAnalysis (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.domAnalysis)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.domAnalysis)`)`
`   o The core of DOM analysis functionality`
` * com.ibm.wala.domAnalysis.js (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.domAnalysis.js)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.domAnalysis.js)`)`
`   o JavaScript-specific aspects of DOM analysis`

In addition, you should get the test projects, which contain both tests
that you can use to verify the code is working for you and also example
that you can use to get started with learning how the analysis is used.

` * com.ibm.wala.domAnalysis.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.domAnalysis.test)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.domAnalysis.test)`)`
`   o Tests of core DOM analysis functionality`
` * com.ibm.wala.domAnalysis.js.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.domAnalysis.js.test)`) (`[`source`](http://svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.domAnalysis.js.test)`)`
`   o Tests of JavaScript-specific DOM analysis`

The place to start looking at this code is the
`com.ibm.wala.domAnalysis.js.examples.SimpleDOMAnalysisDriver`. This
driver simply analyzes a JavaScript program and executes a single query
as to whether a given variable contains all the strings in a given
pattern. To use this example driver, first try the run configuration for
it that is part of the com.ibm.wala.domAnalysis.js.test project. Once
that example is working for you, try loading your favorite JavaScript
program and asking a similar question about it.

You can also look at the
`com.ibm.wala.domAnalysis.js.test.translator.TestGR2RTG` class to see
further examples of queries that are run against the `test.js` script in
`com.ibm.wala.domAnalysis.js.test.example`.