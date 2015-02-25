#Parameterized Libraries

## Contact information

1. **Gilad Bracha.** 

2. **gbracha@google.com.** 

3. **https://github.com/gbracha/parameterizedLibraries** 



##Summary 


This proposal supports the definition and use of libraries whose dependencies are supplied as parameters, supporting multiple library configuration. The proposal is designed such that all libraries remain second class entities determined at compile time.

A parameterized library has the form 

```
library libName( prefixParameter_1, …, prefixParameter_n) {
  top level declarations
}
```

Each prefixParameter defines an external dependency of the library. Prefix parameters 
are named parameters, and are always associated with default values.  A library can be invoked with explicit actual parameters when a non-default binding of a dependency is desired.


##Motivation

We have encountered a host of issues relating to library configuration. Some arise when applications need to work on both client and server, and need to deal with the different Dart platform libraries supported in each case.  These have been the most pressing. However, as usage spread, we are bound to encounter more general situations like this. Dependency injection frameworks were invented as an ad hoc way of coping with such situations. These frameworks are complex and often impose runtime costs. It would be much better to provide language support to deal with this problem.

###Pros


* Provides a standard solution to most dependency injection problems, without relying on different frameworks, reflection etc. Note: I’m not sure this deals with every such problem. The fully general, Newspeak style solution, does eliminate DI. This is more limited.

* Plays well more ambitious extensions such as extensible libraries or first class libraries.


###Cons

* Unconventional. Requires some open mindedness and a learning curve.


##Examples


`library myLib(io, collections)` 

This is the common case. It is much like

```
library myLib;

import ‘dart:io’ as io;
import ‘dart:collections’ as collections;
```

except that it is shorter and at the same time allows alternate configurations of `myLib`. We’ll show how to create configurations in a moment.


We could also write the above example as 

`library myLib(import io, import collections)` 

if we like being very explicit and more verbose.


`library myLib(io, export collections)`

The nearest equivalent would be

```
library myLib;

import ‘dart:io’ as io;
export ‘dart:collections’;
```

again, the new form is shorter and allows alternate configurations.

`library bar(weird: ‘../../weird.dart’ hide [WeirdClass, io])`

This shows we can get an import from any URI, and we can use the normal namespace combinators to control what gets imported, while still being parametric. The use of list delimiters after the combinator prevents the commas from introducing ambiguity.

The non-parametric analog would be

```
library bar;

import ‘../../weird.dart’ as weird hide WeirdClass, io;
```

The point of all this is of course to allow passing of actual parameters to configure libraries.


`library foo(ui(html: html); html)`

Here, we are importing a library `ui` that itself is parametric wrt a library `html`; our `html` library needs to be the same as the one used in the `ui` library. Furthermore, `foo` is itself parametric.


This proposal differs from previous ones in that it does not change the behavior of existing import/export clauses. 

###More Sugar

A variant has been suggested that introduces a sugar

`import id;`

which defines a parameter implicitly, with a default source. Likewise 
`export id` and `deferred import id;`

all can have show/hide clauses, but no as clause, since that is implicitly there already. This sugar is attractive in itself, even without adding parameter. However, we then get into issues of what happens if you have both the sugar and a real parameter list. I would disallow that.

Of course, this sugar can be introduced without the parameterized libraries. It solves nothing but is less verbose and encourages structured use. For example, a prefix can only be used once, and attached to a single library.

###Semantics

A library with  parameters denotes a library function, which is a function from libraries to libraries. A library without parameters denotes a fixed library as it does today.

A prefix parameter is either an identifier or a prefix function call, optionally preceded by one of import, export or library, and optionally followed by either a default binding, a namespace combinator or both. The built-in identifier deferred may appear before the keyword import to indicate that the import is a deferred. If deferred appears by itself it’s a shorthand for deferred import.

The default binding consists of a colon followed by either a URI string as used in import, or a prefix or prefix function call.

It is a compile-time error if any prefix parameter is named libName. All prefix parameters are available in the scope of the library. In the library body they act as prefixes in the same sense as existing prefixes do, This means exports introduce prefixes that can be used to access things;  in that case the name is there but everything is hidden, unless the parameter is also imported.



Prefix identifiers are interpreted as follows:

If *id* is  a prefix identifier, if there is a prefix parameter named *id*, then *id* is interpreted as that parameter; if *id* is *libName*, then it refers to the current library function. Otherwise, if there is a library `dart:id`, then *id* refers the library or library function found at the URI `dart:id`; in any other case *id* refers to the library or library function found at the URI `'package:id/id.dart’`.

When used as a prefix function, the prefix *id* refers to the library function it denotes. When used in isolation, the prefix *id* refers to the library function it denotes instantiated with the library’s default parameters if any. 

A prefix parameter denotes the library supplied as its actual parameter. If no actual parameter is supplied, it denotes the library given by its source, if any, otherwise it is interpreted as a prefix identifier.

If the source of a library is a URI , then if the URI it refers to a library *L*, then the source denotes *L*. If the URI refers to a library function function *F*, it is taken to refer to *F* instantiated with its default parameters. 

If a prefix parameter is preceded by the word **import**, the library imports the value of the parameter prefixed by the parameter name, modified by any following namespace combinators.  If the word **deferred** appears prior to the import, the import is deferred. If a prefix parameter is preceded by the word **export**, the library exports the value of the parameter, modified by any following namespace combinators.  The prefix is in scope, but if it is not imported than it has no visible members.

If no preceding modifier is used, it defaults to **import**.

Need to check that a library does not depend on itself.

### A working implementation

TBD

### Tests

TBD

## Patents rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form]() and submit it to Ecma.

[tex]: http://www.latex-project.org/
[language spec]: https://www.dartlang.org/docs/spec/
[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[external contributer form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf




