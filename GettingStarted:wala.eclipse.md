---
title: GettingStarted:wala.eclipse
permalink: /GettingStarted:wala.eclipse/
---

*Note for developers*: the `wala.eclipse` functionality is newer and
less mature than the rest of the WALA, and so the APIs may be subject to
more frequent churn and change. WALA newbies may want to experiment with
the rest of WALA first, before diving into the guts of `wala.eclipse`.

Getting started with WALA for Eclipse
-------------------------------------

WALA now includes at least one example application that contributes
fuctionality to an Eclipse workbench. (Thanks, Annie Ying!). The current
example provides a call graph view, linked to the Eclipse workspace,
built by WALA. This example also illustrates how the CAst System is
integrated into WALA, so you need to have that---both the JDT support
for Java and JavaScript---to build the Eclipse projects, which are the
following:

` * com.ibm.wala.eclipse (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.eclipse)`) (`[`source`](http://wala.svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.eclipse)`)`
`   o The current examples of Eclipse integration of WALA`
` * com.ibm.wala.eclipse.tests (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.eclipse.tests)`) (`[`source`](http://wala.svn.sourceforge.net/viewvc/wala/incubator/com.ibm.wala.eclipse.tests)`)`
`   o Tests of the current examples of Eclipse integration of WALA`

Running the Example
-------------------

This example builds a call graph and displays as a tree view in the
Eclipse IDE.

Before running this example, check out the additional project
`com.ibm.wala.eclipse` to your Eclipse workspace. Set up the WALA
properties as you would for [non-Eclipse
examples](http://wala.sourceforge.net/wiki/index.php/UserGuide:Getting_Started#Configuring_WALA_properties).
Now run the `wala.eclipse` launcher. This should launch another instance
of Eclipse. The workspace in this new Eclipse instance is likely empty;
you should import whatever projects or code you would like to explore.

(For javascript, Julian's favorite example is to get ajaxslt from
sourceforge (http://goog-ajaxslt.sourceforge.net/), and then run the
call graph viewer on the test pages, test/xpath.html and
test/xslt.html.)

The call graph view will add an `Open WALA Call Tree` option to the
pop-up menu obtained by right-clicking in the Package Explorer view for
four kinds of items:

-   Files ending in the .jar extension


This will build a call graph using the classes in the .jar file as the
analysis scope, and all main methods in classes in the jar as entry
points

-   Java projects


This will build a call graph using all classes in the project as the
analysis scope, and all main methods in the project as entry points

-   Individual classes within Java projects


This will build a call graph using the classes in the enclosing project
as the analysis scope and the main method of the selected class as the
entry point

-   Files ending in the .html extension


This will build a call graph of the JavaScript code in the selected html
file, using call backs declared in the page as the entry points.

To activate the view, first right click the on the desired target, say
in the Package Explorer view. Then, select `WALA Call Tree` from the
pop-up menu. Alternatively, you can get the view through the `Show View`
dialog (select the jar file; then go to the `Window` menu --\>
`Show View` --\> `Other...` --\> `Java` --\> `WALA Call Tree`). For a
Java project with source code, double-clicking on a method node or call
node in the view should respectively highlight the corresponding method
declaration or method call in the source.

Note that one characteristic of the current WALA Call Tree view is that
it is not currently refreshed: once it is open, it stays with the same
view, ignoring any further attempts to view a call tree. To view a new
tree, it is necessary to explicitly close the current view before
requesting a new tree. This should be changed at some point.