---
title: UserGuide:NamingJavaEntities
permalink: /UserGuide:NamingJavaEntities/
---

[wala.core technical overview](/wala.core_technical_overview "wikilink")

Naming Java entities
====================

In bytecode, there can be more than one name for a single given entity
that arises at runtime.

For example, suppose you have
`class A extends class B extends java.lang.Object`, and neither `A` nor
`B` overrides `java.lang.Object.toString()`.

The following are all legal names for a method that might arise in the
bytecode:

`   * < Application, A, toString() >`
`   * < Application, B, toString() >`
`   * < Application, Ljava/lang/Object, toString() >`
`   * < Primordial, Ljava/lang/Object, toString() >`

However at run-time, these will all be resolved to a single entity:
`< Primordial, Ljava/lang/Object, toString() >`

In WALA, an
[`IMethod`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IMethod.html)
represents the unique entity that will arise at run-time, while a
[`MethodReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/MethodReference.html)
can represent any of the possible names that arise in the bytecode.

Similarly, an
[`IClass`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClass.html)
represents the unique entity that will arise at run-time, while a
[`TypeReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/TypeReference.html)
can represent any of the possible names for a type that arise at
runtime. Note that in Java the unique name of a class is actually a pair
of a *classloader x packagename.typename*. So a `TypeReference` actually
consists of a
[`ClassLoaderReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html)
x
[`TypeName`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/TypeName.html).

There's similar logic between
[`FieldReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/FieldReference.html)
and
[`IField`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IField.html),
and
[`ClassLoaderReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html)
and
[`IClassLoader`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClassLoader.html).

A
[`ClassHierarchy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html)
object allows for resolving references to run-time entities
(<em>e.g.</em>, getting an
[`IMethod`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IMethod.html)
object from a
[`MethodReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/MethodReference.html)):
see
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html#resolveMethod(com.ibm.wala.types.MethodReference>)
`ClassHierarchy.resolveMethod()`\],
\[<http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html#resolveField(com.ibm.wala.types.FieldReference>)
`ClassHierarchy.resolveField()`\], etc. (see
[UserGuide:ClassHierarchy](/UserGuide:ClassHierarchy "wikilink") for
more details on class hierarchies).

Note that most names of entities (e.g. `TypeReference`, `TypeName`,
`MethodReference`) are canonicalized via hash-consing, to save space.