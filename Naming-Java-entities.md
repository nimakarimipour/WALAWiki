In bytecode, there can be more than one name for a single given entity that arises at runtime.

For example, suppose you have `class A extends class B extends java.lang.Object`, and neither `A` nor `B` overrides `java.lang.Object.toString()`.

The following are all legal names for a method that might arise in the bytecode:

`   * `< Application, A, toString() >
`   * `< Application, B, toString() >
`   * `< Application, Ljava/lang/Object, toString() >
`   * `< Primordial, Ljava/lang/Object, toString() >

However at run-time, these will all be resolved to a single entity: &lt;tt&gt;< Primordial,  Ljava/lang/Object, toString() ></tt>

In WALA, an [`IMethod`] represents the unique entity that will arise at run-time, while a [`MethodReference`] can represent any of the possible names that arise in the bytecode.

Similarly, an [`IClass`] represents the unique entity that will arise at run-time, while a [`TypeReference`] can represent any of the possible names for a type that arise at runtime. Note that in Java the unique name of a class is actually a pair of a *classloader x packagename.typename*. So a `TypeReference` actually consists of a [`ClassLoaderReference`] x [`TypeName`].

There's similar logic between [`FieldReference`] and [`IField`], and [`ClassLoaderReference`] and [`IClassLoader`].

A [`ClassHierarchy`] object allows for resolving references to run-time entities (<em>e.g.</em>, getting an [`IMethod`] object from a [`MethodReference`]): see [`ClassHierarchy.resolveMethod()`], [`ClassHierarchy.resolveField()`], etc. (see <UserGuide:ClassHierarchy> for more details on class hierarchies).

Note that most names of entities (e.g. `TypeReference`, `TypeName`, `MethodReference`) are canonicalized via hash-consing, to save space.

  [`FieldReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/FieldReference.html
  [`IField`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IField.html
  [`ClassLoaderReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html
  [`IClassLoader`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClassLoader.html
  [`ClassHierarchy`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html
  [`IMethod`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IMethod.html
  [`MethodReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/MethodReference.html
  [`ClassHierarchy.resolveMethod()`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html#resolveMethod(com.ibm.wala.types.MethodReference)
  [`ClassHierarchy.resolveField()`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html#resolveField(com.ibm.wala.types.FieldReference)

  [wala.core technical overview]: wala.core_technical_overview "wikilink"
  [`IMethod`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IMethod.html
  [`MethodReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/MethodReference.html
  [`IClass`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClass.html
  [`TypeReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/TypeReference.html
  [`ClassLoaderReference`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html
  [`TypeName`]: http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/TypeName.html