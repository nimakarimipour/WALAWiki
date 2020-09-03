---
title: UserGuide:NativeCode
permalink: /UserGuide:NativeCode/
---

Native Code
===========

WALA currently models native code for some library classes with
flow-insensitive summaries. These summaries are expressed in XML in
`natives.xml`. The `natives.xml` file represents methods that were
actually native in a 1.3 JDK.

The current native models are incomplete, and more importantly,
inadequate for flow-sensitive analysis, since the language used in the
XML files is flow-insensitive. We hope to transition to a better model
someday.