---
title: UserGuide:Troubleshooting
permalink: /UserGuide:Troubleshooting/
---

failed to load root \<Primordial,Ljava/lang/Object\> of class hierarchy
-----------------------------------------------------------------------

If you see the following runtime error when running WALA:

`  com.ibm.wala.ipa.cha.ClassHierarchyException: failed to load root <Primordial,Ljava/lang/Object> of class hierarchy`

Probably the `java_runtime_dir` property in your `wala.properties` file
is incorrect.

freeze() is undefined
---------------------

Seeing build errors like:

`  The method freeze() is undefined for the type CorePackageImpl`

You have the wrong version of EMF installed. You currently need
emf-sdo-xsd-SDK-2.1.0.

properties or ecore files not found
-----------------------------------

`wala.properties` not found, or wrong `wala.properties` found.

The Eclipse build process is *supposed* to copy the contents of
`com.ibm.wala.core/dat` to `com.ibm.wala.core/bin` when the
`com.ibm.wala.core project` is built. More than one user has reported
that this copy does not happen their Eclipse environment. Apparently the
Eclipse workspace doesn't notice the new properties files after you copy
and edit them. Forcing a refresh of the dat/ directories in the
workspace seems to fix it. '' (Thanks for the tip, David
Greenfieldboyce!)''

This problem has also manifested as WALA failing to find `ecore` files.
Eclipse is supposed to copy ecore files from the projects to bin
directories.

Other problems with properties files not found? Check filenames
carefully! Watch out especially for unintended spaces in filenames.
(Don't search for "`wala.core.tests.properties`").

IProgressMonitor not found
--------------------------

If you build your own Java project that uses WALA, you may have to futz
with the class path a little. It seems that if you create projects as
PDE projects, the PDE environment figures out dependencies correctly,
but not necessarily so for plain old Java projects.

Anyway, the IProgressMonitor class is found in the
org.eclipse.equinox.common project.

Unsatisfied link error for SWT libraries
----------------------------------------

` Exception in thread "Thread-1" java.lang.UnsatisfiedLinkError: no swt-win32-3235 in java.library.path`

Try launching your application as an "SWT Application" in the Eclipse
`Run...` wizard.

Other problems?
---------------

Feel free to [open a
defect](http://sourceforge.net/tracker/?group_id=176742&atid=878458) or
[ask on the mailing list.](mailto:wala-wala@lists.sourceforge.net)