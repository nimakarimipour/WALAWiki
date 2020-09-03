---
title: Coding Conventions
permalink: /Coding_Conventions/
---

1.  Normal Sun-style Java coding conventions, with **2-space indenting**
    and **132 columns**. Please indent with two spaces and don't use
    tabs. If using Eclipse, you can import the following coding styles
    [`codeTemplates.xml`](http://svn.sourceforge.net/viewvc/wala/com.ibm.wala.core/dat/codetemplates.xml?view=markup)
    and
    [`formatter.xml`](http://svn.sourceforge.net/viewvc/wala/com.ibm.wala.core/dat/formatter.xml?revision=140&view=markup)
    from the `dat/` directory of `com.ibm.wala.core`.
2.  Be extremely liberal with comments and javadoc.
3.  All code must compile against JDK 5.0.
4.  No code should ever write to `System.out` or `System.err`. Use the
    `com.ibm.wala.util.debug.Trace` facility to write debugging and
    trace messages.
5.  Use the `com.ibm.wala.util.debug.Assertions` class liberally. All
    calls to `assert()` must be guarded by
    `Assertions.verifyAssertions`. Use the `productionAssert`
    entrypoints for assertions that should be enabled in production, and
    thus not guarded.
6.  Do not explicitly throw exceptions or errors ... any such conditions
    should be redirected through the `Assertions` class .. such as
    `Assertions.UNREACHABLE()`.
7.  All Eclipse code should work with Eclipse 3.3.
8.  All code should be deterministic if possible ... avoid
    `System.identityHashCode` and finalizers.
9.  Any shortcuts, assumptions, or analysis problems should be recorded
    in the static `Warnings` dictionary.
10. `Assertions.verifyAssertions` should be true in HEAD until further
    notice.
11. Each Java file should have a header similar to the one below. You do
    not have to assign the copyright to IBM. However if you want to,
    feel free.

Template for Java file header
-----------------------------

`/*******************************************************************************`
` * Copyright (c) 2002 - 2006 IBM Corporation`
` * [Or Copyright(c) 200x Your name]`
` * All rights reserved. This program and the accompanying materials`
` * are made available under the terms of the Eclipse Public License v1.0`
` * which accompanies this distribution, and is available at`
` * `[`http://www.eclipse.org/legal/epl-v10.html`](http://www.eclipse.org/legal/epl-v10.html)
` *`
` * Contributors:`
` *     IBM Corporation - initial API and implementation`
` *     OR your name.`
` *******************************************************************************/`