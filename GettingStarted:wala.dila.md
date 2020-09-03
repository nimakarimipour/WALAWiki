---
title: GettingStarted:wala.dila
permalink: /GettingStarted:wala.dila/
---

Dynamic Load-time Instrumentation Library for Java
==================================================

**Dila** is a dynamic load-time instrumentation library developed by the
[PROLANGS](http://www.prolangs.rutgers.edu/) group at Rutgers University
in cooperation with IBM Research. Dila uses customized class loading for
instrumenting the byte code of a program before it is executed. During
the execution of an instrumented program dynamic program
representations, such as call graphs, are created, which can be used for
various program analyses. Dila is completely written in Java and uses
the byte code manipulation toolkit Shrike from the
[WALA](http://wala.sourceforge.net/) project. It has been built as a
library for online analysis tools with a particular focus on
performance. The library is licensed under the [Eclipse Public
License](http://www.eclipse.org/legal/epl-v10.html) (EPL).

Key Features
------------

-   *Complete construction of dynamic program representations*, such as
    call graphs. Builders for dynamic representations capture the
    complete program behavior, including static initializers.
-   Simple *API suitable for operations on many graph representations*
    and other data structures, including graph comparators, walkers,
    printers, and visitors.
-   *Highly customizable instrumentation*, with pre-defined support for
    constructing call graphs, and a time stamps representation for
    approximating possible side effects, API for own instrumentations.
-   *Execution support for running target programs*, through a main()
    method as standard way of running Java programs, and
    [JUnit](http://www.junit.org) test cases or test suites.
-   *Selective construction of dynamic program representations.* Besides
    creating a representation for an execution of the entire
    application, Dila can create partial representations through a
    constraint mechanism. It provides constraints for selecting methods
    by signature pattern matching, or complete executions for
    [JUnit](http://www.junit.org) test cases, including their
    initialization, test fixture creation, test code execution and
    finalizers.

How it works
------------

A user configures which instrumentations are applied to a program, and
may provide his own instrumentations. Dila loads the classes of the
target application with a customized class loader that instruments the
byte code of these classes. Additional library classes, such as the JDK,
are excluded from the instrumentation. The instrumentation only calls
hooks in our representation builder API, which makes it small and less
intrusive with the target program. Dila executes the instrumented
program and creates the representations defined by each instrumentation.
Provided builders store all created representations in memory, which can
then be used in post-processors.

How to get Dila
---------------

Dila is currently distributed as source code release only. Go to the
following URL using an SVN client:

[<https://wala.svn.sf.net/svnroot/wala/incubator/>](https://wala.svn.sf.net/svnroot/wala/incubator/)

Dila consists of two Eclipse plugin projects:

-   [com.ibm.wala.dila](https://wala.svn.sf.net/svnroot/wala/incubator/com.ibm.wala.dila/)
-   [com.ibm.wala.dila.tests](https://wala.svn.sf.net/svnroot/wala/incubator/com.ibm.wala.dila.tests/)

Check out *com.ibm.wala.dila* into your Eclipse workspace for using Dila
with your application code. You also may want to check out
*com.ibm.wala.dila.tests* to check whether Dila works properly in your
environment and to browse examples for how to use Dila.

Contacts
--------

*Support Mailing List*

-   Problems running Dila? Check our
    [FAQs](http://wala.sourceforge.net/wiki/index.php/WalaWiki:FAQ)
-   Dila is a research prototype and still under development, but has
    been used in different research projects for online program
    analyses. Dila is published under the [Eclipse Public
    License](http://www.eclipse.org/legal/epl-v10.html) (EPL).
-   Contact our project team for further information on the project,
    technical issues, or with other questions
    <wala-wala@lists.sourceforge.net>
-   [Browse](http://sourceforge.net/mailarchive/forum.php?forum_name=wala-wala)
    the archive