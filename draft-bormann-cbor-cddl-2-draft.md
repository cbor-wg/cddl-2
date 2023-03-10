---
v: 3

title: >
  CDDL 2.0 — a draft plan
abbrev: CDDL 2.0
docname: draft-bormann-cbor-cddl-2-draft-latest
# date: 2022-10-19

keyword: Internet-Draft
cat: info
stream: IETF
wg: CBOR Working group

venue:
  mail: "cbor@ietf.org"
  github: cbor-wg/cddl-2

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org


normative:
  RFC8610: cddl
  RFC9165: control1

informative:
  RFC5234: abnf
  RFC7405: abnf-case
  RFC9090: oid
  useful:
    target: https://github.com/cbor-wg/cddl/wiki/Useful-CDDL
    title: Useful CDDL
  I-D.draft-bormann-cbor-cddl-freezer: freezer
  cddlc:
    title: CDDL conversion utilities
    target: https://github.com/cabo/cddlc
  PSVI:
    target: https://www.w3.org/XML/2002/05/psvi-use-cases
    date: 2002-06-24
    title: Use Cases for XML Schema PSVI API


--- abstract

The Concise Data Definition Language (CDDL) today is defined by
RFC 8610 and RFC 9165.
The latter (as well as some more application specific specifications
such as RFC 9090) have used the extension point provided in RFC 8610,
the control operator.

As CDDL is used in larger projects, feature requirements become known
that cannot be easily mapped into this single extension point.
Hence, there is a need for evolution of the base CDDL specification
itself.

The present document provides a roadmap towards a "CDDL 2.0".
It is based on draft-bormann-cbor-cddl-freezer, but is more selective
in what potential features it takes up and more detailed in their
discussion.
It is intended to serve as a basis for prototypical implementations of
CDDL 2.0.
What specific documents spawn from the present one or whether this
document is evolved into a single CDDL 2.0 specification.

--- middle

