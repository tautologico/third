+++
author = "Andrei Formiga"
date = "2011-06-06T11:06:16-03:00"
description = "A guide about CamlP4 and the available documentation"
draft = false
keywords = ["key", "words"]
tags = ["ocaml", "camlp4", "metaprogramming"]
title = "A (meta-)guide to CamlP4: Metaprogramming in OCaml"
topics = ["programming"]
type = "post"

+++

This post is meant to be a guide to the available documentation and
tutorials about CamlP4, assuming no previous experience with it. The
topics are presented in the best order (IMO, of course) for learning,
under this assumption. Before attempting to learn CamlP4, it is
recommended to learn how to program in OCaml reasonably well, and to
have at least some familiarity with parsing and programming language
tools. If you have the prerequisites, though, this guide should at
least make learning CamlP4 a little easier than going blind after
everything returned by a search for “CamlP4″. The intention is to give
the Big Picture, so that the details can be worked out later.

### Roadmap

- Introduction: what is CamlP4.
- Revised Syntax: CamlP4 uses an alternative concrete syntax for
  OCaml. To learn CamlP4, you must learn this alt syntax.
- Quotations: CamlP4 is generally used to generate OCaml code, one way
  or another. With quotations this is easier. Quotations are
  customizable, so it can be used for other concrete syntaxes beyond
  OCaml’s.
- Grammars and Extensible Parsers: CamlP4 has an embedded notation for
  parser generation. This can be used for defining parsers or
  extending existing ones.
- Filters and Printers: the final output of CamlP4 is a syntax tree,
  printed in a chosen format. This tree can also be filtered before
  printing.
- Putting it all together: the structure of a syntax extension for
  OCaml, and links to examples.
- Sources: links to and comments on the CamlP4 tutorials and guides
  available in the Web. Most of them are linked in a previous part.

### Introduction

CamlP4 is the Pre-Processor and Pretty-Printer for OCaml. Given
textual input, CamlP4 parses the input into an abstract syntax tree,
which is then printed in some format. Both the parser and the printer
processes are customizable, so it’s a very flexible tool for
programming language processors.

CamlP4 is mostly used as a metaprogramming tool for OCaml, but it can
be used in many ways:

