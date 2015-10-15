The
[`ClassHierarchy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html)
is the central collection of
[`IClasses`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClass.html)
that define your analysis scope (the program being analyzed).

Class Loaders
-------------

Note that Java provides namespaces for classes called *classloaders*.
WALA's structures mimic the Java class hierarchy namespaces: note that
each `IClass` belongs to an
[`IClassLoader`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/classLoader/IClassLoader.html),
while each
[`TypeReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/TypeReference.html)
specifies a
[`ClassLoaderReference`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html).

Unless you are doing something exotic or analyzing J2EE, you can assume
that WALA is modelling a class hierarchy with two classloaders:

-   [`Application`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html#Application):
    a classloader used to load application code.
-   [`Primordial`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/types/ClassLoaderReference.html#Primordial):
    a classloader used to load J2SE library code (e.g. `java.*`)

Classloaders operate via *delegation*: the classloaders form a tree with
the primordial class loader at the root. So if the `Application` class
loader fails to find a particular class, it will delegate to the
`Primordial` loader to find the class.

So, if you have a class name in hand, and you don't know what loader it
comes from, you can normally look it up in the `Application` loader: the
`IClass` object you receive from
[`ClassHierarchy.lookupClass`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html#lookupClass(com.ibm.wala.types.TypeReference))
will indicate the `IClassLoader` which actually found the class.

Given a string representing a class name, you this would look it up,
trying the `Application` loader first, as follows:

-   `IClass klass = cha.lookupClass(TypeReference.findOrCreate(ClassLoaderReference.Application, s));`

Implementation
--------------

The current
[`ClassHierarchy`](http://wala.sourceforge.net/javadocs/trunk/com/ibm/wala/ipa/cha/ClassHierarchy.html)
implementation is intentionally *mutable*. When modelling synthetic
classes, we occasionally create new classes on the fly depending on
results of analysis, especially for J2EE.