Introduction        {#intro}
============

(Please see abstract.)

Note that the existing extension point can be exercised for new
features in parallel to the work described here.
One such draft, {{-more-controls}}, is planned to form the first set of
specifications going forward from the CDDL-2 project together with {{-grammar}}.

Mending syntax deficits {#syntax}
======================

The previous content of this section formed the basis for {{-grammar}},
except for {{tagolit-ref}}.

Tag-oriented Literals {#tagolit-ref}
---------------------

Incomplete, see {{tagolit}}.

Processing model: Beyond Validation
================

{:compact}
*Proposal Status*:
: experiments with implementations ongoing

*Compatibility*:
: backwards compatible

The basic (implicit) processing model for CDDL 1.0 applies a CDDL data
model to a data item and returns a Boolean that indicates whether the
data item matches that model ("*validation*").

{{Section 4 of RFC9165}} extends this model with named "*features*".
A validation can indicate which features were used.
Validation could also be parameterized with information about what
features are allowed to be used, enabling variants (see {{Section 4 of
RFC9165}} and {{useful}} for examples).

The `cddl` tool ({{Appendix F of RFC8610}}) also supports experimental
forms of "annotating" a validated data item with information about
which rules were used to support validation, currently entirely based on the
information that is in a standard CDDL 1.0 data model.
This leads to a more general concept of "*annotation*", where the data
model specification supports "annotating" the validated instance by
optionally supplying information in the model.
(The annotated result is a special case of a "post-schema validation
instance" [PSVI], here one where the data item itself is only
augmented, not changed, by the process.)

Annotations could in turn provide input to further validation steps,
as is often done with Schematron validation in Relax-NG; with an
appropriate evaluation language this can be used for checking co-occurrence
constraints ({{Section 5 of -freezer}}).

Finally, annotations are a first step to *transformation*, i.e.,
describing how a validated data item should be interpreted as a
transformed data item by performing certain computations.
This generally requires even more support from an evaluation language,
simple transformations such as adding in default values may not need
much support though.

At this time, existing experimental implementations do not lead to a
clear choice for what processing model enhancements should be in
CDDL 2.0.
This document proposes to continue the experimentation and document
good approaches.

Module superstructure
=====================

{:compact}
*Proposal Status*:
: collection of rough ideas with examples; initial subset implemented

*Compatibility*:
: bidirectional (both backward and forward)

Originally, CDDL was used for small data models that could be
expressed in a few lines.  As the size of data models that need to be
expressed in CDDL has increased, the need to modularize and re-use
components is increasing.

CDDL 1.0 has been designed with a crude form of composition:
Concatenating a number of CDDL snippets creates a valid CDDL data
model unless there is a name collision (identical redefinition is
allowed to facilitate this approach).
With larger models, managing the name space to avoid collisions
becomes more pressing.

The knowledge which CDDL snippets need to be concatenated in order to
obtain the desired data model lives entirely outside the CDDL snippets
in CDDL 1.0.
In CDDL 2.0, rules will be packaged as modules and referenced from other
modules.

There needs to be some control of namespace pollution, as well as
unambiguous referencing into evolving specifications ("versioning")
and selection of alternatives (as was emulated with snippets in
{{Section 11 of ?RFC8428}}, although an alternative approach for expressing
variants is demonstrated in {{useful}} based on {{Section 4 of RFC9165}}).

Compatibility
-------------

One approach to achieve the module structure that is friendly to
existing environments that operate with CDDL 1.0 snippets and CDDL 1.0
implementations is to add a super-syntax (similar to the way pragmas
are often added to a language), e.g., by carrying them in what is
parsed as comments in CDDL 1.0.

This enables each module source file to be valid CDDL 1.0 (if
missing some rule definitions to be imported).


Namespacing
-----------

A convention for mapping CDDL-internal names to external ones could be
developed, possibly steered by some pragma-like constructs.  External
names would likely be URI-based, with some conventions as they are
used in RDF or Curies.  Internal names might look similar to XML
QNames.  Note that the identifier character set for CDDL deliberately
includes $ and @, which could be used in such a convention.

Note that this convention should not pollute the actual contents of
the model, where adding a simple prefix to rule names defined
elsewhere may be all that is needed.

Cross-universe references
-------------------------

See {{cross}}.

The "module", "directives"
------------

A single CDDL file becomes a *module* by processing the (zero or more)
*directives* in it.

The semantics of the module are independent of the module(s) using it,
however, using a module may involve transforming its rule names into a
new namespace.

Directives look like comments in CDDL 1.0, so they do not interfere
with forward compatibility.

Lines starting with the prefix `;#` are parsed as directives in CDDL
2.0.

Finding modules
---------------

For now, we assume that module names are filenames taken from one of
several sources available to the CDDL 2.0 processor via the environment.
This avoids the need to nail down pathnames or partial URIs into the
CDDL files.

In the CDDL 2.0 Tool described in {{cddlc-tool}}, the set of sourced is
determined from an environment value, `CDDL_INCLUDE_PATH`, which is
modeled after usual command-line search paths.
It is a colon-separated list of pathnames to directories, with one
special feature: an empty element points to he tool's own collection.
In the current version, this collection contains 20 fragments of
extracted CDDL from published RFCs, using names such as `rfc9052`.

(Future versions might augment this with Web extractors and/or ways to
extract CDDL modules from github and from Internet-Drafts.)

The default `CDDL_INCLUDE_PATH` is  `.:` — i.e., files are found in the current directory and, if not found there, cddlc’s collection.

Initial Set of Directives {#directives}
-------------------------

Two groups of directives are defined at this point:

* `include`, which includes all the rules from a module (which
  includes the ones imported/included there, transitively), or
  specific explicitly selected rules

* `import`, which includes only those rules from the module that are
   referenced, implicitly or explicitly (see below), including the
   rules that are referenced from these rules, transitively.

The `include` function is more useful for composing a single model
from parts controlled by one author, while the `import` function is
more about treating a module as a library:

{::include code/simple-import.md}

This is appropriate for using libraries that are well known to the
imported.
However, if it is not acceptable that the library can pollute the
namespace of the importing module, the import directive can specify a
namespace prefix:

{::include code/namespaced-import.md}

Note how the imported names are prefixed with `cose.` as specified in
the import directive, but CDDL prelude ({{Appendix D of -cddl}}) names such as `tstr` and `any` are not.

Explicit selection of names
---------------------------

Both `import` and `include` directives can be augmented by an explicit
mentioning of rule names.

Starting with `include`:

{::include code/includefrom.md}

The module from which rules are explicitly imported can be namespaced:

{::include code/includefrom-namespaced.md}

Both examples would work exactly the same with `import`, as the
included rules do not reference anything else from the included
module.

An import however also draws in the transitive closure of the rules
referenced:

{::include code/importfrom-namespaced.md}

The `import` statement can also request an alias for an imported name:

{::include code/importfrom-renamed.md}

Tool Support for Command-Line Control
------------------

{::include code/zero.md}

In other words, the module had an empty CDDL file, which therefore was
not provided (no `–` on the command line).

ABNF is a lot like CDDL
------------------------

Many of the constructs defined here for CDDL also could be used with
ABNF specifications.  ABNF would definitely benefit from a standard
way to import snippets from existing RFCs.
Since CDDL contains ABNF support ({{Section 3 of RFC9165}}), it would be
natural to make some of the functionality discussed in this section
available for ABNF as well.

IANA Considerations
==================

(Insert new registry for application specific literals here, if adopted.)


Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

Fridge
======

This appendix contains sections that may not make it to a 2.0, but
might be part of a followup.


Tag-oriented Literals {#tagolit}
---------------------

{:compact}
*Proposal Status*:
: rough idea, porting from EDN

*Compatibility*:
: backward (not forward)

Some CBOR tags often would be most natural to use in a CDDL spec with a literal
syntax that is tailored to their semantics instead of their
serialization in CBOR.  There is currently no way to add such syntaxes, no
defined extension point either.

The proposal
"Application-Oriented Literals in CBOR Extended Diagnostic Notation"
{{?I-D.bormann-cbor-edn-literals}} defines application-oriented
literals, e.g., of the form

> dt'2019-07-21T19:53Z'

for datetime items.  With additional considerations for unambiguous
syntax, a similar literal form could be included in CDDL.

This proposal opens a name space for the prefix that indicates an
application specific literal.
A registry could be provided to make this name space a genuine
extension point.
(This is currently the production `bsqual` in {{Appendix B of RFC8610}}.)

The syntax provided in {{?I-D.bormann-cbor-edn-literals}} does not
enable the use of CDDL types — it has the same flaw that is being
fixed for tag numbers in {{tagnum}}.


Cross-universe references {#cross}
-------------------------

Often, a CDDL specification needs to import from specifications in a
different language or platform.

### IANA references

In many cases, CDDL specifications make use of values that are
specified in IANA registries.  The `.iana` control operator can be
used to reference such a set of values.

The reference needs to be able to point to a draft, the registry of
which has not been established yet, as well as to an established IANA registry.

An example of such a usage might be:

~~~~ CDDL
cose-algorithm = int .iana ["cose", "algorithms", "value"]
~~~~

Unfortunately, the vocabulary employed in IANA registries has not been
designed for machine references.  In this case, the potential values
would come from applying the XPath expression

~~~~ xpath
//iana:registry[@id='algorithms']/iana:record/iana:value
~~~~

to `https://www.iana.org/assignments/cose/cose.xml`, plus some
filtering on the records returned that only leaves actual allocations.
Additional functionality may be needed for filtering with respect to other
columns of the registry record, e.g., `<capabilities>` in the case of
this example.

A CDDL 2.0 Tool {#cddlc-tool}
===============

This appendix is for information only.

A rough CDDL 2.0 tool is available {{cddlc}}.  It can process CDDL 2.0
models into CDDL 1.0 models that can then be processed by the CDDL
tool described in {{Appendix F of -cddl}}.

A typical command line involving both tools might be:

~~~
cddlc -2 -tcddl mytestfile.cddl | cddl - gp 10
~~~

Install on a system with a modern Ruby (Ruby version ≥ 3.0) via:

    gem install cddlc

The present document assumes the use of `cddlc` version 0.1.5.

Acknowledgements
================
{: numbered="no"}

TBD
