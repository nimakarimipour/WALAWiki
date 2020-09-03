---
title: Getting Started:JavaScript frontend
permalink: /Getting_Started:JavaScript_frontend/
---

We plan to improve this page with more details on how to get started
with the JavaScript frontend. Feel free to make suggestions on the
mailing list for what could be improved.

Getting the code
----------------

The JavaScript front end makes use of
[`Rhino`](http://www.mozilla.org/rhino/) to parse JavaScript and create
ASTs. The JavaScript front end consists of the projects below, along
with the core CAst projects and the WALA projects.

` * com.ibm.wala.cast.js (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.js)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.js)`)`
`   o The CAst-based JavaScript front end`
` * com.ibm.wala.cast.js.rhino`
`   o Parsing code specific to Rhino`
` * com.ibm.wala.cast.js.test (`[`javadoc`](http://wala.sourceforge.net/javadocs/com.ibm.wala.cast.js.test)`) (`[`source`](https://github.com/wala/WALA/tree/master/com.ibm.wala.cast.js.test)`)`
`   o Tests for the CAst-based JavaScript front end`
` * com.ibm.wala.cast.js.rhino.test`
`   o Test code specific to Rhino`

For legal reasons, you will need to obtain version 1.7R3 of Rhino from
[here](http://ftp.mozilla.org/pub/mozilla.org/js/rhino1_7R3.zip) and
install it as `js.jar` in the `lib/` directory of
`com.ibm.wala.cast.js.rhino`. Running `ant` on
`com.ibm.wala.cast.js.rhino/build.xml` should fetch the jar for you.
Similarly, run `ant` on `com.ibm.wala.cast.js/build.xml` to fetch the
[`Jericho`](http://jericho.htmlparser.net/docs/index.html) and
[nu.validator](http://about.validator.nu/htmlparser/) HTML parsers,
another required dependence.

The test data project also requires some steps to build. Once again for
legal reasons, we do not include some test program source in the
project, so you must obtain that separately. Run `ant` on
`com.ibm.wala.cast.js.test.data/build.xml`
(`com.ibm.wala.cast.js.test/build.xml` in 1.3.4) to fetch the additional
test sources. Once you are set up, you should be able to run the JUnit
tests by executing the `com.ibm.wala.cast.js.rhino.test-JUnit` launcher
(see Run -\> Run Configurations... -\> JUnit in Eclipse).

Building call graphs
--------------------

You can use the following code to create a call graph for a stand-alone
JavaScript file:

    // use Rhino for parsing; change if you want to use a different parser
    com.ibm.wala.cast.js.ipa.callgraph.JSCallGraphUtil.setTranslatorFactory(new CAstRhinoTranslatorFactory());
    CallGraph CG = com.ibm.wala.cast.js.test.JSCallGraphBuilderUtil.makeScriptCG("dir", "file.js");

The call graph has a "fake" root node, and its successors are
[`prologue.js`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast.js/dat/prologue.js),
which contains models of built-in JS functions, and your JS files. For
JavaScript executed in a browser, WALA additionally includes a
[`preamble.js`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast.js/dat/preamble.js)
file modeling certain aspects of the DOM and its APIs. You can create a
call graph for all the JavaScript loaded from some HTML file as follows:

    com.ibm.wala.cast.js.ipa.callgraph.JSCallGraphUtil.setTranslatorFactory(new CAstRhinoTranslatorFactory());
    CallGraph CG = com.ibm.wala.cast.js.test.JSCallGraphBuilderUtil.makeHTMLCG(url_of_html_file);

(You can get a URL for a local `File f` by calling `f.toURI().toURL()`.)
[`HTMLCGBuilder`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast.js.rhino.test/harness-src/com/ibm/wala/cast/js/rhino/test/HTMLCGBuilder.java)
in the `com.ibm.wala.cast.js.rhino.test` project is a command-line
driver for building a call graph for the JS code referenced from an HTML
file. See the docs on the `main` method for command-line parameters. The
code in `HTMLCGBuilder.buildHTMLCG()` may also be of interest for
writing your own driver; in particular it shows how to enable the
techniques described in the ECOOP'12 paper [Correlation Tracking for
Points-To Analysis of
JavaScript](http://researcher.ibm.com/researcher/files/us-msridhar/ECOOP12Correlation.pdf).

For each JS file *f*, the call graph will contain a `CGNode`
representing the top-level code in *f*, and functions declared within
*f* are distinguished by having *f* as part of the WALA representation
of the method name. You can find the top-level `CGNode` for your file as
follows:

    private CGNode getFunctionNode(CallGraph CG, String dir, String file) {
      TypeName type = TypeName.findOrCreate("L" + dir + "/" + file);
      if (CG != null) {
        Iterator<CGNode> iter = CG.iterator();
        CGNode node;
        while (iter.hasNext()) {
          node = iter.next();
          TypeName tempType = node.getMethod().getDeclaringClass().getName();
          if (tempType.equals(type)) {
            return node;
          }
        }
      }
      System.err.println("Can't find :" + dir + "/" + file);
      return null;
    }

For further technical details on how WALA constructs call graphs for
JavaScript, see [CAst CallGraph
Details](/CAst_CallGraph_Details "wikilink").

Building IRs
------------

Given a `CallGraph CG`, you can iterate over each method's IR
instructions as follows:

    for (CGNode node: CG) {
            // Get the IR of a CGNode
            IR ir = node.getIR();

            // Get CFG from IR
            SSACFG cfg = ir.getControlFlowGraph();

            // Iterate over the Basic Blocks of CFG
            Iterator<ISSABasicBlock> cfgIt = cfg.iterator();
            while (cfgIt.hasNext()) {
              ISSABasicBlock ssaBb = cfgIt.next();

              // Iterate over SSA Instructions for a Basic Block
              Iterator<SSAInstruction> ssaIt = ssaBb.iterator();
              while (ssaIt.hasNext()) {
                SSAInstruction ssaInstr = ssaIt.next();
                //Print out the instruction
                System.out.println(ssaInstr);
              }
            }
    }

You can also construct IRs for all methods without building a call
graph. This technique is used, e.g., by the
[`CorrelationFinder`](https://github.com/wala/WALA/blob/master/com.ibm.wala.cast.js/source/com/ibm/wala/cast/js/ipa/callgraph/correlations/CorrelationFinder.java)
class. Here is some code, lifted from `CorrelationFinder`, for doing so,
given a `URL url` for an HTML file (some imports have been elided):

        // add in preamble.js for modeling of DOM APIs
        JavaScriptLoader.addBootstrapFile(WebUtil.preamble);
        Set<? extends SourceModule> script = WebUtil.extractScriptFromHTML(url);
        SourceModule[] scripts = script.toArray(new SourceModule[script.size()]);
        WebPageLoaderFactory loaders = new WebPageLoaderFactory(translatorFactory);
        CAstAnalysisScope scope = new CAstAnalysisScope(scripts, loaders, Collections.singleton(JavaScriptLoader.JS));
        IClassHierarchy cha = ClassHierarchy.make(scope, loaders, JavaScriptLoader.JS);
        // to bail out early in the case of parse errors
        Util.checkForFrontEndErrors(cha);
        IRFactory<IMethod> factory = AstIRFactory.makeDefaultFactory();
        for(IClass klass : cha) {
          for(IMethod method : klass.getAllMethods()) {
            IR ir = factory.makeIR(method, Everywhere.EVERYWHERE, SSAOptions.defaultOptions());
            // do what is needed with the IR
          }
        }

Prototype Chain
---------------

1.  Each object has an explicit prototype field which holds a pointer to
    its prototype. In the case of a chain of prototypes, the prototype
    object itself will have a prototype, and so on. The properties are
    initialized in the constructors of the different kinds of objects;
    this logic is generated in
    `com.ibm.wala.cast.js.ipa.callgraph.JavaScriptConstructTargetSelector`.
2.  In JavaScript, reads of properties follow the prototype chain but
    writes do not. Prototype chain lookup used to be modeled by
    generating a loop in the WALA IR, but this has changed in recent
    versions. Now, we generate a
    `com.ibm.wala.cast.js.ssa.PrototypeLookup` instruction for all
    property reads, and analyses are expected to model the prototype
    lookup semantics for this instruction (call graph construction
    already does so).