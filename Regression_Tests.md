---
title: Regression Tests
permalink: /Regression_Tests/
---

Currently we are running a substantial suite of regression tests at
Watson. Some of these tests are proprietary, and some we've made
open-source.

BTW ... [JUnit](http://www.junit.org/index.htm) tests are a great way to
contribute to the WALA project.

Running the tests yourself
--------------------------

1.  Check out the `com.ibm.wala.core.tests` and
    `com.ibm.wala.core.testdata` projects into your workspace.
2.  Manually run `ant`, which will build the file
    `com.ibm.wala.core.testdata/bin/com.ibm.wala.core.testdata_1.0.0.jar`
    and download some dependent jars. Do this by running the `build.xml`
    file in the `com.ibm.wala.core.testdata` project. In Eclipse, you
    can select the file, right-click, and `Run as` then `Ant build...`
    Make sure the default `build.update.jar` target is selected to be
    run.
3.  Run the JUnit tests with the `wala.core` or
    `wala.core short profile` launchers (or
    `wala.core short profile (non-windows)` if you're not on a Windows
    machine). The former exercises a wider test suite than the latter,
    and will take longer to run.

Protocol for checking in code
-----------------------------

If you have been granted check-in privileges to WALA, please observe the
following:

1.  Run the `wala.core short profile` regression driver before checking
    in any change to `wala.core` or `wala.shrike`.
2.  If you make any change which might affect the CAst system,
    additionally run the following drivers:
    1.  `com.ibm.wala.cast.test-JUnit`
    2.  `com.ibm.wala.cast.java.test-JUnit`
    3.  `com.ibm.wala.cast.js.test-JUnit`
3.  If your check-in is non-trivial, please create additional JUnit
    tests as required to make sure the change works as intended.
4.  All tests are expected to pass. Every time. Do not check in code if
    any JUnit test fails.
5.  All tests are expected to pass. Every time. Do not check in code if
    any JUnit test fails.
6.  Don't Forget ....
    1.  All tests are expected to pass. Every time. Do not check in code
        if any JUnit test fails.
7.  If your change is non-trivial, add a note on the wiki for the
    Release Notes for the next release.