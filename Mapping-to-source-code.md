Some notes on mapping back to source when analyzing bytecode. This does
not include the CAst functionality.

This page is incomplete.

Getting the source file name 
----------------------------

Note that `IClass` has a method `getSourceFileName()`. The
`ClassLoaderImpl` attempts to set up a mapping between IClasses and
source files during class loading. After loading all classes, the class
loader goes back and looks at each `.java` file in the scope, and maps
it to a loaded `IClass`. So, in order to use `getSourceFileName()`, be
sure to include the source files in the analysis scope.

Mapping value numbers back to local names 
-----------------------------------------

If the information is available, you can retrieve it via
`IR.getLocalNames()`. Note that not every value number corresponds to a
local variable in Java source code. Some value numbers may represent
multiple local variables, as a result of copy propagation during IR
construction.

From IR instruction to bytecode index / source line number 
----------------------------------------------------------

Given the index `i` of some `SSAInstruction` in an `IR ir` produced from
bytecode, you can get the corresponding bytecode index and source line
number as follows:
```java
    IBytecodeMethod method = (IBytecodeMethod)ir.getMethod();
    int bytecodeIndex = method.getBytecodeIndex(i);
    int sourceLineNum = method.getLineNumber(bytecodeIndex);
```

Note that source line information may not always be available.

From Slices to source line numbers 
----------------------------------

Given a `Statement` that came from a Java bytecode file (via Shrike),
you can get the line number in the original source with the following:

```java
if (s.getKind() == Statement.Kind.NORMAL) { // ignore special kinds of statements
  int bcIndex, instructionIndex = ((NormalStatement) s).getInstructionIndex();
  try {
    bcIndex = ((ShrikeBTMethod) s.getNode().getMethod()).getBytecodeIndex(instructionIndex);
    try {
      int src_line_number = s.getNode().getMethod().getLineNumber(bcIndex);
      System.err.println ( "Source line number = " + src_line_number );
    } catch (Exception e) {
      System.err.println("Bytecode index no good");
      System.err.println(e.getMessage());
    }
  } catch (Exception e ) {
    System.err.println("it's probably not a BT method (e.g. it's a fakeroot method)");
    System.err.println(e.getMessage());
  }
}
```

For `Statement`s that came from a Java source file thru cast, there is
no bytecode index, so (despite what the javadoc may lead you to believe)
the parameter to `getLineNumber()` is simply an instruction index:

```java
if (s.getKind() == Statement.Kind.NORMAL) {
  int instructionIndex = ((NormalStatement) s).getInstructionIndex();
  int lineNum = ((ConcreteJavaMethod) s.getNode().getMethod()).getLineNumber(instructionIndex);
  System.out.println("Source line number = " + lineNum );
}
```

This is useful when investigating the output of
[[slicing|Slicer]].

Getting the .jar file for an IClass
-----------------------------------

If you have an IClass klass representing a .class file inside a .jar,
you can do the following to figure out the .jar file name:

```java
 ShrikeClass shrikeKlass = (ShrikeClass) klass;
 JarFileEntry moduleEntry = (JarFileEntry) shrikeKlass.getModuleEntry();
 String jarFile = moduleEntry.getJarFile();
```

Note that the casts may fail if it's not a .class file inside a .jar.