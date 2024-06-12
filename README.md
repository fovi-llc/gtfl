GTFL - A Graphical Terminal For Lisp
====================================

### Abstract

GTFL is a graphical terminal for Common Lisp. The client is a html page running in a web browser and GTFL provides mechanisms for sending content to the client page from within Lisp (using [HUNCHENTOOT](http://www.weitz.de/hunchentoot/) (redirects to https://edicl.github.io/hunchentoot/) and [HT-SIMPLE-AJAX](https://martin-loetzsch.de/ht-simple-ajax)).

It is meant for Lisp programmers who want to debug or visualize their own algorithms. Instead of printing tracing information to the Lisp listener (which everybody normally does to understand what's going on), more readable graphical representations of the algorithm's internals are sent to the GTFL client page.

GTFL also comes with mechanisms for visualizing complex (hierarchical) data or control structures. It provides functions for drawing trees and for hiding complexity in elements that expand when the user clicks on them.

Two real-life examples for an application of GTFL can be found [here](https://www.martin-loetzsch.de/gtfl/application-example-1) (or [html/application-example-1.html](https://html-preview.github.io/?url=https://github.com/fovi-llc/gtfl/blob/main/html/application-example-1.html)) and [here](https://www.martin-loetzsch.de/gtfl/application-example-2) (or [html/application-example-2.html](https://html-preview.github.io/?url=https://github.com/fovi-llc/gtfl/blob/main/html/application-example-2.html)). These are debug traces of linguistic parsing and production in the [Fluid Construction Grammar](http://fcg-net.org) (FCG) framework. Such traces help the developers FCG to understand and debug their programs and they help users of FCG to see what their linguistic rules are doing. By encapsulating visualizations for representations in expandable html elements, the complete trace fits into one browser window and still includes every little detail and intermediate processing step of the involved FCG algorithms (which would be thousands of pages debugging output to the listener).

#### GitHub Copy

[jimwhite@](https://github.com/jimwhite) copied the archive https://martin-loetzsch.de/gtfl/gtfl.tar.gz with version 0.1.3 on June 11, 2024 from the homepage https://www.martin-loetzsch.de/gtfl/ to https://github.com/fovi-llc/gtfl.  This README is an incomplete conversion of the [index.html](https://html-preview.github.io/?url=https://github.com/fovi-llc/gtfl/blob/main/index.html) file which has all the details formatted by GTFL itself.  I've updated (most of) the HTTP URLs to HTTPS.

### Contents

1.  [Download and installation](#download-and-installation)
2.  [Browser compatibility](#browser-compatibility)
3.  [The GTFL terminal](#the-gtfl-terminal)
    1.  [start-gtfl](#start-gtfl)
    2.  [\*gtfl-address\*](#_gtfl-address_)
    3.  [\*gtfl-port\*](#_gtfl-port_)
    4.  [gtfl-out](#gtfl-out)
    5.  [replace-element-content](#replace-element-content)
    6.  [append-to-element](#append-to-element)
    7.  [reset-gtfl](#reset-gtfl)
    8.  [\*reset-functions\*](#_reset-functions_)
    9.  [who](#who)
    10.  [who2s](#who2s)
    11.  [who-lambda](#who-lambda)
    12.  [define-css](#define-css)
    13.  [define-js](#define-js)
    14.  [make-id-string](#make-id-string)
4.  [Expandable elements](#expandable-elements)
    1.  [make-expandable/collapsable-element](#make-expandable_collapsable-element)
    2.  [make-expand/collapse-link](#make-expand_collapse-link)
    3.  [make-expand/collapse-all-link](#make-expand_collapse-all-link)
    4.  [\*create-static-expandable/collapsable-elements\*](#_create-static-expandable_collapsable-elements_)
5.  [Tree drawing](#tree-drawing)
    1.  [draw-node-with-children](#draw-node-with-children)
6.  [Resizing s-expressions](#resizing-s-expressions)
    1.  [html-pprint](#html-pprint)
7.  [Acknowledgements](#acknowledgements)

### Download and installation

GTFL together with examples and this documentation can be downloaded from [https://martin-loetzsch.de/gtfl/gtfl.tar.gz](https://martin-loetzsch.de/gtfl/gtfl.tar.gz). The current version is 0.1.3.

GTFL directly relies on [CL-WHO](https://www.weitz.de/cl-who/) for html generation, [HUNCHENTOOT](https://www.weitz.de/hunchentoot/) (version >= 1.1.0) for running the web server and [HT-SIMPLE-AJAX](https://martin-loetzsch.de/ht-simple-ajax) for the asynchrounous client/server communication. And these libraries themselves require quite a number of other libraries (see the dependency graph [above](#asdf-dependencies)). Make sure you have recent versions of everything.

If you don't want to download all these libraries manually, you can use [Quicklisp](https://www.quicklisp.org/) or [ASDF-INSTALL](https://www.cliki.net/ASDF-Install):

(ql:quickload "gtfl")

(asdf-install:install 'gtfl)

Once everything is installed, GTFL is compiled and loaded with:

(asdf:operate 'asdf:load-op :gtfl)

### Browser compatibility

As of 2012, all contemporary web browsers except Internet Explorer work well with gtfl.

The output of GTFL is XHTML 1.0 Strict and CSS level 2.1 conform as can be checked below (this page is a document created with GTFL):  
   [![Valid XHTML 1.0 Strict](valid-xhtml10.png)](https://validator.w3.org/check?uri=https://martin-loetzsch.de/gtfl/)   [![Valid CSS level 2.1](valid-css21.gif)](https://jigsaw.w3.org/css-validator/validator?uri=https://martin-loetzsch.de/gtfl/)

### The GTFL terminal

GTFL consists of two main components that are defined in [_gtfl.lisp_](gtfl.lisp): a html client page running in a web browser and a Lisp web server that mediates between your program and the client page. In order to push stuff from Lisp to the client, the Lisp side of GTFL maintains a "request list" into which the output routines put their content. The client page has a continuously running event loop that polls this list every 200ms using asynchronous AJAX calls.

Here's now some basic usage examples:

([start-gtfl](#start-gtfl))

This starts the web server. You should see now the client page at [http://localhost:8000](http://localhost:8000) (or the address and port you have set).

Now you can push any content there, for example

([gtfl-out](#gtfl-out) (:h1 "hello world"))

\=>This will show up in the client page:

hello world
===========

GTFL uses [CL-WHO](https://www.weitz.de/cl-who/) for rendering s-expressions into XHTML. If you are not familiar with CL-WHO then read its documentation first.

More examples:

([gtfl-out](#gtfl-out) (:p "some text, " (:span :style "color:red;" "and some in red."))
          (:p "and a second paragraph"))

\=>

some text, and some in red.

and a second paragraph

(defparameter \*element-id\* nil)

([gtfl-out](#gtfl-out) (:p "a paragraph, " 
              (:span :id (setf \*element-id\* ([make-id-string](#make-id-string))) :style "border:1px solid red"
                     "and a span as child element")))

\=>

a paragraph, and a span as child element

([replace-element-content](#replace-element-content) \*element-id\* "and " (:b "new") " span content")

\=>

a paragraph, and **new** span content

([gtfl-out](#gtfl-out) (:p "a paragraph, " 
              (:span :id (setf \*element-id\* ([make-id-string](#make-id-string))) :style "border:1px solid red"
                     "and a span as child element")))

\=>

a paragraph, and a span as child element

([append-to-element](#append-to-element) \*element-id\* ", and " (:b "more") " content")

\=>

a paragraph, and a span as child element, and **more** content

\[Special variable\]  
**\*gtfl-address\***

The address to use for the web server. Default: "localhost".

\[Special variable\]  
**\*gtfl-port\***

The port to use for the web server. Default: 8000.

\[Function\]  
**start-gtfl** \=> _no values_

Starts the web server at the specified address.

\[Macro\]  
**gtfl-out** _&rest expressions_ \=> _no values_

Adds some content to the bottom of the client page.

_expressions_ is something that's ok within CL-WHO's with-html-output macro.

See examples above.

\[Macro\]  
**replace-element-content** _id &rest expressions_ \=> _no values_

Replaces the content of the element with _id_ (a string) by _expressions_.

See examples above.

\[Macro\]  
**append-to-element** _id &rest expressions_ \=> _no values_

Appends _expressions_ to the element with _id_.

See examples above.

\[Function\]  
**reset-gtfl** \=> _no values_

Clears the content area of the client page and resets things on the Lisp side.

This function is called either

*   directly,
*   when the 'reset' button on the client page was clicked,
*   when the client page is (re)loaded.

\[Special variable\]  
**\*reset-functions\***

A list of functions that are called by [reset-gtfl](#reset-gtfl). If you want to reset some of your stuff in this case, add your function here: (pushnew #'my-reset-function \*reset-functions\*).

\[Macro\]  
**who** _&rest expressions_ \=> _no values_

Writes rendered html to \*standard-output\*. This is a shortcut for (with-html-output (\*standard-output\*) <expressions>).

_expressions_ is something that's ok within CL-WHO's with-html-output macro.

Example:

GTFL> ([who](#who) (:div :foo "bar" "baz"))
<div foo="bar">baz</div>
NIL

\[Macro\]  
**who2s** _&rest expressions_ \=> _html string_

Renders expression into a html string. This is a shortcut for (with-html-output-to-string (\*standard-output\*) <expressions>).

Example:

GTFL> ([who2s](#who2s) (:div :foo "bar" "baz"))
"<div foo="bar">baz</div>"

\[Macro\]  
**who-lambda** _&rest expressions_ \=> _anonymous function_

Makes an anonymous function that writes the rendered html to \*standard-output\*.

Example:

GTFL> ([who-lambda](#who-lambda)) (:div :foo "bar" "baz"))
#<Anonymous Function #x300043366A4F>
GTFL> (funcall \*)
<div foo="bar">baz</div>
NIL

\[Function\]  
**define-css** _id css_ \=> _no values_

Adds css fragments to the client page. When the client page is created, all these code fragments are concatenated into one big inline css. Note that you will have to reload the client page when you change css definitions.

_id_ is an id for the code fragment. Repeated calls with the same id overwrite previous definitions. _css_ is the css code fragment. You are responsible for adding line endings.

Example:

([define-css](#define-css) 'foo "
div.foo {font-weight:bold;}
")

\[Function\]  
**define-js** _id js_ \=> _no values_

The same as [define-css](#define-css) above, but for javascript code.

([define-js](#define-js) 'bar "
function bar () { return (1 + 1); }
")

\[Function\]  
**make-id-string** _&optional base_ \=> _string_

Creates an uniquely numbered id string that can be used as an id for html elements.

_base_ is the prefix of the id (default: "id").

Example:

GTFL> ([make-id-string](#make-id-string))
"id-1"
GTFL> ([make-id-string](#make-id-string))
"id-2"
GTFL> ([make-id-string](#make-id-string) "foo")
"foo-1"
GTFL> ([make-id-string](#make-id-string) "foo")
"foo-2"

### Expandable elements

The file [_expandable-elements.lisp_](expandable-elements.lisp) contains functionality to create html elements that expand when the user clicks on them and that collapse again when they are clicked a second time. Furthermore, several elements can be expanded/ collapsed at once using another button.

In order to save browser resources (memory, time), the expanded and collapsed version of each element are kept in Lisp and are only sent to the browser when needed: the version that is initally sent to the client contains only the collapsed version and when the expand link is clicked, it is replaced with the expanded version stored on the lisp side.

Example:

([gtfl-out](#gtfl-out) 
 (:div 
   (let ((expand/collapse-all-id ([make-id-string](#make-id-string)))
         ([\*create-static-expandable/collapsable-elements\*](#_create-static-expandable_collapsable-elements_) t))
         ([make-expandable/collapsable-element](#make-expandable_collapsable-element) 
          ([make-id-string](#make-id-string)) expand/collapse-all-id
          ([who2s](#who2s) ([make-expand/collapse-all-link](#make-expand_collapse-all-link) expand/collapse-all-id t nil "expand all"))
          ([who2s](#who2s) ([make-expand/collapse-all-link](#make-expand_collapse-all-link) expand/collapse-all-id nil nil "collapse all")))
         (loop repeat 3
            for element-id = ([make-id-string](#make-id-string))
            do (htm (:div :style "border:1px solid #aaa;display:inline-block;margin-left:10px;"
                          ([make-expandable/collapsable-element](#make-expandable_collapsable-element) 
                           element-id expand/collapse-all-id
                           ([who2s](#who2s) (:div ([make-expand/collapse-link](#make-expand_collapse-link) element-id t nil "expand")
                                        (:br) "collapsed"))
                           ([who2s](#who2s) (:div ([make-expand/collapse-link](#make-expand_collapse-link) element-id nil nil "collapse") 
                                        (:br) (:div :style "font-size:150%" "expanded"))))))))))

\=>

[expand all](javascript:expandAllStatic('id-16'); "expand all")

[collapse all](javascript:collapseAllStatic('id-16'); "collapse all")

[expand](javascript:expandStatic('id-18'); "expand")  
collapsed

[collapse](javascript:collapseStatic('id-18'); "collapse")  

expanded

[expand](javascript:expandStatic('id-19'); "expand")  
collapsed

[collapse](javascript:collapseStatic('id-19'); "collapse")  

expanded

[expand](javascript:expandStatic('id-20'); "expand")  
collapsed

[collapse](javascript:collapseStatic('id-20'); "collapse")  

expanded

\[Function\]  
**make-expandable/collapsable-element** _element-id expand/collapse-all-id collapsed-element expanded-element &key expand-initially_ \=> _no values_

Creates an element that allows to switch between an expanded and a collapsed version.

_element-id_ (a string) is the id given to the element so that function [make-expand/collapse-link](#make-expand_collapse-link) can reference it. _expand/collapse-all-id_ (a string) is the id of a group of elements that can be expanded/collapsed at the same time (see [make-expand/collapse-all-link](#make-expand_collapse-all-link)).

_collapsed-element_ and _expanded-element_ are the collapsed and expanded version of the element. They can be either an expression that evaluates to a html string, e.g. "<div foo/>" or ([who2s](#who2s) :div "foo"), or an anonymous function that writes an html string, e.g. ([who-lambda](#who-lambda) (:div "foo")) or #'(lambda () (princ "<div foo/>")). In the latter case the expanded version only gets computed when requested by the client, which avoids unneccessary computation.

When _expand-initially_ is t then the expanded version is shown initially.

\[Macro\]  
**make-expand/collapse-link** _element-id expand? title &rest content_ \=> _no values_

Makes a link for expanding/collapsing an element.

_element-id_ is the id of the element to expand or collapse. When _expand?_ is t, then the element gets expanded, otherwise collapsed. _title_ is the title of the link (shown when the mouse is over the link). When nil, then "expand" or "collapse" are used. _body_ is the content of the link (a cl-who expression).

Example:

GTFL> ([make-expand/collapse-link](#make-expand_collapse-link) "foo" t nil "expand")
<a href="javascript:expand('foo');" title="expand">expand</a>
NIL
GTFL> ([make-expand/collapse-link](#make-expand_collapse-link) "foo" nil "click here to collapse" (:b "collapse"))
<a href="javascript:collapse('foo');" title="click here to collapse"><b>collapse</b></a>
NIL

\[Macro\]  
**make-expand/collapse-all-link** _expand/collapse-all-id expand? title &rest content_ \=> _no values_

Makes a link for expanding/collapsing a group of elements.

_expand/collapse-all-id_is the id of the element group to expand/collapse. All other parameters as above.

\[Special variable\]  
**\*create-static-expandable/collapsable-elements\***

When this is set to t, both the expanded and collapsed version will be embedded in the html code (one visible and the other hidden). This makes the html code bigger (and thus rendering slower), but allows to save generated html pages, with the expand/collapse functionality still working when not connected to the web server. Note that in the example above this was also set to t, because otherwise the example would not work in this static html page that is not connected to the lisp server.

### Tree drawing

In [_tree-drawing.lisp_](tree-drawing.lisp) there is a function for recursively drawing trees. It takes html expressions for drawing a node and its children (which can be trees themselves) and connects them with horizontal and vertical lines. The decision how to layout the tree (i.e. how wide to draw each node) is left to the html rendering engine of the web browser.

Examples (resize the browser window to see the dynamic layout in action):

(defparameter \*example-tree\* 
  '("top node."
    ("child node one with three children" 
     ("first out of three children") ("second out of three children") ("third out of three children"))
    ("child node two with one child" 
     ("very long text. very long text. very long text. very long text. 
       very long text. very long text. very long text."))))

(defun draw-node (string) 
  ([who](#who) 
   (:div :style "padding:4px;border:1px solid #888;margin-top:4px;margin-bottom:4px;background-color:#eee;"
         (princ string))))


(defun draw-tree\* (tree) 
  ([draw-node-with-children](#draw-node-with-children) 
   ([who-lambda](#who-lambda) (draw-node (car tree)))
   (mapcar #'(lambda (x) ([who-lambda](#who-lambda) (draw-tree\* x))) (cdr tree))))

([gtfl-out](#gtfl-out) (draw-tree\* \*example-tree\*))

\=>

top node.

child node one with three children

first out of three children

second out of three children

third out of three children

child node two with one child

very long text. very long text. very long text. very long text. very long text. very long text. very long text.

With parameters :right-to-left t :color "green" :style "dotted" \=>

first out of three children

second out of three children

third out of three children

child node one with three children

very long text. very long text. very long text. very long text. very long text. very long text. very long text.

child node two with one child

top node.

With parameters :color "#33a" :line-width "2px" \=>

top node.

child node one with three children

first out of three children

second out of three children

third out of three children

child node two with one child

very long text. very long text. very long text. very long text. very long text. very long text. very long text.

\[Function\]  
**draw-node-with-children** _node children &key right-to-left color width line-width style_ \=> _no values_

A function for recursively drawing trees in html. It draws a node with connections to it's children (which can be trees themselves).

_node_ is a parameterless function that creates the html code for the parent node. _children_ is a list of functions for drawing the children of the node. The reason to use closures instead of for example strings here is both efficiency (no intermediate structures are built) and to allow for re-ordering of elements (when the tree is drawn from right to left.

When _right-to-left_ is t, then the top node of the tree is drawn at the right. _color_ sets the color of the connecting lines (e.g. "#ff0000" or "red", default is "#888"), _width_ the the width of the horizontal connectors (e.g. "25px", default is "10px"), _line-width_ the the thickness of the lines (e.g. "3px", default is "1px") and style the line style ("solid", "dotted" or "dashed", default is "solid").

### Resizing s-expressions

File [_html-pprint.lisp_](html-pprint.lisp) provides the function [html-pprint](#html-pprint) to display s-expressions in html. This could of course be done with (:pre (pprint x)), but this has the disadvantage that the result will have a fixed width. [html-pprint](#html-pprint) produces output that is very similar to the one of pprint but that also dynamically resizes depending on the available width (while maintaining proper indentation).

Examples (resize browser window to see the dynamic relayout and click on the symbols to highlight other symbols with the same name):

(defparameter \*example-list\*
  \`((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) 
     (,(make-symbol "FOO-4") ,(make-symbol "BAR-4"))
     (foo-5 . bar-5))
    ((0 1 2 3 4 5 6 7 8 9) ,(asdf:find-system :gtfl))))

([gtfl-out](#gtfl-out) (:div :style "border:1px solid #aaa;" ([html-pprint](#html-pprint) \*example-list\*)))

\=>

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

([gtfl-out](#gtfl-out)
 (:table :style "border-collapse:collapse;"
         (:tr (loop for i from 1 to 3 
                 do (htm (:td :style "border:1px solid #aaa;"
                              ([html-pprint](#html-pprint) \*example-list\*)))))))

\=>

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

with :max-width 50 \=>

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

with :max-width 100 \=>

((("foo-1" "bar-1") (foo-2 bar-2) (:foo-3 :bar-3) (#:foo-4 #:bar-4) (foo-5 . bar-5)) ((0 1 2 3 4 5 6 7 8 9) #<asdf:system "gtfl">))

\[Function\]  
**html-pprint** _thing &key max-width_ \=> _no values_

Displaying an x-expression in the html. The result lookes like coming from pprint (proper indention), but the layout is dynamic depending on the available width.

_thing_ is the thing to display. _max-width_limits the width of the result (in characters). When nil, then the full available width is used

### Acknowledgements

GTFL was initially developed as part of the Babel2 framework ([https://emergent-languages.org/](https://emergent-languages.org/)).

[Joris Bleys](https://arti.vub.ac.be/~jorisb/) and [Pieter Wellens](https://arti.vub.ac.be/~pieter/) have helped a lot in improving the system by finding bugs and suggesting many of its current features.

This page was created with GTFL itself (see [examples/index-html.lisp](examples/index-html.lisp)). The layout and structure is heavily inspired by (or directly copied from) [DOCUMENTATION-TEMPLATE](http://weitz.de/documentation-template/) (redirects to https://edicl.github.io/documentation-template/).

Last change: 2012/01/25 15:05:34 by [Martin Loetzsch](https://martin-loetzsch.de/)