- to quickly create small parsers in OCaml
- to pretty-print OCaml programs (it probably could be used as the
  standard formatting tool for OCaml code, as [gofmt is used in the Go
  Language](http://golang.org/doc/effective_go.html#formatting))
- to integrate external syntaxes via a quotation mechanism (SQL
  queries, for example)
- to generate OCaml code from the output of a parser or a quotation
  (so you can compile a DS(E)L to OCaml code)
- to change OCaml’s concrete syntax, adding new syntactic forms,
  removing existing forms or changing the existing forms. Of course,
  it is possible to completely change OCaml’s syntax using these
  mechanisms

As this list shows, it is a very powerful and useful tool for parsing
and metaprogramming. It is often used to write syntax extensions to
OCaml, like adding support for a
[notation for monads](http://www.cas.mcmaster.ca/~carette/pa_monad/)
or for [list comprehensions](https://github.com/ocaml-batteries-team/batteries-included/blob/master/src/syntax/pa_comprehension/).

It’s also kind of a mess.

Many factors contribute to this. First of all, if you’re used to
metaprogramming in Lisp languages (as I was before meeting CamlP4),
there’s the increased complexity of metaprogramming with a
statically-typed language with non-uniform syntax. Then there’s the
fact that CamlP4 defines a different concrete syntax for OCaml, called
Revised Syntax, and to effectively use CamlP4 you have to know this
new syntax; not only that, but a non-trivial program using CamlP4 will
probably use both syntaxes, in the same source files. So the first
order of business if you want to learn to use CamlP4 is learning the
Revised syntax.

### Revised Syntax, or A Tale of Two Pre-Processors

Ok, how’s the Revised syntax then? This brings up another source of
confusion: there are actually two Pre-Processor-Pretty-Printers for
OCaml: CamlP4 and CamlP5 (which was CamlP4 before). The story, as far
as I know (which isn’t much) is this:
[Daniel de Rauglaudre](http://pauillac.inria.fr/~ddr/index-english.html)
wrote the original CamlP4, which was available for OCaml since its
early versions. For OCaml 3.10, however, the OCaml team wanted to make
changes in CamlP4, and the original author didn’t agree with them. So
a fork ensued: CamlP4 is the version included in the official OCaml
distribution, maintained by the core team; CamlP4 is mostly compatible
with the old CamlP4, but enough changes were made so that most code
written for the old version does not work with the current one. CamlP5
is the “old” CamlP4, renamed and maintained by the original author,
Daniel de Rauglaudre. It is (or was) completely compatible with the
old versions of CamlP4, although apparently it is now
[introducing incompatible changes](http://pauillac.inria.fr/~ddr/camlp5/MODE).
In this post I’ll stick with CamlP4 for the
current versions, mostly because it’s already there in the official
OCaml distribution.

Back to the Revised syntax: it’s not very well documented. Actually,
although the old CamlP4 had an official reference manual and tutorial,
the new CamlP4 has neither. The current version of CamlP4 has a kind
of official documentation in a
[wiki](http://brion.inria.fr/gallium/index.php/Camlp4), but it’s quite
incomplete. There’s a
[page in the wiki](http://brion.inria.fr/gallium/index.php/Revised)
about the Revised syntax, but it’s missing a lot. The
[section about the Revised syntax](http://caml.inria.fr/pub/docs/manual-camlp4/manual007.html)
in the latest official reference manual for the old CamlP4 (version
3.07) is complete, but inaccurate; some changes in the Revised syntax
were made in version 3.10. For now, there are no better options, so the
only way out is to read both sources and try to integrate them
mentally. Or read the CamlP4 sources, in this case the OCaml
parsers. The relevant files are pointed later, in the section about
parsers.

However, there is a good source of examples of the revised syntax:
CamlP4 itself is written in this syntax. So it’s possible to take a
look at the sources (included in the
[OCaml source distribution](http://caml.inria.fr/ocaml/release.en.html)) to
look at how things are done.

If you know the revised syntax, you can start to use quotations to
generate OCaml code.

### Quotations and Abstract Syntax

Quotations allow the programmer to treat a piece of code as data
instead of being part of the program itself. They are widely used in
Lisp because of its uniform representation for code and data, and are
widely used when programming in CamlP4 because they make it easier to
generate code. Ultimately, CamlP4’s output is an abstract syntax tree,
printed in some chosen format. The AST nodes are defined with
algebraic data types, so it’s possible to generate code just by
creating a value of the right type. However, this type is recursive
(as expected for an AST) and trees for any non-trivial piece of code
will be complicated to create as a value of the AST type. For example,
this piece of code:

~~~ocaml
let f x = x * x in f 5

~~~

corresponds to this AST as a value:

~~~ocaml
Ast.StExp (_loc,
  (Ast.ExLet (_loc, Ast.ReNil,
     (Ast.BiEq (_loc, (Ast.PaId (_loc, (Ast.IdLid (_loc, "f")))),
        (Ast.ExFun (_loc,
           (Ast.McArr (_loc, (Ast.PaId (_loc, (Ast.IdLid (_loc, "x")))),
              (Ast.ExNil _loc),
              (Ast.ExApp (_loc,
                 (Ast.ExApp (_loc,
                    (Ast.ExId (_loc, (Ast.IdLid (_loc, "*")))),
                    (Ast.ExId (_loc, (Ast.IdLid (_loc, "x")))))),
                 (Ast.ExId (_loc, (Ast.IdLid (_loc, "x")))))))))))),
     (Ast.ExApp (_loc, (Ast.ExId (_loc, (Ast.IdLid (_loc, "f")))),
        (Ast.ExInt (_loc, "5")))))))

~~~

It’s easy to see that it’s already unbearable to generate AST nodes
creating values of the algebraic data type, even for a single line of
code. If, instead, you simply quote the line of code above, CamlP4
will expand the quotation into the same AST. The only thing to be
aware of is that OCaml code inside quotations must use the revised
syntax. Support for the original syntax inside quotations was added in
OCaml 3.10, but it is considered to be in beta, and it seems to be
broken as of 3.12.

Quotations also allow for antiquotations, which are parts of a
quotation that should be evaluated instead of directly transformed to
AST nodes. This is basic in code generation: we use code templates for
translation/generation, but the templates wouldn’t be very interesting
if they were fixed pieces of code. Instead, each template has gaps
that must be filled with data which depends on the situation. This is
similar to the way format strings work in printf-like functions. It is
also possible to have a quotation inside an antiquotation, and an
antiquotation inside this quotation that is inside the antiquotation
of the original quotation, and so on. I know this sounds confusing, as
messing with quotations can often be, but in most cases it is easier
to learn them by example.

As with most things in CamlP4, quotations are also customizable. It is
possible to define new quotation expanders, in practice adding support
for quoting pieces of code in a syntax different than
OCaml’s. Expanders can also generate strings instead of AST nodes,
although this is less useful.

To learn about quotations (and antiquotations), the reference manual
for the old CamlP4 has a
[general overview](http://caml.inria.fr/pub/docs/manual-camlp4/manual004.html)
that’s still applicable. To learn how to use quotations to generate
OCaml AST nodes, you can look at
[this appendix](http://caml.inria.fr/pub/docs/manual-camlp4/manual010.html)
from the same manual. The CamlP4 wiki has a
[page on quotations](http://brion.inria.fr/gallium/index.php/Quotation),
and a page making an
[analogy between quotations and strings](http://brion.inria.fr/gallium/index.php/Strings_with_expansion).
Jake Donham has written a
[series of posts](http://ambassadortothecomputers.blogspot.com/p/reading-camlp4.html)
in his blog about
CamlP4\. [Part 2](http://ambassadortothecomputers.blogspot.com/2009/01/reading-camlp4-part-2-quotations_04.html)
and
[part 3](http://ambassadortothecomputers.blogspot.com/2009/01/reading-camlp4-part-3-quotations-in.html)
are about quotations from the perspective of a user, while
[part 8](http://ambassadortothecomputers.blogspot.com/2010/08/reading-camlp4-part-8-implementing.html)
and
[part 9](http://ambassadortothecomputers.blogspot.com/2010/08/reading-camlp4-part-9-implementing.html)
are about implementing new quotations and antiquotations. Two warnings
though: he sometimes writes as if you already knew this stuff, and
parts 2 and 3 about quotations use OCaml quotations in original
syntax; they mostly don’t work with OCaml 3.12\. That’s one more
situation where knowing the revised syntax comes in handy.

### Grammars and Extensible Parsers

CamlP4 makes it easy to create parsers, because it includes an
embedded notation for parser generation. The user defines a grammar
using a special notation, and CamlP4 generates a parser for
it. There’s a
[simple tutorial](http://brion.inria.fr/gallium/index.php/Syntax_extension_tutorial)
in the CamlP4 wiki about grammars; despite being titled “Syntax
Extensions”, it is not about syntax extensions for OCaml, but about a
parser for a simple external language for expressions. The reason for
the title will be explained in a bit. There’s also a
[sequel tutorial](http://brion.inria.fr/gallium/index.php/OCaml_code_generation_tutorial)
showing how to generate code for the simple expression language, using
quotations.

The good thing about grammars and parsers in CamlP4 is that they are
extensible. Any loaded module can extend a grammar defined in another
module, and an extension can not only add new productions, but also
change existing ones or even delete them. That’s why the tutorial
above is called “Syntax Extension”: it is extending the empty grammar
to define the syntax for a simple expression language.

And now for the punchline: CamlP4 comes with parsers for the syntax of
OCaml (revised and original variants, possibly others). So if you
extend the OCaml grammar from CamlP4, you can change OCaml’s
syntax. That’s the way to create syntax extensions for OCaml.

Besides the above-linked tutorials in the CamlP4 wiki, the
[section about grammars](http://caml.inria.fr/pub/docs/manual-camlp4/manual005.html)
in the old manual is still very
useful. [Part 6](http://ambassadortothecomputers.blogspot.com/2010/05/reading-camlp4-part-6-parsing.html)
of Jake Donham’s series about CamlP4 is a good reference, and goes
into quite some detail about the parsing methods used by CamlP4\. You
can ignore everything about streams and stream parsers if you want,
it’s just a somewhat old notation for simple recursive descent parsers
that are not needed for extending OCaml’s syntax.

### Filters and Printers

So CamlP4 parses its input and then builds an abstract syntax tree out
of it. This AST will be emitted, or printed, in a chosen
format. Between parsing and printing, it is possible to define
[AST Filters](http://ambassadortothecomputers.blogspot.com/2010/03/reading-camlp4-part-5-filters.html)
that can transform the tree, including maps and folds over it.

From the point of view of syntax extensions for OCaml, CamlP4 parses
OCaml code, most likely using an extended syntax, generates an AST
that may be filtered, and then prints it. The generated AST can be
emitted by a pretty-printer, showing code in a readable format for
humans. You could feed the output of the pretty-printer to the OCaml
compiler, thus effectively activating the syntax extension. However,
this has some disadvanges:

-   The compiler would need to read and parse the input again, making
    the compiling process take longer
-   Errors detected by the compiler wouldn’t necessarily be reported
    at their original locations, but rather at the locations resulting
    from pre-processing (this is a common problem with pre-processors)

Fortunately, there’s a solution: CamlP4 has a printer that emits the
AST in a binary, marshaled form for the OCaml compiler, which can then
skip the parsing stages and use the tree directly. The marshaled tree
also includes location information, which allows the compiler to
report errors correctly for the input source. To use this from the
OCaml compilers, you just need to use the -pp command-line
flag. [This page](http://brion.inria.fr/gallium/index.php/Using_Camlp4)
in the CamlP4 wiki has a good overview about using CamlP4 by itself
and together with a compiler.

It is also possible to define new printers, though most of the time
this is not very useful.

### Putting it all together: OCaml syntax extensions

Conceptually, the plan is simple: extend the OCaml parser in CamlP4,
generate code for the extensions using quotation, then feed the
generated tree to the compiler. To extend the OCaml parser, it may be
useful to take a look at how it is defined for the standard
syntax(es). In the OCaml sources, the parsers are available in the
directory camlp4/CamlP4Parsers. CamlP4OCamlRevisedParser.ml is the
parser for the revised syntax, while CamlP4OCamlParser.ml is for the
original syntax. The latter one is defined as an extension of the
former, so you may need to consult both.

A good idea is to look at examples of simple syntax extensions. The
wiki has a page with a
[simple extension for float expressions](http://brion.inria.fr/gallium/index.php/Pa_float)
(this extension uses a map over the AST to avoid having to rewrite
many grammar productions). Richard Jones posted an example in the
official Caml-list for
[wrapping pattern matching in a predicate](http://www.mail-archive.com/caml-list@yquem.inria.fr/msg00049.html).
[Part 11](http://ambassadortothecomputers.blogspot.com/2010/09/reading-camlp4-part-11-syntax.html)
in Jake Donham’s series includes a sequence of examples, starting with
a very simple one. More examples can be found in the recommended
sources below.

### Sources and Final Thoughts

CamlP4 gives OCaml programmers much of the power of metaprogramming
available in Lisp languages, added with static type checking and
customizable components. Some things CamlP4 can do, like integration
of external syntaxes in OCaml programs, are not easy to replicate in
Lisp. However, it is a quite complex piece of software and this is
sometimes exposed to users. Furthering the difficulties, it is now
fragmented (CamlP4 and CamlP5) and not very well documented. I hope
this post helps people get up to speed in using this handy tool.

As I mentioned, this is not a tutorial on CamlP4\. A proper tutorial
would be quite useful, but it would also demand much more time from
me, so I decided to do the next best thing: give some pointers and
commentary on the documentation and tutorials that are available out
there. So here’s a list of other good sources about CamlP4, with
commentary:

*   The
    [old CamlP4 manual](http://caml.inria.fr/pub/docs/manual-camlp4/index.html)
    is outdated, but still useful because there’s no equivalent for
    the new CamlP4.
*   [The series of posts on CamlP4](http://ambassadortothecomputers.blogspot.com/p/reading-camlp4.html)
    over at Ambassador at the Computers is a good source, with some
    caveats. Jake Donham probably knows a lot more about this stuff
    than me, but sometimes he seems to be writing to people who
    already know about CamlP4, especially in the first few posts. The
    sequence could start better. He also uses quotations in original
    syntax in the earlier parts, rendering the example code unusable
    in current versions of OCaml (in some cases he linked to newer
    versions that work). On the other hand, the posts often go deeper
    than what is available elsewhere, so it’s worth it.
*   The
    [new CamlP4 wiki](http://brion.inria.fr/gallium/index.php/Camlp4)
    has useful stuff, although it is incomplete both as a tutorial and
    as a reference.
*   [This CamlP5 tutorial](http://martin.jambon.free.fr/extend-ocaml-syntax.html)
    by Martin Jambon is very good. I just found out about it as I was
    almost finished writing this post. It is the kind of tutorial I
    think is missing for CamlP4, but unfortunately, it is targeted to
    CamlP5, aka the old CamlP4\. Still very useful.

Besides that, there are always the sources.
